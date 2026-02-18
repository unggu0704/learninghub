---
title: "[Cloud Design Pattern] SpringBoot에서 Azure Event Hub를 통한 pub-sub 패턴 구현"
author: "unggu"
date: 2025-07-02 12:34:12 +0800
categories: [DevOps, Container]
tags: [Kubernets, Container, DevOps, pubsub, Azure, Eventhub, springboot]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/pubsub.png
---


## Pub-Sub 패턴이란

송신자(publisher)와 수신자(Subscriber)가 서로 알지 못한 상태로도 비동기 통신할 수 있도록 하는 메세징 패턴입니다.

### 일반적인 통신(Rest API)
![image.png]({{ site.baseurl }}{{ page.url }}/img/ssss.drawio.png)

**A라는 publisher 서비스**와 **B라는 Consumer**라고 가정하겠습니다. 
B서비스는 A서비스에게 요청하기 위해서는 아래와 같은 제약사항이 존재합니다.
- **A의 IP 주소를 알아야함**
- **A 서비스에 문제가 생기면 B에도 영향** 
- **1대1 통신만 가능함**
A와 B는 둘 사이에 강력한 결합의 상태입니다. 
pub-sub 패턴을 구현하기 위해 **Azure Event Hub**을 사용하여  **느슨한 결합**을 구현해보겠습니다.


### 느슨한 결합

![image.png]({{ site.baseurl }}{{ page.url }}/img/제목 없는 다이어그램.drawio.png)

B는 더이상 A의 주소를 몰라도 됩니다. 
자신이 데이터를 직접 Azure Event Hub에 발행해놓으면 B는 여기서 데이터를 받아오기만 하면됩니다.

만약 수신자가 늘어난다고 하여도,  수신자끼리 별도의 **Consumer Group**을 통해 각자 데이터를 수신 받을 수 있습니다.

A 서비스가 장애가 나더라도 그동안 A가 보낸 메세지는 Event hub에 7일간 보관됩니다. 기존에 저장된 데이터는 지속적으로 소비가 가능합니다.
추후 A 서비스가 복구된다면 그 다음(**offset**)을 통해 수신자들은 그 다음 데이터부터 이어서 받아볼 수 있습니다.

**Consumer Group**
여러 명의 수신자가 있을 때 Consumer Group은 독립적인 **offset**(어디까지 읽었는지)를 통해  여러 수신자에 대한 독립적 읽기를 지원합니다.

> *Azure Event Hub는 추가적으로 partition이라는 개념을 통해 내부적으로 저장소를 분리해놓습니다. 
> 하나의 partition은 한개의 인스턴스만 읽을 수 있기에 각각의 수신자는 다수의 인스턴스 설정을 통해, 병렬적으로 데이터를 수신할 수 있습니다.*

---
## Azure EventHub를 사용하여 pub-sub 패턴 구현하기


MVP 프로젝트 어플리케이션에서 Azure Eventhub를 연동하여 pub-sub 패턴을 구현한 예시를 확인해보도록 하겠습니다. 

해당 서비스의 아키텍쳐는 아래와 같습니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/pubsub.drawio.png)

Store와 Review는 Spring Boot기반 컨테이너 환경에서 MSA로 구현되어 있습니다.
Store는 외부 가게 리뷰(카카오맵)에서 데이터를 크롤링하여 가져옵니다. 
이를 Review는 자신의  review 데이터베이스에 저장을 해야합니다.
이 사이에서 Azure Event Hub를 사용해 Pub-Sub 패턴을 컨테이너 기반 Spring Boot에서 어떻게 구현하는지 알아보겠습니다.

#### k8s 영역
Java 코드를 설정하기 전에 Event Hub 연동을 위한 연결 Key(Connection String)를 설정해야합니다.,
Connection String은 Azure Portal에서 확인할 수 있습니다.

해당 Connection String는 민감한 정보이기에 Secret을 통해 관리합니다.

**secret.yml**

```
apiVersion: v1
kind: Secret
metadata:
  name: eventhub-secret
  namespace: ns-hiorder
type: Opaque
data:
  connectionString: <connection string>
```

이 secret을  deployment에게 주입해줍니다.

**deployment.yml**

```
env:
  - name: AZURE_EVENTHUB_CONNECTION_STRING
    valueFrom:
      secretKeyRef:
        name: eventhub-secret   
        key: connectionString    
```
여기까지 k8s(manifest) 영역입니다.

---
### 코드 영역
EventHub를 사용할 Spring Boot의 application.yml에서 주입받은 값을 환경변수로 등록합니다.
```
# Azure Event Hub 설정 (추가)  
azure:  
  eventhub:  
    connection-string: ${AZURE_EVENTHUB_CONNECTION_STRING}
```

build.gradle에서 관련 라이브러리를 설정합니다.
```
implementation 'com.azure:azure-messaging-eventhubs:5.15.0'
```

#### store MSA 전체 코드(Publisher)
```
// ExternalIntegrationInteractor.java

@Value("${spring.azure.eventhub.connection-string}")  
private String eventHubConnectionString;


private void publishSyncEvent(Long storeId, String platform, int syncedCount) {
    // 1. 매번 Producer 생성하는 방식 
    EventHubProducerClient producer = new EventHubClientBuilder()
        .connectionString(eventHubConnectionString)  // EntityPath 포함
        .buildProducerClient();
    
    // 2. 이벤트 페이로드 생성
    Map<String, Object> eventPayload = createEventPayloadFromRedis(storeId, platform, syncedCount);
    
    // 3. 발행
    EventDataBatch batch = producer.createBatch();
    batch.tryAdd(new EventData(payloadJson));
    producer.send(batch);
    
    producer.close();  // 매번 종료
}
```

**1. Producer 선언**
```
@Value("${spring.azure.eventhub.connection-string}")  
private String eventHubConnectionString;
(...) 생략
// 1. 매번 Producer 생성하는 방식 
    EventHubProducerClient producer = new EventHubClientBuilder()
        .connectionString(eventHubConnectionString)  // EntityPath 포함
        .buildProducerClient();
```
*Kubernetes Secret → Deployment 환경변수 → application.yml* 
*→ Java 필드 eventhub-secret  → AZURE_EVENTHUB_CONNECTION_STRING → ${...} → @Value 주입*

**위 과정을 통해 eventHubConnectionString에 connectionString이 설정되었습니다.**

connection-string를 바탕으로 `producer`라는 객체를  선언합니다.

**2. payload 설정**
```
// 2. 이벤트 페이로드 생성
    Map<String, Object> eventPayload = createEventPayloadFromRedis(storeId, platform, syncedCount);
```
 
 redis에 있는 데이터를 `createEventPayloadFromRedis`에서 주입 받아 `eventPayload`에 저장합니다.
`eventPayload`에 저장되어 있는 payload는 아래와 같습니다. 
 
```
Map<String, Object> payload = new HashMap<>();  
payload.put("eventType", "EXTERNAL_REVIEW_SYNC");  
payload.put("storeId", storeId);  
payload.put("platform", platform);  
payload.put("syncedCount", syncedCount);  
payload.put("timestamp", System.currentTimeMillis());
```
 위 payload는 아래와 같은 데이터가 생성될겁니다.
```
{
    "eventType": "EXTERNAL_REVIEW_SYNC",  // 이벤트 타입 (고정값)
    "storeId": 123,                       // 매장 ID
    "platform": "KAKAO",                  // 플랫폼 (NAVER/KAKAO/GOOGLE)
    "syncedCount": 50,                    // 동기화된 리뷰 개수
    "timestamp": 1704067200000            // 이벤트 발생 시간
}
{reviews...}
```


**3. 이벤트 발행**

여러 이벤트를 한번에 전송 가능한 EventDataBatch를 통해 EventHub로 데이터를 전송합니다.
```
    // 3. 발행
    EventDataBatch batch = producer.createBatch();
    batch.tryAdd(new EventData(payloadJson));
    producer.send(batch);
    
    producer.close();  // 매번 종료
```

> *Store Publisher 코드는 매번 이벤트를 발행할 때 마다 producer를 선언하는 비효율적 코드입니다.*
> *이 방식은 Review Consumer에서 개선되었습니다.*

#### review MSA (Consumer)

**1. EventHubConfig 설정하기 (재사용되는 Consumer)**
```
@Configuration
public class EventHubConfig {

    @Value("${azure.eventhub.connection-string}")
    private String connectionString;

    @Value("${azure.eventhub.consumer-group:$Default}")
    private String consumerGroup;

    @Bean("externalReviewEventConsumer")
    public EventHubConsumerClient externalReviewEventConsumer() {
        return new EventHubClientBuilder()
                .connectionString(connectionString)
                .consumerGroup(consumerGroup)
                .buildConsumerClient();
    }
}
```
매번 생성하던 Producer와 달리, Consumer는 별도의 Config를 통해 **Bean으로 관리합니다**.
connectionString에 EntityPath를 포함시켜서 저장하기에 별도의 설정은 없습니다.

Consumer Group은 `$Default`로 설정하여 놓습니다.

**2. 리스너 설정하기**
```
@Slf4j
@Component
@RequiredArgsConstructor
public class ExternalReviewEventHubAdapter {

    @Qualifier("externalReviewEventConsumer")
    private final EventHubConsumerClient externalReviewEventConsumer;
    private final ReviewJpaRepository reviewJpaRepository;
    private final ObjectMapper objectMapper;
    private final ReviewRepository reviewRepository;

    private final ExecutorService executorService = Executors.newFixedThreadPool(3);
    private volatile boolean isRunning = false;

    // 🔥 중복 방지를 위한 이벤트 ID 캐시
    private final Set<String> processedEventIds = ConcurrentHashMap.newKeySet();
```
Consuemer를 주입받고 디비에 저장하기 위한 변수들을 설정합니다.

**3. 리스너 시작 / 종료 설정**
```
@PostConstruct
public void startEventListening() {
    log.info("외부 리뷰 Event Hub 리스너 시작");
    isRunning = true;
    // 별도 스레드에서 이벤트 수신 시작
    executorService.submit(this::listenToExternalReviewEvents);
}

@PreDestroy
public void stopEventListening() {
    log.info("외부 리뷰 Event Hub 리스너 종료");
    isRunning = false;
    executorService.shutdown();
    externalReviewEventConsumer.close();
}
```
Spring Boot 시작시에 자동으로 리스너가 시작할 수 있게 합니다. (`@PostConstruct`) `listenToExternalReviewEvents`에서 리스너 로직이 담겨있습니다.

**4. 이벤트 리스너 수신 로직 (Core)**
```
private void listenToExternalReviewEvents() {
    log.info("외부 리뷰 이벤트 수신 시작");

    try {
        while (isRunning) {
            Iterable<PartitionEvent> events = externalReviewEventConsumer.receiveFromPartition(
                    "4",                          // 파티션 ID 고정
                    100,                          // 최대 100개씩 가져오기
                    EventPosition.earliest(),     // 처음부터 읽기
                    Duration.ofSeconds(10)        // 10초 타임아웃
            );

            for (PartitionEvent partitionEvent : events) {
                handleExternalReviewEvent(partitionEvent);
            }

            Thread.sleep(1000);  // 1초 대기
        }
    } catch (InterruptedException e) {
        log.info("외부 리뷰 이벤트 수신 중단됨");
        Thread.currentThread().interrupt();
    } catch (Exception e) {
        log.error("외부 리뷰 이벤트 수신 중 오류 발생", e);
    }
}
```
4번 파티션만 설정 한것은 확장성을 제한하고 있지만, 처리속도를 빠르게 하기 위해서 임시적으로 설정했습니다. (**잘못된 방법**)
offset을 사용하지 않고 `EventPosition.earliest()`를 사용해 항상 처음부터 읽는 문제도 존재합니다.(**잘못된 방법**)


**5. 이벤트 검증 로직**
```
private void handleExternalReviewEvent(PartitionEvent partitionEvent) {
    try {
        EventData eventData = partitionEvent.getData();
        String eventBody = eventData.getBodyAsString();

        // 🔥 이벤트 고유 ID 생성 (중복 방지)
        String eventId = String.format("%s_%s",
                eventData.getOffset(),
                eventData.getSequenceNumber());

        // 🔥 이미 처리된 이벤트인지 확인
        if (isEventAlreadyProcessed(eventId)) {
            log.debug("이미 처리된 이벤트 스킵: eventId={}", eventId);
            return;
        }

        Map<String, Object> event = objectMapper.readValue(eventBody, Map.class);
        String eventType = (String) event.get("eventType");
        Long storeId = Long.valueOf(event.get("storeId").toString());

        log.info("외부 리뷰 이벤트 수신: type={}, storeId={}, eventId={}", 
                eventType, storeId, eventId);

        if ("EXTERNAL_REVIEW_SYNC".equals(eventType)) {
            handleExternalReviewSyncEvent(storeId, event);
            // 🔥 처리 완료된 이벤트 ID 저장
            markEventAsProcessed(eventId);
        } else {
            log.warn("알 수 없는 외부 리뷰 이벤트 타입: {}", eventType);
        }

    } catch (Exception e) {
        log.error("외부 리뷰 이벤트 처리 중 오류 발생", e);
    }
}
```
`EXTERNAL_REVIEW_SYNC`올바른 이벤트 type인지 검증하면서 이벤트 고유 ID를 생성해 중복 처리를 방지합니다.

**6.이벤트 처리 로직**
```
private void handleExternalReviewSyncEvent(Long storeId, Map<String, Object> event) {
    try {
        String platform = (String) event.get("platform");
        Integer syncedCount = (Integer) event.get("syncedCount");

        // Store에서 발행한 reviews 배열 처리
        @SuppressWarnings("unchecked")
        List<Map<String, Object>> reviews = (List<Map<String, Object>>) event.get("reviews");

        if (reviews == null || reviews.isEmpty()) {
            log.warn("리뷰 데이터가 없습니다: platform={}, storeId={}", platform, storeId);
            return;
        }

        log.info("외부 리뷰 동기화 처리 시작: platform={}, storeId={}, count={}",
                platform, storeId, reviews.size());

        int savedCount = 0;
        int duplicateCount = 0;
        int errorCount = 0;

        for (Map<String, Object> reviewData : reviews) {
            try {
                Review savedReview = saveExternalReviewWithDuplicateCheck(storeId, platform, reviewData);
                if (savedReview != null) {
                    savedCount++;
                } else {
                    duplicateCount++;
                }
            } catch (Exception e) {
                errorCount++;
                log.error("개별 리뷰 저장 실패: platform={}, storeId={}, error={}",
                        platform, storeId, e.getMessage());
            }
        }

        log.info("외부 리뷰 동기화 완료: platform={}, storeId={}, saved={}, duplicated={}, errors={}",
                platform, storeId, savedCount, duplicateCount, errorCount);

    } catch (Exception e) {
        log.error("외부 리뷰 동기화 이벤트 처리 실패: storeId={}, error={}", storeId, e.getMessage(), e);
    }
}
```
최종적으로 `saveExternalReviewWithDuplicateCheck()`로 보내면서 외부 리뷰들을 review MSA DB에 저장하였습니다.

**최종 흐름도**

![image.png]({{ site.baseurl }}{{ page.url }}/img/제목 없는 다이어그램.drawio (1).png)
