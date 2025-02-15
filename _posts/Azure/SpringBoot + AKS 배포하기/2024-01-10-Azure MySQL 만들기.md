---
title: "07. [Azure/Spring] Azure MySQL 만들기"
author: "unggu"
date: 2025-01-11 19:27:13 +0800
categories: [Azure, AKS로 Spring Web 배포하기]
tags: [Azure, AKS, Kubernets, Container, DevOps, Docker]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/Azurespring.png
---

BackEnd Server를 컨테이너화 하여 AKS안에 가동 시키는것까지는 성공했다.. 

당연하게도 DB 연결이 안되어 아래와 같이 JDBC 오류가 발생

![스크린샷 2024-12-31 오후 7.32.29.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-12-31_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_7.32.29.png)

DB를 연결하기 위해서는 두가지 방법이 필요하다.

1. **기존의 MySQL 이미지를 컨테이너화하여 배포**
2. **Azure MySQL 서버를 이용**

고민을 했지만.. 이미 기존의 image를 말아서 AKS에 배포하는건 해본 경험이라고 생각해 이번에는 Azure에서 PaaS 형태로 지원하는 Azure SQL Server를 사용해서 AKS ↔ Azure Server와 연동을 해보기로 하였다…

### Issue

MySQL 서버를 생성하는데 있어 아래와 같은 에러 발생

```jsx
{"code":"DeploymentFailed",
"target":"/subscriptions/5707ca3b-ac07-414d-bdc1-9d769db260e5/resourceGroups/jiiniRG/providers/Microsoft.Resources/deployments/MySqlFlexibleServer_5f870a70c76311efbb94dbed7cd2e58d",
"message":"At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/arm-deployment-operations for usage details.",
"details":[{"code":"InvalidParameterValue","message":"Invalid value given for parameter administratorLogin. Specify a valid parameter value."}]}
```

매개변수가 잘못 된 에러로 추정…

| **관리자 사용자 이름** | **나의 데모저** | 로그인 계정은 서버에 연결할 때 사용됩니다. 관리자 사용자 이름은 **azure_superuser** , **admin** , **administrator** , **root** , **guest** , **sa** 또는 **public** 일 수 없습니다 . 허용되는 최대 문자 수는 32자입니다. |
| --- | --- | --- |
| **비밀번호** | 귀하의 비밀번호 | 서버 관리자 계정의 새 비밀번호입니다. 8~128자여야 합니다. 또한 다음 범주 중 세 가지의 문자를 포함해야 합니다. 영어 대문자, 영어 소문자, 숫자(0~9) 및 영숫자가 아닌 문자( , `!`, `$`, `#`, `%`등). |

> [*https://learn.microsoft.com/en-gb/azure/mysql/flexible-server/quickstart-create-server-portal*](https://learn.microsoft.com/en-gb/azure/mysql/flexible-server/quickstart-create-server-portal)
> 

마이크로소프트 공식문서 참조 결과.. 아이디와 비번에 대한 검증 rule이 있다. → **해결**

근데 이러면 입력단에서 검증을 해서 다음으로 못넘어가게 하는게 맞지 않나…?

---

### Issue - Azure MySQL 연결

일반적인 host와 password만 입력했지만 연결 에러 오류?

**SSL 인증서 오류**

DigiCertGlobalRootCA.crt가 없다고 한다.

서버 파라미터에서 SSL을 미설정..

![스크린샷 2024-12-31 오후 9.19.02.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2024-12-31_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_9.19.02.png)

하지만 여전히 연결 오류 발생

**`nsloolkup`을 통해 DNS 확인**

```jsx
(base) gimgyuhyeong@UNGGU-2 ~ % nslookup jinnidb.mysql.database.azure.com

Server:		fe80::6c3a:ffff:fe13:8f64%12
Address:	fe80::6c3a:ffff:fe13:8f64%12#53

Non-authoritative answer:
Name:	jinnidb.mysql.database.azure.com
Address: 20.249.183.48
```

**MySQL 버전 확인**

```jsx
 mysql --version
mysql  Ver 9.0.1 for macos14.7 on arm64 (Homebrew)
```

**Telnet 확인**

```jsx
(base) gimgyuhyeong@UNGGU-2 ~ % telnet jinnidb.mysql.database.azure.com 3306

Trying 64:ff9b::14f9:b730...

telnet: connect to address 64:ff9b::14f9:b730: Operation timed out
Trying 20.249.183.48...
telnet: connect to address 20.249.183.48: Operation timed out
telnet: Unable to connect to remote hostq
```

연결 자체에 문제가 있다고 생각하여 구글링 결과 Azure DB 자체적인 방화벽이 있다는것을 알았다..

![스크린샷 2025-01-04 오후 6.23.14.png]({{ site.baseurl }}{{ page.url }}/img/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2025-01-04_%25E1%2584%258B%25E1%2585%25A9%25E1%2584%2592%25E1%2585%25AE_6.23.14.png)

방화벽 란에 자신의 IP를 추가하여 접속을 허용한다.

**연결이 성립�