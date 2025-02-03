---
author: "unggu"
title: "[AZ-204] Azure Blob Storage"
date: 2025-01-06 22:41:32 +0800
categories: [Azure, Azure 204]
tags: [Azure, AZ 204, Azure Blob Storage]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/azure204/Azure_Blob_Storage.png
---
### 아키텍쳐

![스크린샷 2024-10-14 오후 5.04.43.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_5.04.43.png)

### Azure Blob Storage란?

클라우드를 위한 객체 기반 스토리지 솔루션으로 대량의 데이터를 저장하는데 최적화 되어있다. 

데이터의 특정 정의는 따라지 않는 형식

Blob Storage는 브라우저에 이미지나 문서를 제공하거나 비디오 스트링 및 분산 엑세스 파일 저장 그리고 클라우드의 가용성을 보장한다. 

*Azure 스토리지 계정*은 모든 Azure blob Storage의 최상위 컨테이너로 전세계 어디서든 HTTP, HTTPS를 통해 엑세스 할 수 있다.

일종의 파일 서버의 역할이며 온프레미스와 동기화 가능하다.

### Storage 계정 유형

두 가지 성능 수준의 스토리지 계정을 제공하는데 **표준**은 대부분의 시나리오에 권장되는 유형이다. **Premeium**은  SSD를 사용해 *블록 Blob*, *페이지 Blob*, *파일 공유* 중에서 선택이 가능하다.

**블록 Blob 데이터의 액세스 계층**

데이터 엑세스 계층은 Hot, Cool, Cold, Store 계층으로 나뉘며 앞에서부터 스토리지 비용은 높은 순이며, 엑세스 비용은 낮은 순이다.

- Hot 엑세스 : 자주 엑세스, 새 스토리지 계정은 Hot 계층이다.
- Cool 엑세스 : 대량의 데이터를 30일간 저장함
- Cold 액세스 : 90일간 데용량 데이터를 저장
- Archive 엑세스 : 개별 블록으로 저장되며 약 180일

### Storage 중복성
Azure Storage 계정의 데이터는 항상 기본지역에서 세번 복제된다. 
이런 중복성은 LRS와 ZRS 그리고 GRS로 나누어지는데

**LRS(로컬 중복 스토리지)**
LRS는 가장 저렴하지만 데이터 센터 내 재해에 있어 취약하다. 

**ZRS(영역 중복 스토리지)***
주 지역의 세 가지 가용성 영역에 복사하기에 가용성에 있어 뛰어나 Azure에서 추천하는 방식이다.

**GRS(지역 중복 저장소)**
주 지역을 제외한 보조 지역에 데이터를 비동기적으로 복사한다. 여기서 더 나아가 GZRS는 GRS를 ZRS처럼 하는 방식이다.

### Azure Blob Storage 리소스의 유형

![스크린샷 2024-10-14 오후 5.19.33.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-10-14_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_5.19.33.png)

**Storage 계정**

스토리지 계정은 Azure에서 고유한 네임스페이스를 제공한다. 또한 모든 Blob 데이터는 스토리지 계정 내부에 저장된다.

```yaml
http://mystorageaccount.blob.core.windows.net
```

**컨테이너**

하나의 스토리지 계정에 디렉터리와 비슷한 구조로 Blob을 논리적으로 그룹화한다. 이렇게 그룹화된 blob은 컨테이너의 이름을 일부로 URI가 제작된다.

```yaml
https://myaccount.blob.core.windows.net/mycontainer
```

**Blob**

Azure Storage는 세가지 유형의 Blob을 제공하는데 **블록 Blob**은 텍스터 및 이미지를 저장하며 **추가 Blob**은 추가 데이터를 저장하는데 최적화 되어 있어 로깅 같은 시나리오에 적합하다. **페이지 Blob**은 임의 엑세스 파일을 저장하는데 Azure VM의 디스크에 사용된다 
HTTP의 `PUT` 메서드를 사용하여 메타 데이터의 값을 저장할 수 있다.

```yaml
https://myaccount.blob.core.windows.net/mycontainer/myblob
```

### Azure Storage 보안 기능

Azure Storage는 SSE를 사용하여 암호화를 기능한다. 이때 256비트의 AES를 사용한다. 

이때 사용되는 암호화 키는 Microsoft 관리형 키, 고객 관리형 키, 고객 제공키로 나뉘어 키 옵션을 관리한다. 

## Azure Blob Storage 수명 주기

![image.png]({{ site.baseurl }}{{ page.url }}/img/image21.png)

Azure Blob Storage의 수명 주기 관리는 Blob 데이터를 적절한 엑세스 계층으로 전환해 수명주기가 끝나면 규칙 기반으로 제공한다.

어떠한 Blob에 있어 엑세스가 된다면 해당 Blob은 더욱 Hot 계층으로 옮겨진다. 이러한 Blob이 더이상 엑세스 되지 않거나 수정될 경우 더 Cool한 계층으로 옮겨져 비용을 자동적으로 최적화한다. 더 다음 버전의 Blob이 나온다면 이전 버전의 수명주기가 종료할시 해당 Blob은 삭제한다. 

### 수명 주기 rule의 설정

```yaml
{
  "rules": [
    {
      "name": "rule1",
      "enabled": true,
      "type": "Lifecycle",
      "definition": {...}
    },
    {
      "name": "rule2",
      "type": "Lifecycle",
      "definition": {...}
    }
  ]
}
```

`rules` 는 규칙 개체의 배열을 갖는 JSON 문서의 규칙 컬렉터로 필터집합과 작업 집합으로 이루어져있다. 

일반적으로 하나의 *rule*의 경우 이름을 뜻하는 `name` 과 규칙을 일시적으로 허용하지 않을 수 있는 `enabled` , 열거형 값인 `type`, 정의하는 개체의 `definition` 으로 구성되어진다.

### Blob 규칙

아래에 규칙 정의 집합이 있다.

```yaml
{
  "rules": [
    {
      "enabled": true,
      "name": "sample-rule",
      "type": "Lifecycle",
      "definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
        "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "sample-container/blob1"
          ]
        }
      }
    }
  ]
}
```

**fileters**

```yaml
     "filters": {
          "blobTypes": [
            "blockBlob"
          ],
          "prefixMatch": [
            "sample-container/blob1"
          ]
        }
```

`blobTypes` : Blob의 유형 

`prefixMatch` : Blob 이름이 특정 접두사로 시작하는 경우의 규칙

위의 예시는 Blob이름이 `sample-container/blob1` 으로 시작하면서 *blockBlob*인 경우 선택된다.

**definition**

```yaml
"definition": {
        "actions": {
          "version": {
            "delete": {
              "daysAfterCreationGreaterThan": 90
            }
          },
          "baseBlob": {
            "tierToCool": {
              "daysAfterModificationGreaterThan": 30
            },
            "tierToArchive": {
              "daysAfterModificationGreaterThan": 90,
              "daysAfterLastTierChangeGreaterThan": 7
            },
            "delete": {
              "daysAfterModificationGreaterThan": 2555
            }
          }
        },
```

`tierToCool` ” 덜 자주 사용되는 데이터를 Cool 계층으로 이동

`tierToArchive` : 거의 사용되지 않은 데이터를 Archive 계층으로 이동

`delete` : 삭제

`enableAutoTierToHot` : 쿨에서 핫 계층으로 이동 활성화

- **`daysAfterModificationGreaterThan`**
    - Blob이 마지막으로 수정된 지 얼마나 오래됐는지를 기준으로 판단.→ 주로 **기본 Blob**에 사용.
- **`daysAfterCreationGreaterThan`**
    - Blob이 생성된 지 얼마나 오래됐는지를 기준으로 판단.→ 주로 **스냅샷**에 사용.
- **`daysAfterLastAccessTimeGreaterThan`**
    - Blob에 마지막으로 액세스한 지 얼마나 오래됐는지를 기준으로 판단.→ 액세스 추적이 활성화된 경우에 사용.
- **`daysAfterLastTierChangeGreaterThan`**
    - Blob 계층이 마지막으로 변경된 지 얼마나 오래됐는지를 기준으로 판단.

### Archive 계층의 Blob 수정

Blob이 일반적으로 Archive 계층에 있다면 읽거나 수정이 불가능하다. 이 Blob을 수정하기 위해서는 Hot 또는 Cool 계층으로 올린 뒤에 수정해야한다. 이를 수정하는데 있어서 **URL등을 이용해 Blob을 복사**하거나 **Blob 계층 설정**을 통해 상위 계층으로 올린 뒤에 수정해야한다. 

**두 가지 방법의 수정법**

Archive 계층에 있는 Blob을 상위의 계층에서 **완전히 새로운 Blob**으로 복사한다.  이럴 경우 원본 Blob은 **변경되지 않으며** 덮어쓰는것 조차 불가능하다.

또는 Archive 계층의 원본 Blob을 상위 계층으로 **직접 변경**하는 방식은 **취소가 불가능**하며 이러한 계층 변경 작업은 **마지막 수정시간**에 영향을 주지 않는다.
이렇게 상위의 계층으로 옮기는 것을 **리하이드레이션**이라고 한다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/image22.png)

### 리하이드레이션의 시간과 우선 순위

**표준 우선 순위**

- Queue 형태로 처리된다.

**높은 우선 순위**

- 더 빠르게 처리 되며 10GB미만 데이터는 약 1시간 이내로 처리 된다.

이러한 우선 순위는 요청중 `x-ms-rehydrate-priority` 헤더 값을 확인하여 확인이 가능하다.

이러한 리하이드레이션은 취소할 수 없으며, 작은 Blob 여러개를 동시에 하는것보다 큰 Blob을 리하이드레이션 하는것이 효율적이다.


### Change Feed

Blob Storage에서 모든 변경사항을 추적하고 모니터링 할 수 있는 기능이다 (`BlobServiceClient`나 `BlobContainerClient`를 이용한다)
일반적으로 순서를 보장하지 않은 Storage에서 발생한 순서대로 자동으로 나열되며 비동기 처리를 지원한다.

주로 기업에서 법적 요구사항을 달성하기 위해 감사로그를 분석하는데 주로 사용된다.

### Storage 권한

일반적으로 Azure Storage 서비스에 사용되는 엑세스 제어 방식으로는 **SAS(공유 엑세스 서명)**이 있다. 
제한된 시간동안 계정 키 없이도 읽기,쓰기,삭제 등의 권한을 부여하며 일반적으로 URL 형태로 제공된다.

SAS로 접근을 하는 방식 이외에도 스토리지 게정 키, 관리 ID, RBAC, Entra ID등으로 권한을 조절할 수 있다.