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

---

Issue: secret이 정상적으로 생성되었지만 Secret이 갱신되지 않은 현상

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






