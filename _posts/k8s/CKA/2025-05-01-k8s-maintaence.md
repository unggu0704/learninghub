---
author: "unggu"
title: "[CKA] 컨테이너 운영 및 네트워크"
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

```yaml
strategy:
  type: RollingUpdate
  rollingUpdate:
    maxSurge: 1          
    maxUnavailable: 1    
```

`maxSurge`는 새 버전의 pod를 몇 개까지 더 추가할지를 뜻하며, `maxUnavailable`은 롤링 업데이트 중 사용할 수 없는  pod가 몇개인지를 설정한다. 

### Rollback

최근 변경된 Deploy를 이전버전으로 되돌리는 작업으로 `rollout`을 통해 배포 실패나 장애 발생시 유용하다.

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

![image.png]({{ site.baseurl }}{{ page.url }}/img/km0.png)

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

## k8s Upgrade

k8s cluster를 업그레이드 하는데 있어, 다양한 환경에 따라 방법이 달라진다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/km1.png)

1. GCP를 사용한다면 쉽고 빠르고 간편하다! (안알아봄)
2. kubeadm을 사용하기(배운 것)
3. 혼자 하기(현재 수준이 아님)

일반적으로 GCP를 사용하거나, `kubeadm`을 사용하여 업그레이드 한다고 한다.

CKA 시험 및 순수 k8s 환경에서 주로 사용하는 `kubeadm`을 활용한 업그레이드에 대해 알아보자.

**Master Node 업그레이드 하기**

![image.png]({{ site.baseurl }}{{ page.url }}/img/km2.png)

MasterNode에는 서비스에 기동되는 pod가 존재하지 않는다. 비록 *controlplane*같은 관리형 pod가 존재하지만 이런건 사소하다. 

```yaml
	$ kubectl drain master --ignore-daemonsets --force
	$ apt-get upgrade -y kubeadm=1.13.0-00
	$ apt-get upgrade kubelet=1.32.0-00
	$ systemctl restart kubelet
```

`kubeadm`과 `kubelet`을 업그레이드 함으로써 master Node의 업그레이드가 완료되었다.

> Master Node에서 drain을 해야하는 이유?
Node를 업그레이드하면서 kubelet을 비롯한 static pod들 또한 재기동 되어진다.
> 

**Worker Node 업그레이드 하기**

Worker Node는 실제 서비스가 가동되어지는 Pod가 있는 Node이다. 업그레이드는 즉 일시적인 서비스 다운을 의미하기에 Node Upgrade 전략을 확인할 필요가 있다.

만약 Node에 다양한 Pod가 기동된 상태에서 Node가 작동불가 상태가 된다면 시스템은 *5분*간 해당 Node에 Pod들을 유지한다.

그 시간동안 replica가 관리하는 pod들은 다른 node로 재기동하지 않고 대기하고 있지만, 해당 시간이 초과된다면 replica로 관리되는 pod들은 다른 node에서 새롭게 재기동 되어진다. 단, 일반 pod들은 삭제되고 완전히 사라진다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/km3.png)

만약 Node의 정비를 이유로 해당 Node가 일정시간 서비스 지장 없이 무결한 상태로 중지될 필요가 있다면? `drain`명령어를 사용한다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/km4.png)

`drain`명령어 사용시 deployment가 관리하지 않아도 해당 node의 pod들은 다른 node에서 재기동 되어지며, 해당 node에는 스케줄러가 pod를 스케줄링하지 않는다. 

`drain`명령어 이외에도, `cordon` 명령어와 `uncordon` 이 있는데 `cordon`은 해당 node에 pod들을 지금 당장 옮기지는 않지만 새로운 pod가 이 node에서 스케줄링 되지 않도록 설정하며 `uncordon`은 `drain` 또는 `cordon` 상태여 스케줄링 되지 않은 node를 다른 pod들도 스케줄링 가능하도록 lock을 해제하는 명령어이다.

이를 바탕으로 worker node를 업그레이드 하는 방식에는 하나의 Node를 `drain`시키고 `kubeadm`과 `kubelet`을  업그레이드 하는 방식이 주로 사용된다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/km5.png)

업그레이드가 정상적으로 끝났다면 `uncordon`을 반드시 해줘야한다.

> *worker node 하나를 down 시키고 하는것보단, 새로운 업그레이드 된 Node를 하나 기동시키고 하나의 구 node를 `drain` 시켜서 점진적으로 새로운 Node들로 대체되게 하는 방식도 존재한다!*
>


---

### k8s에서의 네트워크

일반적인 물리적 장비가 없는 pod들은 클러스터 내 어디서든지 서로 통신이 가능해야한다.(**라우팅 가능**) 이러한 내부 통신을 위해 DNS라는 개념 특히 **CoreDNS**라는 개념을 사용하는데 pod 내부에서는 `svc.cluster.local` 형식으로 서비스 이름을 조회 가능하다. 이런 클러스터에 `kube-dns`라는 이름으로 서비스가 등록되어져 있다.

일반적으로 Docker에서는 `bridge`라는 개념을 사용하는것과 달리  **CNI 플러그인**을 사용한다. 

이런 CNI 플러그인은 *Calico*, *Flannel* 등 다양한 종류가 있는데 공통적으로 pod의 ip 할당 및 라우팅 규칙 추가의 역할을 담당한다.

| CNI | 특징 | 구조 방식 | 장점 | 단점 |
| --- | --- | --- | --- | --- |
| **Flannel** | 가장 단순, 많이 쓰임 | VXLAN, host-gw | 설치 간편 | 정책 제어 없음 |
| **Calico** | 고급 기능, 실무 선호 | BGP, IP-in-IP | 네트워크 정책, 퍼포먼스 좋음 | 복잡함 |
| **Weave** | 자동화 & 간편함 | VXLAN + 암호화 | 설치 간단, 암호화 가능 | 성능 이슈 (VXLAN 오버헤드) |

### Pod 네트워킹

 pod끼리 통신은 가능하다(심지어 서로 다른 노드에 있어도!) 이런 구현을 위해 k8s에서는 `veth`라는 가상의 이더넷 페어로 pod ↔ CNI(`cni0`) 연결을 한다. 이 `cni0`은 같은 node안에서는 직접 다른 노드는 라우팅을 통해 통신을 제공한다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/knimage.png)

즉 **CNI**는 pod 통신에 있어 자동으로 라우팅 정보나 터널을 구성해준다!

## Service Networking

Pod IP는 동적이다. Pod가 죽고 재시작한다면 IP주소가 바뀐다.(동적 IP) 이로인해 Pod의 IP를 통해 통신하는것은 비효율적이기에 **Sercice**를 통해 통신을 한다. 

**Cluster IP**는 Service가 사용하는 기본 타입으로 동적 IP가 아닌 **고정 IP**를 사용한다.  그리고 각 Pod마다 배정(*kube-proxy*)되어 LoadBalancer 역할을 수행한다. 하지만 Cluster-IP는 Node 내부 통신용으로 Node 외부에서는 접근할 수가 없다

**Cluster IP**

```yaml
**apiVersion: v1
kind: Service
metadata:
  name: local-cluster
spec:
  ports:
  - port: 80
    targetPort: 80
  selector:
    app: nginx** 
```

**Kube-proxy**는 Pod와 Svc를 연결하는 역할을 한다. 일반적으로 **iptables**나 **ipvs**를 이용해서 연결을 관리하는데, Cluster IP로 요청시 kube-proxy가 table 기반으로 Pod로 라우팅 해주는 형식이다. 

**Service 생성하기**

```yaml
kubectl expose deployment myapp --port=80 --target-port=8080 --name=myapp-service
```

### k8s DNS

네트워크 상에서 목적지를 찾아가기 위해 사용되는 것인 IP와 Port 정보이다. 그리고 DNS는 해당 도메인을 IP + Port로 해석하게 해주는 네트워크의 필수적인 개념 중 하나이다. 

**A 레코드**

Address Record라고 불리는 이 식은 도메인 이름을 IP 주소로 바꿔주는 것으로 

`unggu0704.github.io/learninghub/`→ `142.250.206.46`

라면 `142.250.206.46` 가 A-레코드라고 불린다. 

**CNAME 레코드**

[`unggu0704.github.io/learninghub/`](http://unggu0704.github.io/learninghub/) → `unggg0704.github.io`

특정 도메인에 대한 alias로 IP + Port를 알고 싶으면 이 친구에게 물어보라는 방식이다.

**PTR 레코드**

IP → 도메인의 역방향 이다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/coredns.png)

k8s에서는 하나의 클러스터 안에서 이러한 개념들을 사용하는 바로 **CoreDNS**서비스이다. 

> *CoreDNS는 `kube-system` 네임스페이스 안에 **Deployment** 형태로 실행되고 있다.*
> 

이를 통해 Pod들은 서로 도메인 이름으로 통신할 수 있는데 주로 아래와 같이 이름이 지어진다.

```yaml
http://my-service.default.svc.cluster.local
```

이 `url`을 하나하나 뜯어내면 `my-service`는 서비스 이름 `default`는 네임스페이스 `svc`는 서비스 그리고 `cluster.local`은 해당 svc의 상위이다.

이러한 DNS 이름은 모든 **Service**가 가지고 있다.

만약 해당 URL의 ip를 알고 싶다면 `nslookup`이 유용하다.

```yaml
nslookup my-service
```

## Ingress

![image.png]({{ site.baseurl }}{{ page.url }}/img/knimage%201.png)

일반적으로 하나의 pod-service 사이에서 서비스를 제공하기 위해서 Load Balancer가 사용된다. 하지만 다양한 Service가 생기고 요구사항에 따라 분류될 수 있는 수십개의 Service가 생긴다면 이를 관리하기도 어려울 뿐더러 L4의 갯수 증가로 인해 클라우드 비용이 높게 측정될 수 있다. 

하나의 클러스터 내에서 L4 앞에서 이런 분산을 관리해주는 L7의 성질을 가진 오브젝트로 **Ingress**가 있다.

일반적인 L7의 기능에서 경로 기반과 호스트(URL)기반 라우팅과 SSL/TLS 같은 기능 지원이 되는 특징이 있다.

### Ingress Controller

Ingress 객체는 스스로 동작하지 않는다. 언제나 이를 처리해줄 **Ingress Controller**가 필요하다. 주로 `nginx Ingress Controller` 가 인기가 많으며 `Trafik`, `Istio`등이 대표적이다. 

아래에는 Niginx Ingress Controller를 사용한 Ingress yaml이 있다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: example-ingress
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  ingressClassName: nginx        # IngressClass 이름 참조
  rules:
  - host: example.com
    http:
      paths:
      - path: /app
        pathType: Prefix
        backend:
          service:
            name: my-web-service
            port:
              number: 80
```

하나의 서비스만 들어가 있다고 가정한 Ingress이다. `host`에는 서비스의 url 정보를 입력하고 그 뒤 `paths`에는 요청 경로를 적는다. 특히 `pathType`에는 `Prefix`또는 `Exact`를 입력하는데 `Prefix`의 경우 앞에만 맞으면 뒤에 어떠한 추가 경로가 와도 매칭해준다. `/app/bar/...` 도 매칭한다는 소리! 

`Exact`의 경우 `/app`만 매칭한다.  CKA 시험에서는 반드시 `pathType`을 지정해야한다고 한다.

**여러개의 host를 가진 라우팅**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-host-ingress
spec:
  ingressClassName: nginx        # IngressClass 이름 참조
  rules:
  - host: app1.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-service
            port:
              number: 80
  - host: app2.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-service
            port:
              number: 80
```

host의 종류에 따라 받는 service의 트래픽이 달라진다!

### Ingress와 TLS

대부분의 사용자 접속은 HTTPS로 접속을 통해 이루어진다. 이를 위해 TLS 인증서를 필요하며 k8s에서는 이런 인증서(Secret)를 Ingress에서 로드하여 HTTPS 통신에 사용할 수 있다. 

먼저 아래와 같은 `tls`타입의 Secret이 있다고 가정

```yaml
kubectl create secret tls tls-secret \
  --cert=path/to/tls.crt \
  --key=path/to/tls.key
```

그리고 아래와 같이 기존의 Ingress에 tls secret을 적용한다.

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: tls-ingress
spec:
  tls:
  - hosts:
    - example.com
    secretName: tls-secret
  rules:
  - host: example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: my-service
            port:
              number: 80
```

당연하게 `secretName`등은 이전에 만든 secret과 이름이 일치해야한다. 

이렇게 설정하면 Ingress Controller는 TLS Secret을 지속적으로 감시하고 만약 변경시에는 자동으로 로딩되어 새로 갱신하는 작업을 진행한다.

## GateWay API

단일 진입점이 되는 **Ingress**를 멀티 테넌시를 지원하는 **GateWay**, **HTTPRout**, **GatewayClass**등으로 분리하고  HTTP,HTTPS를 넘어 TCP, UDP를 관리한다. 

**GateWayClass**
해당 Gateway가 어떤 종류의 Gateway인지 정의한다. Nginx, istio 등등..

**GateWay** 
실제 네트워크가 진입하는 리소스

**HTTPRoute**
URL 경로 및 호스트 기반하여 라우팅을 정의한다.

주로 GateWay + HTTPRoute를 사용하여 서비스를 라우팅한다.

GateWay.yml
```yml
apiVersion: gateway.networking.k8s.io/v1
kind: Gateway
metadata:
  name: example-gateway
spec:
  gatewayClassName: example-class
  listeners:
  - name: http
    protocol: HTTP
    port: 80
```

HTTPRoute.yml
```yml
apiVersion: gateway.networking.k8s.io/v1
kind: HTTPRoute
metadata:
  name: example-httproute
spec:
  parentRefs:
  - name: example-gateway
  hostnames:
  - "www.example.com"
  rules:
  - matches:
    - path:
        type: PathPrefix
        value: /login
    backendRefs:
    - name: example-svc
      port: 8080
```

### Storage Class

> 동적 volume provisoning을 제공 PVC 생성시 자동으로 PV를 만들어준다.

기존 관리자가 PV를 만들면 사용자나 시스템이 PVC를 만들어 적합한 PV <-> PVC 바인딩을 진행하였다. 

하지만 StorageClass는 사용자가 SC를 통해 PVC를 만들면 StorageClass가 자동으로 적합한 PV를 생성과 동시에 이를 바인딩하는 기능을 제공한다.

**예시**
```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: my-storage-class
provisioner: kubernetes.io/no-provisioner  # 중요!
parameters:  # provisioner별로 다름
  type: gp2
reclaimPolicy: Delete  # Delete, Retain, Recycle
allowVolumeExpansion: true  # 볼륨 확장 허용
volumeBindingMode: Immediate  # Immediate, WaitForFirstConsumer
```
yml 파일 중 `provisioner`에 대해서 AWS나 Azure 같은 GCP에 의해 달라진다.


---

#### Exercise

`local-sc`라는 StorageClass를 만들어야한다. 이에 대한 명세(Specification)은 아래와 같다. 

- provisioner는 `kubernetes.io/no-provisioner`여야한다. 
- `WaitForFirstConsumer` Volume과 바인딩 되어야한다. 
- Volume의 확장은 허용되어야한다.


**Solution**
```yml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-sc
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
allowVolumeExpansion: true
```