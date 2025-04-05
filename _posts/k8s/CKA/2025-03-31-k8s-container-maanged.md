---
author: "unggu"
title: "[CKA] 컨테이너 설정하기"
date: 2025-03-31 11:11:12 +0800
categories: [k8s, CKA]
tags: [CKA, k8s, Container, Docker, Pod, Secret]
render_with_liquid: true
comments: true
image:
  path: assets/img/metaimg/cka/cka.png
---

## Command & Arguments

### 컨테이너의 Life Cycle

Container는 별도의 OS가 존재하지 않기 때문에 실행 중인 프로세스가 없다면 즉시 종료되는 성질을 가지고 있다. (PID 1의 Life Cycle과 동일)

모든 프로레스가 종료된다면 컨테이너는 `Completed` 상태(또는 `CrashLoopBackOff`)로 변경되어 마치 문제가 있는 상태처럼 보이기도 한다.

이를 방지하기 위해 컨테이너는 지속적으로 실행되는 프로세스가 있어야하는데 일반적으로 사용되는 `Sleep`이 대표적이다.  

**busybox 이미지를 사용하는 컨테이너를 60분동안 기동**

```yaml
kubectl run mypod --image=busybox -- /bin/sh -c "sleep 3600"
```

*사실 busybox는 pod가 죽지 않게 하기위한 가벼운 유틸리티 도구의 모임으로 `sleep` 명령어 없이도 죽지 않는다…*

### Commands와 Arguments 개념

Kubernetes에서 컨테이너의 실행 방식을 결정할 때는 주로 명령어와 인자를 사용하는데 아래와 같이 작성한다고 가정하자.

```yaml
command: ["/bin/sh"]
args: ["-c", "echo Hello, Kubernetes!"]
```

이러한 명령은 컨테이너가 실행 되어질 때

```yaml
/bin/sh -c "echo Hello, Kubernetes!"
```

이렇게 변경될 수 있다.

실제 `YAML`을 작성한다면 아래와 같다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
  - name: busybox
    image: busybox
    command: ["/bin/sh"]
    args: ["-c", "echo Hello CKA && sleep 3600"]
```

**EntryPoint**

일반적으로 컨테이너들은 기본적인 EntryPoint가 있는데 nginx 같은 경우 기본적으로 `nginx -g 'daemon off;` 명령어가 사용된다. 

이걸 **`command:`** 를 사용해 변경할 수 있는데 아래와 같이 `"/bin/sh"` 를 사용한다면 nginx가 실행되지 않고 커스텀 명령어를 사용할 수 있게 된다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-custom
spec:
  containers:
  - name: nginx
    image: nginx
    command: ["/bin/sh"]
    args: ["-c", "echo 'Nginx is replaced!' && sleep 1000"]
```

**Ubuntu를 사용하는 컨테이너**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    args: ["sleep", "3600"]
```

Ubuntu는 기본적으로 `/bin/sh -c`가 실행되기에 해당 args를 입력시 약 1시간 동안 pod가 가동되어진다. 

하지만 Command를 입력하여 직접 기본 실행 프로세스를 지정할 수 있다.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-pod
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    command: ["sleep"]
    args: ["3600"]
```

sleep이 기본 실행 파일로 지정되면서 별도의 bash 없이 1시간 동안 유지가 된다.

> ***Pod는 Immutable(불변)임으로 이미 시작된 pod의 command나 args는 변경 될 수 없다.***
> 

## ConfigMaps

k8s에서 환경설정 데이터(Key-Value)를 관리하는 객체로 컨테이너 안에 하드코딩하지 않고, 주요 환경 변수를 분리해서 설정할 수 있다. 

단순히 환경 변수뿐만 아니라, 파일 Mount 및 명령줄 인수로 저장할 수 있는데 민감한 정보는 저장하는데는 적절하지 않다. (`Secret` 사용!)

### ConfigMap 생성 및 Pod 이용

**1. `yaml`로 ConfigMap 생성**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: my-config
data:
  APP_MODE: "production"
  DB_HOST: "mysql-service"
  DB_PORT: "3306"
```

**2. CLI로 ConfigMap 생성**

```yaml
kubectl create configmap my-config --from-literal=APP_MODE=production --from-literal=DB_HOST=mysql-service
```

`--from-literal=`만 반복하면 된다.

**1. `env`에 원하는 환경변수 불러와서 사용**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  containers:
  - name: my-container
    image: my-app
    env:
    - name: APP_MODE
      valueFrom:
        configMapKeyRef:
          name: my-config  # ConfigMap 이름
          key: APP_MODE    # ConfigMap 내부 Key
```

**2. `envFrom`으로 모든 환경 변수 불러와 사용**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: env-configmap
spec:
  containers:
    - name: app
      command: ["/bin/sh", "-c", "printenv"]
      image: busybox:latest
      envFrom:
        - configMapRef:
            name: myconfigmap
```

해당 방식으로 선언할 경우 `kubectl describe` 방식으로 보이는 `env`에서는 환경 변수가 보이지 않고 `kubectl exec -it`를 통해 pod에 들어가서 환경변수를 확인해야한다.

**3. Volume으로 마운트**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  volumes:
  - name: config-volume
    configMap:
      name: my-config
  containers:
  - name: my-container
    image: my-app
    volumeMounts:
    - name: config-volume
      mountPath: /etc/config
```

### 왜 ConfigMap을 쓰는 이유?

주요 환경 설정을 코드에서 분리함으로써 Pod 재배포 없이 설정값을 변경 가능하다. 하지만 pod가 최초로 생성할 때 ConfigMap이 존재하지 않았다면 환경 변수가 설정되지 않을 수 있다. 

## Secret

ConfigMap으로 저장하기에는 조금 민감한 정보(비밀번호, API 키, TLS 인증서)를 저장하는 저장소로 ConfigMap과 다른점은 **Base64 인코딩된 데이터**를 저장한다. 

### Secret의 생성

 **1. `yaml`파일 사용**

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: my-secret
type: Opaque
data:
  username: dXNlcm5hbWU=  # "username"을 base64로 인코딩한 값
  password: cGFzc3dvcmQ=  # "password"를 base64로 인코딩한 값
```

**2. CLI 사용**

```yaml
kubectl create secret generic my-secret \
  --from-literal=username=user1 \
  --from-literal=password=pass123
```

ConfigMap과 마찬가지로 `--from-literal=...`을 사용한다.

### Secrets의 사용

**1. `envFrom`또는 `env`를 사용해 환경 변수로 사용**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx
    env:
    - name: USERNAME
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: username
    - name: PASSWORD
      valueFrom:
        secretKeyRef:
          name: my-secret
          key: password
```

**2. Volume으로 마운트 하여 사용**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-app
spec:
  containers:
  - name: app
    image: nginx
    volumeMounts:
    - name: secret-volume
      mountPath: "/etc/secret"
      readOnly: true
  volumes:
  - name: secret-volume
    secret:
      secretName: my-secret
```

### Secrets의 적절한 사용법

**Secret**은 그저 값들을 Base64 인코딩을 해준다. → **암호화가 아니다.** 즉 필요하다면 추가적인 암호화를 할 필요가 있으며, 이러한 `Secret.yaml`을 Github 같은 곳에 upload해서는 안된다. 

**Secret**에 접근하기 위해서는 **CSP 또는 RBAC을 통해 접근을 제한**해야한다. 

또한 `kubectl get`으로 확인하지 못하도록 `encryption at rest` 항목을 설정하는 것이 보안성을 높이며 환경변수로 등록하는 것보다는 Volume을 사용해 노출을 최소화하여야 한다.