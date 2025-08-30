---
author: "unggu"
title: "[CKA] 컨테이너 운영"
date: 2025-05-01 11:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

### 컨테이너 모니터링

클러스터 운영에 있어 리소스 사용량 또는 문제 해결을 위해 모니터링을 사용할 경우가 있다. kubernetes는 이러한 모니터링을 지원하기 위해 **경량 모니터링 도구(Metrics Server)**를 지원하는데 `kubectl top`명령어를 사용하게 해준다. 

**설치 여부 확인**

```yaml
kubectl get deployment metrics-server -n kube-system
```

Metric Server를 사용해 node나 pod의 메모리, CPU 사용량 등을 확인할 수 있다.

```yaml
kubectl top node           # 노드들의 CPU, 메모리 사용량 확인
kubectl top pod --all-namespaces  # 모든 파드의 사용량 확인
kubectl describe pod <pod-name>  # 파드 상세 정보 (이벤트, 상태 등 포함)
kubectl get events         # 클러스터 이벤트 확인 (CrashLoopBackOff 등)
```

이러한 경량 모니터링 도구이외에도 **프로메테우스**(오픈소스 모니터링툴), **Grafana**(프로메테우스의 시각화 툴) 등이 있다.

## k8s 배포

### Rolling Update

기존의 pod를 종료하고 새로운 pod를 생성하며 배포하는 **무중단 배포**를 위해 사용되는 개념 

**설정 `yaml`**

일시적으로 라도 비활성화되는 pod는 없어야하고 새로운 버전의 pod가 기동되어야지, 이전 pod가 삭제 된다.
```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          
    maxUnavailable: 0    
```

`maxSurge`는 새 버전의 pod를 몇 개까지 더 추가할지를 뜻하며, `maxUnavailable`은 롤링 업데이트 중 사용할 수 없는  pod가 몇개인지를 설정한다. 

**deployment 업그레이드 하기**
`kubectl set image deployment/nginx-deploy nginx=nginx:1.17`

### Rollback

최근 업그레이드된 Deploy를 이전버전으로 되돌리는 작업으로 `rollout`을 통해 배포 실패나 장애 발생시 유용하다.

```yaml
kubectl rollout undo deployment myapp-deployment
```

만약 특정 revision으로 돌아가고 싶다면

```yaml
kubectl rollout undo deployment myapp-deployment --to-revision=2
```

으로 특정 리비전으로 돌아갈 수 있다.

### Multi-Container pod

하나의 pod안에 둘 이상의 컨테이너가 존재하는 multi-container pod는 하나의 컨테이너 life cycle과 자원을 공유한다.  

이런 방식은 **Sidecar**(보조 컨테이너 로그, 프록시), **Adapter**(데이터 변환)의 패턴에서 주로 사용된다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/csmimage.png)

**Multi-Conatiner `yaml`**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: multi-container-pod
spec:
  volumes:
  - name: shared-logs
    emptyDir: {}  # 공유 스토리지

  containers:
  - name: main-app
    image: nginx
    volumeMounts:
    - name: shared-logs
      mountPath: /usr/share/nginx/html

  - name: log-writer
    image: busybox
    command: ["sh", "-c", "while true; do echo hello >> /logs/hello.txt; sleep 5; done"]
    volumeMounts:
    - name: shared-logs
      mountPath: /logs
```

여기서 추가된 `emptyDir`는 컨테이너 간의 파일 공유에 사용되며 각 컨테이너는 `localhost`를 통해 내부적으로 통신이 가능하다.

### Probe

> *살아있어요? 컨테이너씨*
> 

kubernetes에서 컨테이너의 상태를 체크하기 위해 사용되는 개념으로 `HTTP GET` (*특정 URL 접근*)방식이나 `TCP Socket` (*특정 포트 연결 테스트*) 같은 방식으로 컨테이너의 health를 체크한다.

컨테이너가 가지고 있을 수 있는 상태는 총 3가지인데 아래 표와 같이 크게 정리된다.

| Probe 종류 | 목적 | 실패 시 동작 |
| --- | --- | --- |
| **Liveness Probe** | "살아있냐?" 확인 | 실패하면 **컨테이너를 재시작** |
| **Readiness Probe** | "트래픽 받을 준비 됐냐?" 확인 | 실패하면 **트래픽을 안 보냄** (Pod는 Running이어도 Ready 아님) |
| **Startup Probe** | "앱 시작됐냐?" 확인 (초기 부팅 오래 걸리는 앱용) | 실패하면 **컨테이너를 재시작** |

**Liveness Probe**

컨테이너가 무한 루프 등을 통해무응답 상태에 빠졌을 경우 컨테이너가 죽었다고 판단하여 재시작 정책을 시도한다. 

```yaml
livenessProbe:
  httpGet:
    path: /healthz
    port: 8080
  initialDelaySeconds: 5
  periodSeconds: 10
```

`/healthz`를 8080포트 최초 시작 5초후 호출하며 10초 간격으로 체크한다. 

**Readiness Probe**

컨테이너가 준비되었을 때 Service 엔드포인트에 등록하고 만약 컨테이너가 적절하게 준비되지 않았다면, 등록하지 않아 Service가 애초에 트래픽을 보내지 않게 한다. 

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/ready
  initialDelaySeconds: 5
  periodSeconds: 5
```

`/tmp/ready` 파일이 존재하면 준비된 것으로 간주한다.

**Startup Probe**

일부 컨테이너에서는 어플리케이션이 느리게 기동될 수도 있다. 이럴 경우 시작이 완료되기전 까지 Liveness/Readiness Probe를 무시한다.

```yaml
startupProbe:
  httpGet:
    path: /startup
    port: 8080
  failureThreshold: 30
  periodSeconds: 10
```

`/startup`을 호출하는데 30번 실패하면 컨테이너가 죽었다고 판단한다.