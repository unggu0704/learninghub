---
title: "[CI/CD] Github Runner 서버란"
author: 김규형
date: 2025-01-01 16:18:22 +0800
categories: [DevOps, CI/CD]
tags: [DevOps, CI/CD, Github]
render_with_liquid: true
comments: true
image:
    assets/img/metaimg/gitrunner.png
---


사내 CI/CD 구축 뿐만 아닌 사이드 프로젝트로 진행하는 Spring boot app CI 등뿐만 아니라 이 블로그까지 Github Action을 사용하고 있다. 

이런 Github Action을 위한 `yaml` 파일 작성에 있어 공통적으로 아래와 같은 Runner 서버를 사용하는 부분을 적는다.

```yaml
  build:
    runs-on: ubuntu-22.04
```

```yaml
  build-and-push:
    runs-on: ubuntu-latest
```

```yaml
  build:
    runs-on: self-hosted
```

여기서 말하는 `runs-on` 이라는건 뭔지 특정한 OS(ubuntu) `github-hosted`를 지정하는 것과 `self-hosted` 와는 어떤 차이가 있는지 알아보자

### Github-hosted runner

Github-hosted Runner는 github에서 제공하는 가상머신으로 사용자가 지정한 workflow를 실행하는데 있어 깨끗한 인스턴스를 제공한다. 

관리와 유지 보수는 모두 github에서 제공하고 항상 최신 상태를 유지한다. 또한 격리된 가상머신에서 실행되기에 보안성을 높이며 일관성을 보장한다. 

일반적으로 아래와 같이 정의하고 사용이 가능하다.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest #최신 ubuntu 환경
    steps:
      - name: Checkout code
        uses: actions/checkout@v3

```

이외에도 *window*와 *macOS*같은 환경 또한 지원한다.

만약 최신버전을 사용하는데 있어 특정한 issue가 발생할 수 도 있다. (*해당 블로그를 만드는데 있어 `ubuntu`의 버전이 올라감에 따라 action의 정상 실행이 되지 않닸다*.) 

그렇기에 legacy 버전 또한 `ubuntu-20.04`, `macos-11` 지정 및 사용이 가능하다.

또한 레포지터리를 public으로 설정시 무제한으로 실행할 수 있는… 혜택이 제공된다. 

### self-hosted runner

사용자가 관리하는 물리적 또는 가상머신으로 workflow를 실행하는데 있어 깃허브의 정책 및 보안성을 사용하지 않고 자체적인 runner 서버를 사용한다.

이는 맞춤형 환경을 만드는데 큰 이점을 가지는데 요구사항에 맞는 소프트웨어나 라이브러리 등을 자체적으로 구성할 수 있다. 또한 상용 서비스에 있어 높은 하드웨어 성능이나 GPU가 필요한 경우 적합하다. (여러 Runner를 구성하여 병렬 작업 실행)

일반적으로 공용 네트워크로 연결된 *Github-hosted Runner*와 다르게 폐쇄적인 환경에서 구축될 수 있다. 이는 보안성에 큰 이점을 주며, 민감한 정보를 처리해야하는 사내망에서 사용되기 적합하다…

회사에 돈이 많다면? Runner를 많이 설치해 작업을 실행 할 수 있다… 

퇴근 시간에 가까워질수록 Runner 서버의 큐에 작업이 쌓이는걸 볼 수도 있다.

일반적으로 아래와 같이 정의하고 사용이 가능하다.

```yaml
jobs:
  build:
    runs-on: self-hosted  # Self-hosted Runner 사용
    steps:
      - name: Checkout code
        uses: actions/checkout@v3
      - name: Build project
        run: make build

```

사내에 있는 특정 Runner를 지정하고 싶다면 태그(tag)를 사용할 수도 있다.

```yaml
jobs:
  test:
    runs-on: [self-hosted, linux, x64]
    steps:
      - name: Run tests
        run: npm test
```