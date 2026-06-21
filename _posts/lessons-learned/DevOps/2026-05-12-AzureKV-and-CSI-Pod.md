---
title: "[Azure] Key Vault의 비밀을 환경 변수로 사용하기"
author: "unggu"
date: 2026-05-12 19:12:42 +0800
categories: [lessons-learned, DevOps]
tags: [Azure, AKS, Kubernets, Container, DevOps, Key Vault]
render_with_liquid: true
comments: true
image:
  path: assets\img\metaimg\azure204\keyvault.png
---

이전 글에서는 Key Vault와 AKS간의 UMI 설정과 Pod안의 Volume으로 Mount까지 진행하였습니다. 

> [AKS와 Azure Key Vault](https://unggu.dev/azure/azure%20공부/AKS와KeyVault/)

이번에는 실제 소스코드 내 민감정보를 비밀 저장소에 저장하고 이걸 서비스내 환경 변수(ENV)로 사용하기 위한 방법을 알아보겠습니다.

Azure Monitor의 연결 변수(`connectionString`)이 `ConfigMap`내 하드코딩 되어 있어 이를 Key Vault로 이관해야하는 상황을 가정합니다.

```yaml
data:
  SPRING_PROFILES_ACTIVE: prd
  applicationinsights.json: |-
    {
      "connectionString": "{대강 엄청 중요한 정보, KV 이관 필요}",
      "role": {
        "name": "serviceName",
        "instance": "serviceName-was"
      },
      "sampling": {
        "percentage": 100
      },
```

소스 코드 내 하드코딩 되어져 있는 이 정보를 삭제하고 Azure Key Vault내 비밀을 하나 만들어 값을 저장해줍니다.

![image.png]({{ site.baseurl }}{{ page.url }}/img/portal.png)_connection-string을 추가_

이 값은 WAS내에서 사용되는 중요한 값이기에, 환경변수(ENV)로 저장이 되어야합니다.  

이전에 Pod내 지정된 Volume내 `/mnt/secrets`에 값은 저장하였지만, 이 값을 환경변수로 사용할려면 추가적인 작업이 필요합니다.

### SecretProviderClass에 가져올 Secret 명시

SecretProviderClass는 지정한 CSI 드라이버를 사용해 Key Vault로 비밀을 가져올지 지정하면서 어떤 비밀을 가져올지도 지정 가능합니다.

아래 `yaml`은 Key Vault의 두가지 비밀 `decrypt-key`, `applicationsinsight-connection-string`를 가져온다고 선언하였습니다.

```yaml
apiVersion: secrets-store.csi.x-k8s.io/v1
kind: SecretProviderClass
metadata:
  name: my-secret-provider
spec:
  provider: azure
  parameters:
    usePodIdentity: "false"  
    clientID: "<AKS_MANAGED_IDENTITY_CLIENT_ID>"   # key-vault 접근 가능한 UAMI의 client ID
    keyvaultName: <KEY_VAULT_NAME>   # key-vault명
    cloudName: ""                       
    objects: |
      array:
        - |
          objectName: decrypt-key # Key Vault에 지정된 이름
          objectType: secret
          objectVersion: ""
        - |
          objectName: applicationsinsight-connection-string # Key Vault에 지정된 이름
          objectType: secret
          objectVersion: ""
    tenantId: "<TENANT_ID>"   # tenantID
  secretObjects:
    - secretName: my-secret   
      type: Opaque
      data:
        - objectName: decrypt-key # Key Vault에 지정된 이름
          key: DECRYPT-KEY  # 환경변수로 사용할 이름
        - objectName: applicationsinsight-connection-string
          key: APPLICATIONINSIGHTS_CONNECTION_STRING
```
          
CSI Driver는 Pod가 재기동 되어질 때, 이 SecretProviderClass를 읽고 Pod의 지정된 Volume인 `/mnt/secret-store`에 저장하고 이를 기반한 Secret 리소스를 생성합니다


#### SPC가 생성한 Secret 리소스
```
[root:~]$ k describe secret my-secret
Name:         my-secret
Labels:       secrets-store.csi.k8s.io/managed=true
Annotations:  <none>

Type:  Opaque

Data
====
APPLICATIONINSIGHTS_CONNECTION_STRING:  250 bytes
```

### Pod에서 Secret 값 사용하기 

Secret의 값들을 환경변수로 사용하기 위해서는 두 가지 방법이 있습니다.

**1. `secretKeyRef`를 활용하여 하나씩 지정**
```yaml
env:
  - name: DECRYPT-KEY
    valueFrom:
      secretKeyRef:
        name: my-secret
        key: decrypt-key
```

Secret 자체에 여러 값이 있고 이걸 필요한 것만 선별해서 넣을 수 있습니다.

하나의 Secret을 여러 Pod가 공유할 때 선별해서 주입이 가능합니다.

**2.`envFrom.secretRef`로 통째로 추가**
```yaml
envFrom:
  - secretRef:
      name: my-secret
```

Secret과 Pod가 1:1 관계일 때 사용합니다. 

모든 Secret의 비밀이 주입되기에 SPC 자체를 주입기로 사용할 수 있어 관리가 편합니다.

서비스의 기준에 맞추어 원하는 방식대로 설정하여 사용하면 될듯합니다.

#### 타임라인 정리

최종적으로 Azure Key Vault에서 컨테이너네 환경변수까지 주입되는 타임라인은 아래와 같이 정리 가능합니다.

1. CSI Driver가 Azure Key Vault에서 값을 가져온다.
2. Pod가 기동되어질 때, Pod내 지정된 Volume에 정의된 값을 Mount한다.
3. volume의 값을 기반으로 Secret 리소스가 생성됨
4. kublet이 모든 준비를 확인한다. (Volume, Secret 등...)
5. 컨테이너를 기동하면서 Secret의 값을 환경변수로 설정한다.

---

#### Issue: secret이 정상적으로 생성되었지만 Secret이 갱신되지 않은 현상

AKS의 Secret Rotation이 Enabled 되어져 있는지 확인 
```
az aks show -g <리소스-그룹명> -n <클러스터명> --query "addonProfiles.azureKeyvaultSecretsProvider.config"
```

`config` 부분이 `null`인 경우 SPC가 Pod의 Volume에 저장은 하지만 Secret 자체의 Sync가 되지 않는 현상이 발생합니다.

이럴 경우 아래 명령어를 통해 AKS의 Secret Rotation을 활성화 해주면 해결됩니다.
```
az aks update \
--resource-group <리소스-그룹명> \
--name <클러스터명> \
--enable-secret-rotation \
--rotation-poll-interval 2m
```






