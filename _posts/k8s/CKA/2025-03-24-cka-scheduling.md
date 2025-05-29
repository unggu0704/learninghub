---
author: "unggu"
title: "[CKA] 쿠버네티스 스케줄링"
date: 2025-03-24 21:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker, kube-scheduler]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

본 글은 [Udemy Certified Kubernetes Administrator (CKA)](https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/?couponCode=KEEPLEARNING) 강의를 참조해 정리한 내용을 기록했습니다.

### Manual Scheduling

앞서 배운 `kube-scheduler`는 자동으로 pod를 적절한 노드에 배치한다. 

![image.png]({{ site.baseurl }}{{ page.url }}/img/simage.png)

*어떤 Node가 적합한지 점수를 측정해 판단하는 `kube-scheduler`*

하지만 실제 운영 환경 구축을 위해 어떠한 Node가 정상적으로 해당 Pod를 실행할 수 있는지 또는 Node↔Pod에 있어 호환성 등을 테스트 하기 위해 특정 Node에 Pod를 수동으로 배치하는 **디버깅** 목적으로 필요할 수 있다.

이럴 때 사용되는 것이 **Manual Scheduling**으로 `kube-scheduler`로 인해 배치하지 않고 사용자가 직접 Node를 선택해야한다.

이때 사용할 Node는 `nodeName` 으로 지정한다.

**Manual Scheduling을 사용는 경우 YAML**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: manually-scheduled-pod
spec:
  nodeName: worker-node-1  # 특정 노드에 직접 할당
  containers:
  - name: nginx
    image: nginx
```
이렇게 Node를 직접 이름으로 지정하는 방법 이외에도 k8s는 다양한 방법을 지원한다. 아래에서는 어떤식으로 k8s에서 pod의 Node 할당을 제어하는지 알아보겠다.

### Taints & Tolerations

Taints는 *오염* 이라는 뜻을 가지고 Tolerations는 *관용*이라는 뜻을 가진다. 왜 쿠버네티스를 공부하는데 이러한 개념에 굳이 이러한 명칭을 가지는지는 잘 모르겠지만 이러한 개념들은 특정 Taints(오염) 된 Node에 Toleration(관용)되지 않은 Pod가 들어가지 않게 하는… 조금 더 쉽게 표현하자면 특정 Node에 Pod rule을 추가하는 것으로 의미할 수 있겠다.

이제부터 이러한 오염과 관용을 어떤식으로 제어할 수 있는지 알아보도록 하겠다. 

**Taints를 Node에 적용하기**

taints를 적용하기 위해서는 Node에 명령줄을 추가 하여 적용하는다 일반적인 형식은 아래와 같다.

```jsx
kubectl taint nodes <노드명> <키>=<값>:<효과>
```

 일반적으로 `키:값` 형태로 매핑을 하고 이와 같은 Node가 있을 경우 `<효과>` 의 형태를 가지게 설정한다.

**Tolerations를 Pod에 적용하기**

더렵혀진 Node(taintsed Node)에 할당 되기 위해서는 pod의 `spec` 에 아래와 같은 `toleration`항목을 추가해줘야한다.

```jsx
tolerations:
- key: "<키>"
  operator: "Equal"  # 또는 "Exists"
  value: "<값>"
  effect: "<효과>"
```

여기서 `operator`는 `Equal` 또는 `Exists`를 사용할 수 있으며, 각각 키와 값이 일치하거나 키만 존재하는 경우를 의미한다.

**Taint가 Blue이고 Node의 Value에 Blue를 설정 한 경우**

![image.png]({{ site.baseurl }}{{ page.url }}/img/simage%201.png)

`kube-scheduler`는 `Key=blue` 가 아닌 Pod은 Node1에 할당하지 않고 tolerations가 blue인 Pod D만 Node1에 할당 한 것을 볼 수 있다.

**이걸 왜 쓰는거지?**

작업을 하다보면 특정 워크로드를 Node에 설정할 때가 있다. 예를 들어 데이터베이스 수정 같은 작업을 특정 노드에서만 할당 했다면 이 작업을 시작할 Pod는 toleration을 설정이 가능하다. 

또한 이외에도 Node를 유지보수 한다거나(디버깅), 특정 Pod를 격리 시킬 때 이러한 개념을 사용할 수 있다.

## Node Selector

pod는 별다른 제약이 없는한 어떠한 Node로도 할당 될 수 있다. 만약 Node간의 성능이 다르고 사양이 높은 pod가 저 사양의 Node에 할당이 된다면 서비스 운영에 제약이 발생할 수 있다. 

이를 제한하기 위해 `nodeSelector`라는 개념을 사용해서 Key-Value 형식으로 어떠한 Node에 pod를 고정할지 지정이 가능하다.  

![image.png]({{ site.baseurl }}{{ page.url }}/img/simage%202.png)

일반적으로 Node에 `label`을 붙이기 위해서는 아래와 같은 형식을 따른다.

```yaml
kubectl label nodes <node-name> <label-key>=<label-value>
```

*예시*

```yaml
kubectl label nodes node-1 size=Large
```

이러한 방식은 간단하고 쉽게 도달되지만 더 높은 요구사항을 높이기 위해 **Node Affinity**라는 개념이 등장한다.

### Node Aiffinity

label 기반 스케줄링 결정에 있어 강력한 효과를 낸다. 기존의 **Node Selector**에 비해 표현식(expression)을 사용해 더 복잡한 규칙을 만들 수 있다.

**예시**

```yaml
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: zone
            operator: In
            values:
            - us-east1

```

- **`requiredDuringSchedulingIgnoredDuringExecution`** (필수 규칙)
    
    pod가 스케줄링 되어질 때 **반드시 해당하는 규칙의 Node에만 할당된다.**
    
    위의 예제에서는 `disktype` ****이 `ssd`인 node에만 스케줄링이 되도록 지정한다.
    

- **`preferredDuringSchedulingIgnoredDuringExecution`** (선호 규칙)
    
    pod에 이러한 규칙이 **우선적으로 적용되지만 반드시 만족하지 않아도 할당된다.**
    
    `zone` 이 `us-east1` 인 node를 우선적으로 선호한다.
    

- **Operator (연산자)**
    
    `In`: 지정한 값 중 하나와 일치
    
    `NotIn`: 지정한 값 중 어느 것도 일치하면 안된다.
    
    `Exists`: 지정한 Key가 Node에 존재해야 한다.
    
    `DoesNotExist` : 지정된 Key가 Node에 없어야한다.
    

Node Affinity의 동작 방식은 **Pod가 스케줄링 될 때**에만 적용된다는 것을 명심하자. 즉 Pod가 한번 Node에 배치가 된다면 `label` 이 변경되더라도 Pod는 지속적으로 실행된다.  또한 이러한 규칙의 우선순위에 따라 스케줄링 방식이 꼬일 수 있음을 인지하고 요구 사항에 맞게 적절하게 조합해야 한다.

### Taints & Tolerations vs Node Affinity

해당 개념들은 모두 Pod의 스케줄링을 제어하는 기능으로 각자 다른 방식으로 동작한다.

**Taints & Tolerations**

- 특정 Node에서 Pod를 거부하거나 허용함

**Node Affinity**

- Pod의 배치에 있어 **선호도**를 사용 가능
- Label을 통해 제한함

일반적으로 Taints & Tolerations는 **특정 Node에서 Pod를 퇴출**하는데 사용되고 **Node에 특정 Pod를 배치**하는데는 Node Affinity가 선호된다.

### Multiple Schedulers

**kube-scheduler**가 Node에 pod를 할당시키지만 **하나 이상의 스케줄러**도 실행할 수 있다!
또한 pod의 `yaml`에 `spec.schedulerName` 필드를 사용하면 특정 스케줄러를 고를 수 도 있다. 

**pod에서 특정 scheduler를 지정**

```yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-custom
spec:
  schedulerName: my-scheduler
  containers:
  - name: nginx
    image: nginx
```

my-scheduler를 만드는 방법은 기본으로 존재한 `kube-scheduler`를 복사하여 만들 수가 있다.  
기존의 yml을 복사하여 수정하는 방법
```yml
sudo cp /etc/kubernetes/manifests/kube-scheduler.yaml /etc/kubernetes/manifests/my-scheduler.yaml
```
이렇게 별도로 복사해서 아래와 같이 수정하면된다. 이렇게 해서 배포하면 `kubelet`은 이걸 **static pod**으로 감지하여 실행시킨다.

```yml
spec:
  containers:
  - command:
    - kube-scheduler
    - --scheduler-name=my-scheduler
    - --leader-elect=false        # 하나만 띄울 거면 이 옵션을 false로
    - --config=/etc/kubernetes/scheduler-config.yaml

```

## Admission Controller
![image.png]({{ site.baseurl }}{{ page.url }}/img/admission_controller1.png)

`get`이나 `create` 명령을 수행하는데 있어, 이 요청을 검사 또는 수정하는 k8s 플러그인 중 하나이다. **API Server** 내부에서 실행되는 Admission Controller는 **web hook**을 통해 사용자의 요청을 검사(*인증 또는 권한을 확인*)하여 **거부** 또는 **수정**을 할 수 있다.

사용자가 요청을 명령을 요청할때 순서는 아래와 같다.
```
Client → API Server → Authentication → Authorization → Admission Controller → etcd 저장
```
여기서 Admission Controller는 리소스가 etcd에 저장되기전 최종 검증자 역할을 실행한다.

### Admission Controller의 유형

 **Validating Admission Controller** - 리소스를 유요한지 검증만

 **Mutating Admission Controller** - 리소스를 검증하고 변경도 가능 

이를 통해 외부 웹서버(WebHook)과 통신하여 요청을 조작 및 감시가 가능하다.
