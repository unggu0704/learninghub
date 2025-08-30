---
author: "unggu"
title: "[CKA] 쿠버네티스 보안과 접근 관리"
date: 2025-04-13 11:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker, SA, RBAC]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

### TLS in Kubernetes

통신 데이터를 암호화해 인증을 보장해주는 보안 프로토콜 TLS는 k8s에서의 내부 통신에서 주로 사용된다. 
kube-apisever, kubelet, ETCD 등등... control-plane의 구성요소들과 k8s의 작업을 위해 통신시 사용되어지고 있다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/k8s_tls.png)

일반적으로 인증서는 `/etc/kubernetes/pki` 경로에 저장되며 추후 trouble shooting이나 인증서 관련 Node 이슈가 발생하면 해당 경로를 조사해보면 된다.

**인증서의 유효기간 확인하기**
```yaml
kubeadm certs check-expiration
```

유효기간 만료 이외에도 누락, 설정(`kubectl config`) 등 다양한 인증서 관련 이슈로 TLS 관련 시나리오가 출제된다.

### Authentication

k8s에서 인증은 사용자가 누구인지 확인하는 단계로 클러스터에 요청한 사람의 신원을 확인하는 방식이다. 
이러한 사람의 신원에 대한 정보는 `kube-config`에 저장되어 있으며, 권한을 부여 받으며 클러스터에 접근하여 작업(`get pods`, `delete secrets`등)을 할 수 있도록 **역할**을 지정한다. 

이러한 권한을 주는데는 주로 **RBAC** 방식을 사용한다.

**권한을 확인하기**

```yaml
kubectl auth can-i <verb> <resource> [flags]

kubectl auth can-i get pods # get pods를 할 수 있는 사용자 목록을 원합니다.
kubectl auth can-i delete secrets --namespace=dev # dev ns에서 secret을 지울 수 있는 사용자들을 원합니다.
kubectl auth can-i list nodes --as=dev-user dev-user # dev-user는 node list를 나열 할 수 있는지요?
```

`get pods`를 할 수 있는 사용자는 누구인가? `--as` 특정 사용자로 어떤 node에 접근할 수 있는지 시뮬레이션 하며 `--namespace`에 대한 권한을 확인한다. 

### RBAC이란?

k8s에서 리소스에 대한 접근을 역할(Role)기반으로 정의하고 사용자나 서비스 계정에 연결하는 시스템이다. 

단순히 k8s에서만 사용되지 않고 Azure 같은 곳에서도 사용되는 역할 개념이다.

**RBAC 구성요소**

k8s에서 RBAC의 구성요소는 크게 4가지 리소스로 구분된다. **Role** 같은 경우 특정 네임스페이스의 범위에 대한 권한을 정의하고, 이보다 더 큰 클러스터에 대한 권한은 **ClusterRole**에서 정의를 한다. 

또한 **RoleBinding**은 만든 Role을 사용자/서비스 계정에 연결하며, **ClusterRoleBinding**은 클러스터 단위로 Role을 연결한다. 

**Role 예시 `yaml`**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

`dev` 네임스페이스에 pod 조회 권한을 부여하는 Role
이런 Role을 만든다고 바로 사용자에게 연결이 되지는 않는다. **RoleBinding**으로 role을 연결 시켜줘야한다.

**RoleBinding 예시 `yaml`**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: dev
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

이렇게 RBAC을 설정하고 `auth`명령으로 확인 할 수 있다.

```yaml
kubectl auth can-i get pods --as jane -n dev
```

**ClusterRole 예시 `yaml`**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-reader
rules:
- apiGroups: [""]
  resources: ["nodes"]
  verbs: ["get", "list"]
```

**ClusterRoleBinding 예시 `yaml`**

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: read-nodes-binding
subjects:
- kind: User
  name: jane
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: node-reader
  apiGroup: rbac.authorization.k8s.io
```

### ServiceAccount

저번 Azure Key Vault를 pod에 연동시킬 때 사용했던 SA(ServiceAccount)가 다시 등장했다. 

그때와 동일하게 SA는 시스템(user가 아닌)이 role을 할당받을 수 있게 사용되는 개념이다. 

이런 SA는 일반적으로 네임스페이스 안에 디폴트적으로 생성되지만, 사용자가 원한다면 아래와 같이 SA를 생성 할 수 있다. 

**`my-sa`라는 sa를 네임스페이스 `dev`에 생성**

```jsx
kubectl create serviceaccount my-sa -n dev
```

이렇게 생성된 SA는 Pod에 적용할 수 있는데 pod의 `yaml`에 아래와 같이 적용하면된다.

**`my-sa`의 SA를 가진 pod**

```jsx
apiVersion: v1
kind: Pod
metadata:
  name: mypod
  namespace: dev
spec:
  serviceAccountName: my-sa
  containers:
  - name: app
    image: nginx
```

기본적으로 SA마다 하나의 `JWT` 토큰이 발급되고, 이러한 작업을 통해 pod에 자동으로 마운트되어진다.

이 pod가 API와 통신할때 누구인지 식별하는 수단이 되어진다. 

RBAC을 통해 접근을 제어할 경우  기존의 User의 RoleBinding과 동일하게 설정하여 pod에 권한을 넣어줄 수 있다. 

```jsx
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: ServiceAccount
  name: my-sa
  namespace: dev # group과는 달리 SA를 사용할때 ns는 무조건 지정해줘야함.
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

`kubectl get sa`를 통해 현재 존재하는 SA를 확인할 수 있으며, `describe` 명령어를 통해 SA에게 할당받은 토큰과 `secret`정보를 얻을 수 있다.