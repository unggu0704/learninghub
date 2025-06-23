---
author: "unggu"
title: "[CKA] 쿠버네티스 네트워킹"
date: 2025-05-08 11:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker, Pod, Storage]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
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

`unggu.xyz`→ `142.250.206.46`

라면 `142.250.206.46` 가 A-레코드라고 불린다. 

**CNAME 레코드**

[`unggu.xyz`](http://unggu.xyz) → `unggg0704.github.io`

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