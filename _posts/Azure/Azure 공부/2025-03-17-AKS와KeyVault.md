---
title: AKS에서 Key Vault의 비밀 가져오기
author: "unggu"
date: 2025-03-17 19:12:42 +0800
categories: [Azure, Azure 공부]
tags: [Azure, AKS, Kubernets, Container, DevOps, Key Vault]
render_with_liquid: true
comments: true
image:
  path: https://azure.microsoft.com/svghandler/application-gateway/?width=600&height=315
---


![[Pasted image 20250312192324.png]]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20250312192324.png)

AKS에 컨테이너 기반 개발을 하다보면 비밀에 대한 고민을 많이 하게 된다.
중요정보 또는 개인정보를 `properties` 내부에 암호화를 하고 이를 복호화 할 수 있는 **Key**나 DB에 접속하기 위해 사용되는 **Username/Password**는 `Java` 파일이나 `yaml`에 저장하는것은 옳지 않다. 

그렇기에 이러한 **비밀**이라고 불리는 것들에 대해 저장할 필요성이 있다.

이러한 비밀들을 안전하게 저장할 수 있게 Azure에서는 **Azure Key Vault**라는 리소스를 제공하는데 지금부터 AKS에서 어떤식으로 Azure Key Vault의 비밀을 저장하는지 알아보겠다.

### Azure 비밀 만들기
![[Pasted image 20250312192942.png]]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20250312192942.png)

아래와 같이 비밀(DB 계정정보)를 저장했다고 가정한다.

### 관리 ID 만들기

**AKS 리소스에 할당된 관리 ID를 확인한다.**
```
az aks create \
  --resource-group <RESOURCE_GROUP> \
  --name <AKS_CLUSTER_NAME> \
  --enable-managed-identity
```

**이렇게 확인된 관리 ID에 Azure Key Vault에 대한 Secret 권한을 부여(RBAC 방식)**
```
az role assignment create --role "Key Vault Secrets User" \
  --assignee <AKS_MANAGED_IDENTITY_CLIENT_ID> \
  --scope /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP>/providers/Microsoft.KeyVault/vaults/<KEYVAULT_NAME>
```


### ServiceAccount

> *ServiceAccount는 Pod가 관리 ID를 사용할 수 있게 해주는 리소스*

pod가 일반적으로 Azure 리소스에게 직접 접근할 수 있는 권한은 없다. Azure에서는 리소스에 대한 접근 제어를 관리 ID(Managed Identity)를 통해 관리하는데 이런 관리 ID를 Pod에 매핑하기 위해 사용되는 리소스가  ServiceAccount 리소스이다. 

**일반적으로 사용되는 ServiceAccount 예제**
```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: my-service-account
  annotations:
    azure.workload.identity/client-id: "<AKS_MANAGED_IDENTITY_CLIENT_ID>"
```
`azure.workload.identity/client-id`의 값에 AKS가 사용할 관리 ID의 *Client ID*를 지정하여 SA 리소스에 관리 ID를 매핑하여 **Azure 관리 ID**처럼 사용 할 수 있게 제공한다!

**Pod에서 SA를 사용하기**
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  serviceAccountName: my-service-account
```



### SecretProviderClass
> *Azure Key Vault에서 어떤 비밀을 가져올까?*

SecretProviderClass는 **CSI(Secret Store CSI Driver)** 를 통해 Key Vault -> Pod의 볼륨으로 마운트하는 역할을 담당한다.  

그렇기에 주로 해당 pod가 어떤 Key Vault를 사용하며 어떤 관리 ID와 어떤 비밀을 사용할지를 정의한다.

즉 Pod가 직접 Key Vault에 접근하는 것이 아닌, SPC가 중간에서 Key Vault의 API를 호출하고 이를 Pod의 Volume에 저장하여 접근할 수 있게 한다.

**CSI를 사용하는 SecretProviderClass 예제**
```
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  serviceAccountName: my-service-account  # Managed Identity와 연결된 ServiceAccount 사용
  volumes:
    - name: secrets-store-inline
      csi:
        driver: secrets-store.csi.k8s.io
        readOnly: true
        volumeAttributes:
          secretProviderClass: "my-secret-provider"  # SecretProviderClass 연결
  containers:
    - name: my-container
      image: my-app-image
      volumeMounts:
        - name: secrets-store-inline
          mountPath: "/mnt/secrets"
          readOnly: true
```

**Pod의 동작 방식**

1. Pod가 실행되며 SecretProviderClass가 지정된 CSI 드라이버를 호출 
2. CSI 드라이버가 SecretProviderClass의 정보를 확인하고 Key Vault에 API 요청을 보낸다.
3. Key Vault에서 비밀을 가져온 후 Pod내의 `/mnt/secrets` 경로에 파일을 마운트 한다. 

### CSI란?

> *스토리지(볼륨)과 k8s간의 연결을 담당하는 플러그인(인터페이스)*

Pod의 비정상 상황에 있어, Pod가 재기동 되어질 때, Pod의 설정들은 휘발성이기에 데이터가 사라지게 된다.

기존 Pod에 `/data/a`라는 파일이 있는 상태가 있고 추가적으로 Pod가 운영 중에 `/data/b`가 생겼다고 가정하자.
![[Pasted image 20250317090442.png]]({{ site.baseurl }}{{ page.url }}/image/Pasted image 20250317090442.png)
어떠한 문제로 인해 Pod가 삭제되어진다면 기존에 저장한 `/data/b`는 유실되게 된다.

이러한 이슈를 해결하기 위해 **로컬 Node 저장소**를 이용하는 방법이 있지만 결국 Pod가 어떤 Node위에서 기동 될지는 `kube-scheduler`가 결정하기에 권장되는 방법이라고는 볼 수 없다.

이를 위해 Kubernetes는 Container간 Storage를 이용할 수 있는 Interface를 준비해뒀는데 이게 **CSI**이다. (스토리지 시스템 <-> Kubernetes간의 연결 담당)

**Key Vault에 대한 Secrets Store CSI 동작 방식**

우선 Pod가 실행된다면 CSI 드라이버는  사전에 만들어둔 SecretProviderClass를 참조한다. 이를 통해 직접 Azure Key Vault에 접근하여 비밀을 가져온다. 
이를 Pod 내부에 지정된 Volume 경로에 마운트 시켜 저장한다.

이와 같은 방식을 통해 Pod가 별도의 Key Vault API를 호출하지 않고도 파일처럼 비밀을 손쉽게 읽을 수 있게 되어진다!
