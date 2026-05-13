이전 글에서는 Key Vault와 AKS간의 UMI 설정과 Pod안의 Volume으로 Mount까지 진행하였습니다. 

이번에는 실제 소스코드 내 민감정보를 KV에 저장하고 이걸 WAS 내 환경 변수(ENV)로 사용하기 위한 방법을 알아겠습니다.

`serviceName/dir1/dir2/amuguna.yaml`에 하드코딩 되어 있는 민감 정보가 검출되어 이를 Key Vault로 이관해야한다고 합니다.
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

기존 connection-string을 추가하였다.

이 값은 WAS내에서 사용되는 중요한 값이기에, 환경변수(ENV)로 저장이 되어야합니다.  

이전에 Pod내 /mnt/secrets에 값은 저장하였지만 이 값을 환경변수로 사용할려면 추가적인 작업이 필요합니다.

### SecretProviderClass에 가져올 Secret 명시

SecretProviderClass는 지정한 CSI 드라이버를 사용해 Key Vault로 비밀을 가져올지 지정하면서 어떤 비밀을 가져올지도 지정을 해야합니다.  

아래 yaml은 Key Vault의 두가지 비밀 `decrypt-key`, `applicationsinsight-connection-string`를 가져온다고 선언하였습니다.

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



