---
title: About
icon: fas fa-info-circle
order: 4
---

# 👋 안녕하세요, 김규형입니다

<div class="d-flex align-items-center mb-4 p-3 border rounded">
  <img src="/assets/img/favicons/unggu.jpg" class="rounded-circle me-3" width="100" alt="Profile">
  <div>
    <h3 class="mb-1">김규형 (Unggu)</h3>
    <p class="text-muted mb-1">Backend Developer & DevOps Engineer</p>
    <p class="mb-0">
      <a href="mailto:kyuhyeong.kim@kt.com"><i class="fas fa-envelope"></i> kyuhyeong.kim@kt.com</a>
    </p>
  </div>
</div>

> 클라우드 네이티브 애플리케이션 개발과 DevOps 문화에 관심이 많은 개발자입니다.
> 특히 Azure 기반의 Kubernetes 환경에서 안정적이고 확장 가능한 시스템을 설계하고 구축하는 것을 좋아합니다.
{: .prompt-info }

---

## 💼 경력

### KT Corporation
**Cloud Engineer** | 2024.01 ~ 현재

- Azure 기반 클라우드 인프라 설계 및 구축
- Kubernetes(AKS) 환경에서의 마이크로서비스 운영
- GitOps 기반 CI/CD 파이프라인 구축 및 운영
- 레거시 시스템의 클라우드 마이그레이션 프로젝트 수행

> **주요 성과:**
> - KT 외국인샵 Azure Migration 성공 (무중단 전환)
> - 인프라 운영 비용 35% 절감
> - 배포 자동화로 배포 시간 83% 단축 (30분 → 5분)
{: .prompt-tip }

---

## 🎓 교육

| 기간 | 학교/기관 | 전공/과정 |
|------|----------|-----------|
| 2024.01 ~ 현재 | KT | Cloud & DevOps 실무 |
| 2020.03 ~ 2024.02 | 대학교 | 컴퓨터공학 전공 |

### 자격증

<div class="row g-3 mb-4">
  <div class="col-md-6">
    <div class="card h-100">
      <div class="card-body">
        <h5 class="card-title">
          <i class="fas fa-certificate text-primary"></i> AZ-204
        </h5>
        <p class="card-text">Microsoft Certified: Azure Developer Associate</p>
        <p class="text-muted mb-0">
          <small>취득일: 2025.02 | 점수: 795점</small>
        </p>
      </div>
    </div>
  </div>
  <div class="col-md-6">
    <div class="card h-100">
      <div class="card-body">
        <h5 class="card-title">
          <i class="fas fa-certificate text-success"></i> AZ-900
        </h5>
        <p class="card-text">Microsoft Certified: Azure Fundamentals</p>
        <p class="text-muted mb-0">
          <small>취득일: 2024.11 | Pass</small>
        </p>
      </div>
    </div>
  </div>
</div>

---

## 🛠 기술 스택

### Backend Development
<div class="mb-3">
  <span class="badge bg-primary me-1">Java 17</span>
  <span class="badge bg-primary me-1">Spring Boot 3.x</span>
  <span class="badge bg-primary me-1">Spring Data JPA</span>
  <span class="badge bg-primary me-1">Spring Security</span>
  <span class="badge bg-primary">RESTful API</span>
</div>

### DevOps & Cloud
<div class="mb-3">
  <span class="badge bg-info me-1">Azure (AKS, ACR, Key Vault)</span>
  <span class="badge bg-success me-1">Kubernetes</span>
  <span class="badge bg-success me-1">Docker</span>
  <span class="badge bg-success me-1">Helm</span>
  <span class="badge bg-warning text-dark me-1">Argo CD</span>
  <span class="badge bg-warning text-dark">GitHub Actions</span>
</div>

### Database & Cache
<div class="mb-3">
  <span class="badge bg-secondary me-1">MySQL</span>
  <span class="badge bg-secondary me-1">PostgreSQL</span>
  <span class="badge bg-danger">Redis</span>
</div>

### Monitoring & Tools
<div class="mb-4">
  <span class="badge bg-dark me-1">Prometheus</span>
  <span class="badge bg-dark me-1">Grafana</span>
  <span class="badge bg-dark me-1">Azure Monitor</span>
  <span class="badge bg-dark">SonarQube</span>
</div>

---

## 🚀 프로젝트

### 1. JourneyJinni - 여행 플래너 서비스

<div class="card mb-3">
  <div class="card-body">
    <div class="row">
      <div class="col-md-8">
        <h5 class="card-title">GitOps 기반 클라우드 네이티브 여행 플래너</h5>
        <p class="text-muted">2024.03 ~ 2024.06 (4개월) | 팀 프로젝트 (4인)</p>
        
        <p><strong>역할:</strong> Backend 개발 & DevOps 인프라 구축</p>
        
        <p><strong>주요 기능:</strong></p>
        <ul class="mb-2">
          <li>AI 기반 여행 일정 추천 시스템</li>
          <li>실시간 협업 플래너 (WebSocket)</li>
          <li>소셜 로그인 (OAuth 2.0)</li>
          <li>지도 기반 여행지 탐색</li>
        </ul>
      </div>
      <div class="col-md-4">
        <h6>기술 스택</h6>
        <span class="badge bg-primary me-1 mb-1">Spring Boot</span>
        <span class="badge bg-primary me-1 mb-1">JPA</span>
        <span class="badge bg-success me-1 mb-1">AKS</span>
        <span class="badge bg-success me-1 mb-1">Argo CD</span>
        <span class="badge bg-info me-1 mb-1">Azure MySQL</span>
        <span class="badge bg-danger mb-1">Redis</span>
      </div>
    </div>
  </div>
</div>

<details>
  <summary><strong>🔍 프로젝트 상세 설명 & 느낀점</strong></summary>

  <div class="mt-3">

#### 💡 이 프로젝트는...

개인적으로 여행을 좋아하지만, 매번 일정을 계획하는 것이 번거로웠던 경험에서 시작했습니다. 
특히 여러 명이 함께 여행을 갈 때 의견을 조율하고 일정을 공유하는 과정이 복잡했는데, 
이를 해결할 수 있는 협업형 여행 플래너를 만들고자 했습니다.

#### 🎯 기술적 도전과제

**1. GitOps 기반 자동 배포 시스템 구축**

처음으로 Argo CD를 활용한 GitOps 방식의 배포를 구현했습니다.

```yaml
# Argo CD Application 설정
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: journeyjinni-backend
spec:
  source:
    repoURL: https://github.com/org/k8s-manifests
    path: overlays/production
  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

- **효과:** 코드 푸시 → 자동 빌드 → 자동 배포까지 완전 자동화
- **결과:** 배포 시간 30분 → 5분으로 단축

**2. Azure Key Vault 통합으로 보안 강화**

데이터베이스 비밀번호, API 키 등 민감 정보를 코드에서 완전히 분리했습니다.

```java
@Configuration
public class KeyVaultConfig {
    @Bean
    public SecretClient secretClient() {
        return new SecretClientBuilder()
            .vaultUrl(vaultUrl)
            .credential(new DefaultAzureCredentialBuilder().build())
            .buildClient();
    }
}
```

- **배운 점:** 환경별 설정 관리의 중요성과 보안 Best Practice
- **개선 효과:** 시크릿 로테이션 자동화, 감사 로그 기록

**3. Redis 캐싱으로 성능 최적화**

자주 조회되는 여행지 정보를 캐싱하여 DB 부하를 줄였습니다.

```java
@Cacheable(value = "destinations", key = "#id")
public Destination getDestination(Long id) {
    return repository.findById(id)
        .orElseThrow(() -> new NotFoundException());
}
```

- **성능 개선:** 평균 응답 시간 800ms → 200ms
- **확장성:** 동시 접속자 처리 능력 4배 향상

#### 😊 느낀점

> **가장 뿌듯했던 순간:**
> GitHub에 코드를 푸시하면 자동으로 테스트되고, 빌드되고, Kubernetes 클러스터에 배포되는 
> 전체 파이프라인이 처음 성공했을 때의 성취감은 잊을 수 없습니다.
{: .prompt-tip }

이 프로젝트를 통해 **"코드형 인프라(Infrastructure as Code)"**의 진정한 의미를 깨달았습니다. 
단순히 서버를 설정하는 것이 아니라, Git으로 버전 관리되고, 자동으로 배포되며, 
문제가 생기면 이전 버전으로 즉시 롤백할 수 있는 시스템을 구축하는 과정이 
개발자로서 큰 성장의 계기가 되었습니다.

#### 🔗 관련 링크

- [GitHub Repository](https://github.com/your-org/journeyjinni)
- [기술 블로그 - GitOps 도입기](/posts/journeyjinni-gitops/)

  </div>
</details>

---

### 2. KT 외국인샵 Azure Migration

<div class="card mb-3">
  <div class="card-body">
    <div class="row">
      <div class="col-md-8">
        <h5 class="card-title">레거시 시스템 Azure 클라우드 전환</h5>
        <p class="text-muted">2024.01 ~ 2024.03 (3개월) | KT Corporation</p>
        
        <p><strong>역할:</strong> Cloud Migration Engineer</p>
        
        <p><strong>프로젝트 목표:</strong></p>
        <ul class="mb-2">
          <li>KT Cloud → Azure AKS 환경 전환</li>
          <li>무중단 마이그레이션 (가용성 99.9% 유지)</li>
          <li>인프라 운영 비용 30% 절감</li>
          <li>배포 프로세스 자동화</li>
        </ul>
      </div>
      <div class="col-md-4">
        <div class="bg-light p-3 rounded">
          <h6>달성 성과</h6>
          <ul class="mb-0 small">
            <li>✅ 무중단 마이그레이션 성공</li>
            <li>✅ 응답속도 40% 개선</li>
            <li>✅ 장애 복구 시간 90% 단축</li>
            <li>✅ 월 운영비 35% 절감</li>
          </ul>
        </div>
      </div>
    </div>
  </div>
</div>

<details>
  <summary><strong>🔍 프로젝트 상세 설명 & 느낀점</strong></summary>

  <div class="mt-3">

#### 💡 이 프로젝트는...

입사 후 첫 번째 프로젝트였습니다. 솔직히 Docker와 Kubernetes에 대한 지식이 거의 없는 상태에서
"Azure Migration 프로젝트를 해보라"는 지시를 받았을 때는 정말 막막했습니다.

하지만 이 프로젝트를 통해 클라우드 인프라의 A부터 Z까지 경험할 수 있었고,
실제 운영 중인 서비스를 다루면서 책에서는 배울 수 없는 실무 감각을 익힐 수 있었습니다.

#### 🎯 기술적 도전과제

**1. 무중단 마이그레이션 전략**

> **가장 큰 고민:** 어떻게 서비스 중단 없이 환경을 전환할 것인가?
{: .prompt-warning }

**Blue-Green Deployment 전략 적용:**

```
┌─────────────────────────────────────────┐
│  기존 환경 (Blue) - KT Cloud           │
│  ↓ 트래픽 100%                          │
└─────────────────────────────────────────┘
              ↓ 점진적 전환
┌─────────────────────────────────────────┐
│  신규 환경 (Green) - Azure AKS          │
│  ↓ 트래픽 10% → 50% → 100%              │
└─────────────────────────────────────────┘
```

- DNS 가중치 라우팅으로 트래픽 점진적 이동
- 실시간 모니터링으로 오류율 체크
- 문제 발생 시 즉시 롤백 가능하도록 준비

**2. 데이터베이스 동기화**

가장 민감한 부분은 고객 데이터였습니다.

```bash
# Azure Database Migration Service 활용
az dms create --resource-group rg-migration \
  --name kt-shop-migration \
  --source-connection ktcloud-mysql \
  --target-connection azure-mysql
```

- **전략:** 실시간 복제로 데이터 일관성 보장
- **검증:** 자동화된 데이터 무결성 체크 스크립트 작성
- **결과:** 데이터 손실 0건

**3. Kubernetes 환경 구축**

기존에는 VM 기반이었는데, 컨테이너 기반으로 전환하면서 겪은 시행착오들:

```yaml
# 초기에는 이렇게 작성했다가...
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ktshop-backend
spec:
  replicas: 1  # ❌ 단일 Pod
  
# 이렇게 개선
spec:
  replicas: 3  # ✅ HA 구성
  strategy:
    rollingUpdate:
      maxUnavailable: 1
      maxSurge: 1
```

- **배운 점:** Pod 개수, 리소스 제한, Health Check의 중요성
- **실수:** 초기에 메모리 제한을 너무 낮게 설정해서 OOMKilled 발생

#### 📊 성과 측정

| 항목 | AS-IS (KT Cloud) | TO-BE (Azure) | 개선율 |
|------|------------------|---------------|--------|
| 평균 응답 시간 | 1,200ms | 720ms | **40% ↓** |
| 장애 복구 시간 | 60분 | 5분 | **92% ↓** |
| 월 인프라 비용 | 100만원 | 65만원 | **35% ↓** |
| 배포 소요 시간 | 3시간 | 10분 | **94% ↓** |

#### 🤔 가장 힘들었던 점

> **네트워크 문제 트러블슈팅**
> 
> 배포 후 특정 API만 간헐적으로 타임아웃이 발생하는 문제가 있었습니다.
> 3일 동안 로그를 분석한 결과, Azure Application Gateway의 타임아웃 설정이 
> 백엔드 처리 시간보다 짧게 설정되어 있었던 것이 원인이었습니다.
{: .prompt-danger }

이 경험을 통해 **네트워크 계층에 대한 이해의 중요성**을 깨달았고,
이후 AZ-204 시험 준비를 하게 된 계기가 되었습니다.

#### 😊 느낀점

입사 직후 던져진 큰 프로젝트였지만, 오히려 **실전에서 배우는 것이 가장 빠른 성장**이라는 것을 느꼈습니다.

특히 인상 깊었던 점은:
- **실패의 가치:** 수많은 에러 메시지와 실패를 겪으면서 오히려 더 깊이 이해하게 됨
- **문서화의 중요성:** 트러블슈팅 과정을 기록해두니 팀원들에게도 도움이 됨
- **협업의 중요성:** 네트워크 팀, 보안 팀과의 협업 없이는 불가능했던 프로젝트

이 프로젝트 이후 **"클라우드 엔지니어"**로서의 자신감이 생겼고,
더 깊이 있는 학습을 위해 AZ-204 자격증에 도전하게 되었습니다.

#### 🔗 관련 링크

- [운영 사이트](https://globalshop.kt.com/global/globalMain.do)
- [기술 블로그 - Azure Migration 회고](/posts/kt-migration-retrospective/)

  </div>
</details>

---

## 📝 기술 블로그

개발하면서 배운 내용과 트러블슈팅 경험을 기록하고 있습니다.

### 최근 포스트
{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url | relative_url }}) - `{{ post.date | date: "%Y.%m.%d" }}`
{% endfor %}

[전체 포스트 보기 →](/archives/)

---

## 📞 Contact

<div class="d-flex gap-2 flex-wrap mb-4">
  <a href="mailto:kyuhyeong.kim@kt.com" class="btn btn-outline-primary">
    <i class="fas fa-envelope"></i> Email
  </a>
  <a href="https://github.com/unggu0704" class="btn btn-outline-dark">
    <i class="fab fa-github"></i> GitHub
  </a>
  <a href="https://www.linkedin.com/in/규형-김-5b5b10299" class="btn btn-outline-info">
    <i class="fab fa-linkedin"></i> LinkedIn
  </a>
  <a href="https://unggu.xyz" class="btn btn-outline-success">
    <i class="fas fa-blog"></i> Tech Blog
  </a>
</div>

---

> 함께 성장할 수 있는 기회가 있다면 언제든 연락 주세요! 😊
{: .prompt-tip }
