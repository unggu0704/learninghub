---
title: Introduce | 김규형
icon: fas fa-info-circle
order: 4
hidden: true
---

# 👋 안녕하세요 김규형입니다

<div class="d-flex align-items-center mb-4 p-4 bg-light rounded shadow-sm">
  <img src="/assets/img/favicons/me.jpg" class="rounded-circle me-3 shadow" width="100" alt="Profile">
  <div>
    <h2 class="mb-1">김규형 (Unggu)</h2>
    <p class="text-muted mb-2">Server/Backend Developer &amp; DevOps Engineer</p>
    <p class="mb-0">
      <i class="fas fa-envelope text-primary"></i>
      <a href="mailto:unggu556@naver.com" class="text-decoration-none">unggu556@naver.com</a>
    </p>
  </div>
</div>

> Java/Spring 기반 대고객 서비스를 개발하고, 그 서비스를 클라우드에서 직접 운영하고 있는 개발자입니다.
> 기능을 만드는 일만큼 그 기능이 장애 없이 돌아가게 만드는 일에 흥미를 느껴, 트랜잭션 구조 개선·Redis 단일 장애점 제거·JVM 메모리 원인 분석 같은 프로덕션 이슈를 직접 파고들어 왔습니다.
> "겉으로 드러난 증상과 이면의 구조적 원인은 다를 수 있다"는 것이 지금도 제가 문제를 대하는 기준입니다.
{: .prompt-info }

---

## 💼 경력

<div class="position-relative ps-4 mb-5">
  <div class="position-absolute top-0 start-0 bottom-0" style="width: 2px; background: linear-gradient(to bottom, #0d6efd, #198754);"></div>

  <div class="position-relative">
    <div class="card border-primary shadow-sm">
      <div class="card-header bg-primary text-white">
        <h5 class="mb-0">주식회사 케이티디에스 (KT DS)</h5>
        <small>2024.07 ~ 재직중 | ICT사업본부 CRM담당 / 전임</small>
      </div>
      <div class="card-body">
        <p class="text-muted mb-4">KT 대고객 e-커머스 시스템의 개발·운영과 클라우드 환경 구축을 함께 담당하고 있습니다.</p>

        <h6 class="fw-bold text-primary">
          <i class="fas fa-code me-2"></i>대고객 e-커머스 시스템 개발 및 ITO 운영
        </h6>
        <ul class="mb-3">
          <li>고객 SR(Service Request) 및 VOC 대응을 위한 Java/Spring 기반 기능 개발</li>
          <li>BO 모니터링 자동화 기능 설계·구현 (스케줄러 기반)</li>
          <li>외부 연동 로직을 트랜잭션에서 분리하는 구조 개선 설계 (대용량 트래픽 처리 시 병목 지점 식별 및 해소)</li>
          <li>단일 장애점이던 Redis 캐시 구조를 Failover·Circuit Breaker 패턴으로 재설계
            <a href="https://unggu.dev/lessons-learned/tech/redis/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
          <li>JVM GC 로그 분석으로 Old Gen 누적 원인을 규명해 OOM 재발 방지 (IHOP·메모리 limit 재산정)
            <a href="https://unggu.dev/lessons-learned/java/jvm/Memory-압박-추적/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
            <a href="https://unggu.dev/lessons-learned/java/jvm/Memory-압박-해결/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
        </ul>

        <h6 class="fw-bold text-success">
          <i class="fas fa-cloud me-2"></i>대고객 서비스 클라우드 환경 구축 및 운영
        </h6>
        <ul class="mb-3">
          <li>Kubernetes(AKS) 기반 대고객 서비스 운영</li>
          <li>GitOps 기반 CI/CD 파이프라인 구축 및 자동화</li>
          <li>OpenTelemetry 기반 Azure Monitor 모니터링 체계 구축
            <a href="https://unggu.dev/lessons-learned/devops/proxy환경-X-Forwared-For/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
          <li>Alert 웹훅 → LLM 기반 쿼리 생성 → Log Analytics 조회 → 자동 원인 분석까지 이어지는 AI Agent 파이프라인 개발</li>
          <li>Nginx Ingress EOL 대응을 위한 Gateway API 이관
            <a href="https://unggu.dev/k8s/cka/Gateway/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
          <li>Key Vault 기반 시크릿·권한 관리 체계 구축
            <a href="https://unggu.dev/lessons-learned/devops/AzureKV-and-CSI-Pod/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
        </ul>
      </div>
    </div>
  </div>
</div>

---

## 🔍 문제 해결 사례

<div class="row g-3 mb-5">

  <div class="col-12">
    <div class="card border-danger shadow-sm">
      <div class="card-header bg-danger text-white">
        <h6 class="mb-0 fw-bold"><i class="fas fa-memory me-2"></i>Pod 메모리 92% 도달 — GC 트리거 조건 분석</h6>
      </div>
      <div class="card-body">
        <p class="mb-3">운영 중인 서비스의 Pod가 memory limit 1Gi 대비 RSS 920Mi까지 도달해 OOM 위험에 놓였습니다. 재기동이나 증설로 당장은 넘길 수 있었지만 <strong>"얼마나 늘려야 충분한지"조차 알 수 없다</strong>고 판단해 원인 분석부터 진행했습니다.</p>
        <button class="btn btn-outline-danger w-100" type="button" data-bs-toggle="collapse" data-bs-target="#case1Details" aria-expanded="false">
          <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
        </button>
        <div class="collapse" id="case1Details">
          <div class="card card-body bg-light mt-3">
            <p><strong>제약</strong> — heap_dump / class_histogram은 필연적으로 Full GC를 유발해, 터지기 직전의 운영 Pod에서는 사용할 수 없었습니다.</p>
            <p><strong>접근</strong> — GC 압박이 적은 <code>GC.heap_info</code>로 OOM 직전 Pod와 신규 기동 Pod를 비교해 Old Gen에서만 메모리가 누적되는 패턴을 확인했습니다. 이어 모니터링 툴에서 Mixed GC가 거의 동작하지 않는 것을 확인했습니다.</p>
            <p><strong>원인</strong> — GC 튜닝 없이 운영된 서비스로, MaxHeap 750Mi × 기본 IHOP 45% = 337Mi 초과 시에만 Mixed GC가 동작하는 상태였습니다. Non-Heap 350Mi와 Heap Committed를 합하면 <strong>메모리 사용률이 90%를 넘겨야 GC가 돌기 시작</strong>하는 구조였습니다.</p>
            <p><strong>선택</strong> — IHOP 하향은 GC 빈도 증가로 응답 지연을 유발할 수 있어, 대고객 트래픽을 처리하는 서비스 특성상 지연 증가를 감수하기 어렵다고 봤습니다. 노드 사용량에 여유가 있었으므로 Heap 750Mi + Non-Heap 350Mi의 실제 필요량을 근거로 limit을 1.5Gi로 재산정했습니다.</p>
            <table class="table table-bordered table-sm">
              <thead class="table-light"><tr><th>항목</th><th>AS-IS</th><th>TO-BE</th></tr></thead>
              <tbody>
                <tr><td>Mixed GC 동작 시점</td><td>MEM 90% 초과 이후</td><td>정상 주기 동작 확인</td></tr>
                <tr><td>Memory 사용률</td><td>92% (OOM 위험)</td><td>Spike 시에도 여유 확보</td></tr>
                <tr><td>Memory limit</td><td>1Gi (근거 없음)</td><td>1.5Gi (실사용량 기반 산정)</td></tr>
              </tbody>
            </table>
            <p class="mb-0">결과적으로 리소스를 늘린 결정이지만, Old Gen 누적 패턴부터 GC 트리거 조건까지 직접 계산하고 검증했기에 <strong>근거 있는 운영 작업</strong>이라고 볼 수 있었습니다.</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="col-12">
    <div class="card border-primary shadow-sm">
      <div class="card-header bg-primary text-white">
        <h6 class="mb-0 fw-bold"><i class="fas fa-robot me-2"></i>Alert 오탐 분석 자동화 — AI Agent 파이프라인</h6>
      </div>
      <div class="card-body">
        <p class="mb-3">Alert 대부분이 서비스 코드의 404나 관리 영역 Namespace에서 발생하는 오탐이었지만, <strong>실제 라우팅 장애도 동일한 404로 나타나기 때문에</strong> 임계치 조정으로 억제할 수 없었습니다. 로그를 열어보기 전까지 오탐과 장애를 구분할 방법이 없는 구조였습니다.</p>
        <button class="btn btn-outline-primary w-100" type="button" data-bs-toggle="collapse" data-bs-target="#case2Details" aria-expanded="false">
          <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
        </button>
        <div class="collapse" id="case2Details">
          <div class="card card-body bg-light mt-3">
            <p><strong>해결 방향</strong> — 임계치를 올려 노이즈를 줄이는 대신 <strong>분석 과정 자체를 자동화</strong>했습니다. Alert 웹훅을 수신해 상황에 맞는 KQL 쿼리를 생성하고, Log Analytics에 스스로 질의·응답을 반복해 원인을 도출하는 AI Agent 파이프라인을 구성했습니다.</p>
            <p><strong>인증 문제</strong> — 서비스 운영 환경과 AI가 올라가는 사내 PoC 환경의 테넌트가 달랐습니다. 앱 등록 시크릿 방식은 <strong>보안 통제 수준이 다른 PoC 환경에 운영 리소스 접근 권한을 상주시키는 구조</strong>가 되어 택하지 않았고, 대신 사용자 할당 관리 ID에 Workload Identity Federation을 구성해 Kubernetes ServiceAccount 토큰으로 인증 정보를 교환하도록 했습니다. 파드에 시크릿이 남지 않고 권한은 관리 ID의 RBAC으로만 제어됩니다. 인증이 기본 테넌트로 시도되는 문제는 토큰 요청 시 대상 테넌트를 명시해 해결했습니다.</p>
            <p><strong>검증</strong> — 실제 Alert으로 검증하는 과정에서 문서만 보고 작성한 쿼리가 결과를 반환하지 않는 경우를 발견했고, KubeEvents 테이블의 Pod 식별 컬럼명과 OOM 이벤트의 실제 명칭이 예상과 달랐던 점을 확인해 교정했습니다.</p>
            <p class="mb-0">반복되는 장애 대응 피로도를 줄인 이 경험은 함께 일하는 팀원들에게 가장 크게 공감받은 작업이기도 했습니다.</p>
          </div>
        </div>
      </div>
    </div>
  </div>

  <div class="col-12">
    <div class="card border-success shadow-sm">
      <div class="card-header bg-success text-white">
        <h6 class="mb-0 fw-bold"><i class="fas fa-clipboard-check me-2"></i>배포 후 수동 주문 점검 자동화 — 경량 모니터링 설계</h6>
      </div>
      <div class="card-body">
        <p class="mb-3">배포·운영 작업 후 담당자가 <strong>약 2시간</strong>에 걸쳐 도메인·조건별 주문 유입을 수동 확인하는 프로세스가 있었습니다. 반복 수작업인 데다 확인이 늦어지면 장애 인지 시점도 함께 늦어지는 구조였습니다.</p>
        <button class="btn btn-outline-success w-100" type="button" data-bs-toggle="collapse" data-bs-target="#case3Details" aria-expanded="false">
          <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
        </button>
        <div class="collapse" id="case3Details">
          <div class="card card-body bg-light mt-3">
            <p><strong>제약</strong> — 신규 인프라 구축이 어렵고, 대량의 주문 테이블에 부하를 줄 수 없었습니다. 별도 시스템을 새로 두기보다 기존 BO에 경량 모니터링 기능을 추가하는 방향으로 설계했습니다.</p>
            <p><strong>트레이드오프 비교</strong> — 유선/무선/상담/기타 4개 도메인 × 주문 방식별 32개 카테고리를 판별해야 했습니다. 전체 데이터를 애플리케이션에서 필터링하는 방식과 DB 쿼리로 처리하는 방식을 인덱스 활용도·유지보수성 기준으로 비교했고, 도메인당 일 주문건이 1,000건 수준으로 성능 차이가 크지 않음을 확인했습니다.</p>
            <p>이에 <strong>"지금 빠른가"보다 "규칙이 늘어나도 유지보수 가능한가"</strong>를 기준으로 삼아, 도메인당 DB 1차 필터링 + 코드 레벨 판별 로직을 결합한 구조로 결정했습니다.</p>
            <p><strong>범위 판단</strong> — 배치 서버에서 Cron으로 모니터링 서비스를 호출해 카테고리별 임계값과 비교하고 이상 시 메일을 전송하는 구조까지 확장 가능하도록 설계했으나, 원래 목적인 "배포 직후 확인"은 담당자가 대시보드를 직접 조회하는 것만으로 해결되었기에 조회 기능을 먼저 반영하는 것이 우선순위상 맞다고 판단했습니다. 이후 상시 모니터링으로 범위를 넓힐 경우 설계해둔 배치 호출 구조를 그대로 확장할 수 있도록 열어두었습니다.</p>
            <table class="table table-bordered table-sm mb-0">
              <thead class="table-light"><tr><th>항목</th><th>AS-IS</th><th>TO-BE</th></tr></thead>
              <tbody>
                <tr><td>배포 후 주문 점검</td><td>담당자 수동 확인 약 2시간</td><td>대시보드 조회로 즉시 확인</td></tr>
                <tr><td>판별 대상</td><td>4개 도메인 수작업</td><td>32개 카테고리 자동 판별</td></tr>
                <tr><td>추가 인프라</td><td>-</td><td>0 (기존 BO에 통합)</td></tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>
    </div>
  </div>

</div>

---

## 🛠 기술 스택

<div class="card shadow-sm mb-4">
  <div class="card-body">
    <div class="row g-4">
      <div class="col-12">
        <h6 class="text-primary mb-3">
          <i class="fas fa-code"></i> Language &amp; Framework
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Java</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Spring Framework / Spring Boot</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>RESTful API</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Swift / SwiftUI</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Javascript</li>
        </ul>
      </div>
      <div class="col-md-4">
        <h6 class="text-secondary mb-3">
          <i class="fas fa-database"></i> Database &amp; Cache
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>PostgreSQL</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>MySQL</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Redis</li>
        </ul>
      </div>
      <div class="col-md-4">
        <h6 class="text-info mb-3">
          <i class="fas fa-stream"></i> Messaging
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Kafka (Azure Event Hub)</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Pub-Sub 패턴</li>
        </ul>
      </div>
      <div class="col-md-4">
        <h6 class="text-danger mb-3">
          <i class="fas fa-chart-line"></i> Monitoring
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Azure Monitor / Log Analytics(KQL)</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>OpenTelemetry</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Jennifer</li>
        </ul>
      </div>
      <div class="col-md-6">
        <h6 class="text-success mb-3">
          <i class="fas fa-infinity"></i> DevOps &amp; CI/CD
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Kubernetes / AKS</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Docker</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Helm &amp; Kustomize</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>GitHub Actions &amp; Argo CD</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Azure (Key Vault, Managed Identity, RBAC)</li>
        </ul>
      </div>
      <div class="col-md-6">
        <h6 class="text-warning mb-3">
          <i class="fas fa-users"></i> Collaboration
        </h6>
        <ul class="list-unstyled ms-3">
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Git</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>JIRA &amp; Confluence</li>
          <li class="mb-2"><i class="fas fa-check-circle text-success me-2"></i>Notion</li>
        </ul>
      </div>
    </div>
  </div>
</div>

---

## 🚀 프로젝트

### 하이소피 - AI 기반 리뷰 피드백 시스템

<div class="card mb-5 border-success shadow-sm">
  <div class="card-header bg-success text-white">
    <div class="d-flex justify-content-between align-items-center">
      <div>
        <h5 class="mb-1 fw-bold">
          <i class="fas fa-robot me-2"></i>소상공인을 위한 AI 리뷰 관리 솔루션
        </h5>
        <small>2025.05 ~ 2025.07 | KT DS 사내 교육 MVP | Backend &amp; DevOps</small>
      </div>
      <span class="badge bg-light text-dark px-3 py-2">완료</span>
    </div>
  </div>

  <div class="card-body">
    <p class="mb-3">다양한 플랫폼의 리뷰를 자동 수집·분석하고, ChatGPT API가 생성한 맞춤형 실행계획으로 매장 운영 개선을 지원하는 MSA 기반 웹 서비스</p>

    <div class="mb-4">
      <h6 class="text-muted mb-3">
        <i class="fas fa-layer-group me-2"></i>기술 스택
      </h6>
      <div class="p-3 bg-light rounded">
        <span class="badge bg-primary me-2 mb-2 px-3 py-2"><i class="fas fa-leaf me-1"></i>Spring Boot</span>
        <span class="badge bg-primary me-2 mb-2 px-3 py-2"><i class="fab fa-react me-1"></i>React</span>
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fas fa-cloud me-1"></i>AKS</span>
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fas fa-stream me-1"></i>Event Hub (Kafka)</span>
        <span class="badge bg-info me-2 mb-2 px-3 py-2"><i class="fas fa-database me-1"></i>PostgreSQL</span>
        <span class="badge bg-danger me-2 mb-2 px-3 py-2"><i class="fas fa-memory me-1"></i>Redis</span>
        <span class="badge bg-secondary me-2 mb-2 px-3 py-2"><i class="fas fa-sync-alt me-1"></i>Argo CD</span>
      </div>
    </div>

    <div class="mb-3">
      <button class="btn btn-outline-primary w-100" type="button" data-bs-toggle="collapse" data-bs-target="#project1Details" aria-expanded="false">
        <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
      </button>
    </div>

    <div class="collapse" id="project1Details">
      <div class="card card-body bg-light mt-3">
        <h6 class="fw-bold text-success">
          <i class="fas fa-bullseye me-2"></i>1. MSA 아키텍처 설계
        </h6>
        <ul>
          <li>리뷰 수집·분석·알림 도메인별 독립 서비스로 분리, 기획 담당자와 협업해 요구사항 기반 서비스 구조 설계</li>
          <li>Azure Event Hub(Kafka) 기반 Pub-Sub 구조로 서비스 간 비동기 이벤트 통신 설계, 리뷰 수집·분석 실시간 데이터 파이프라인 구성
            <a href="https://unggu.dev/devops/container/Pub-Sub-패턴과-Azure-Event-Hub/" target="_blank" title="관련 글"><i class="fas fa-link ms-1"></i></a>
          </li>
          <li>서비스 간 결합도를 낮추기 위해 PostgreSQL DB를 서비스별로 격리</li>
        </ul>

        <h6 class="fw-bold text-success">
          <i class="fas fa-rocket me-2"></i>2. 개발 및 배포
        </h6>
        <ul>
          <li>AI 코딩 도구를 활용해 설계한 구조를 빠르게 프로토타입으로 구현</li>
          <li>GitHub Actions + ArgoCD 기반 GitOps 파이프라인 구축, AKS에 각 MSA 배포 자동화</li>
        </ul>

        <div class="alert alert-info mt-3">
          <strong><i class="fas fa-graduation-cap me-2"></i>학습 성과</strong>
          <p class="mb-0 mt-2">MSA 및 클라우드 디자인 패턴을 이론으로 학습한 후 2주 Sprint로 실제 MVP까지 구현하며 Cloud Native 애플리케이션 개발의 전 과정을 경험했습니다.
          이를 바탕으로 실무 이슈를 디자인 패턴으로 해결할 수 있는 지점을 찾아보고 있습니다.</p>
        </div>

        <a href="https://github.com/ktds-garage-04" class="btn btn-dark mt-2" target="_blank">
          <i class="fab fa-github me-2"></i>GitHub Repository
        </a>
      </div>
    </div>
  </div>
</div>

### KT 대고객 서비스 Azure Migration

<div class="card mb-5 border-primary shadow-sm">
  <div class="card-header bg-primary text-white">
    <div class="d-flex justify-content-between align-items-center">
      <div>
        <h5 class="mb-1 fw-bold">
          <i class="fas fa-cloud me-2"></i>사내 클라우드 환경 → Azure 환경 전환
        </h5>
        <small>2024.10 ~ 2025.03 | KT DS | DevOps</small>
      </div>
      <span class="badge bg-light text-dark px-3 py-2">완료</span>
    </div>
  </div>

  <div class="card-body">
    <p class="mb-3">KT Cloud에서 Azure AKS 환경으로의 대고객 서비스 마이그레이션 프로젝트</p>

    <div class="mb-4">
      <h6 class="text-muted mb-3">
        <i class="fas fa-layer-group me-2"></i>기술 스택
      </h6>
      <div class="p-3 bg-light rounded">
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fas fa-dharmachakra me-1"></i>Kubernetes</span>
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fas fa-cloud me-1"></i>AKS</span>
        <span class="badge bg-info me-2 mb-2 px-3 py-2"><i class="fab fa-microsoft me-1"></i>Azure Key Vault</span>
        <span class="badge bg-primary me-2 mb-2 px-3 py-2"><i class="fas fa-leaf me-1"></i>Spring</span>
        <span class="badge bg-danger me-2 mb-2 px-3 py-2"><i class="fas fa-memory me-1"></i>Redis</span>
        <span class="badge bg-warning text-dark me-2 mb-2 px-3 py-2"><i class="fas fa-network-wired me-1"></i>App Gateway</span>
        <span class="badge bg-secondary me-2 mb-2 px-3 py-2"><i class="fas fa-ship me-1"></i>Kustomize</span>
        <span class="badge bg-secondary me-2 mb-2 px-3 py-2"><i class="fas fa-gauge-high me-1"></i>Azure Monitor</span>
      </div>
    </div>

    <div class="alert alert-success mb-4">
      <h6 class="alert-heading mb-3">
        <i class="fas fa-trophy me-2"></i>주요 성과
      </h6>
      <div class="row text-center g-3">
        <div class="col-6 col-md-3">
          <strong class="d-block fs-5 text-success">무중단</strong>
          <small class="text-muted">전환 성공</small>
        </div>
        <div class="col-6 col-md-3">
          <strong class="d-block fs-5 text-success">15분 → 5분</strong>
          <small class="text-muted">배포 소요 시간</small>
        </div>
        <div class="col-6 col-md-3">
          <strong class="d-block fs-5 text-success">30분 → 5분</strong>
          <small class="text-muted">장애 복구 시간</small>
        </div>
        <div class="col-6 col-md-3">
          <strong class="d-block fs-5 text-success">GitOps</strong>
          <small class="text-muted">CI/CD 구축</small>
        </div>
      </div>
    </div>

    <div class="mb-3">
      <button class="btn btn-outline-primary w-100" type="button" data-bs-toggle="collapse" data-bs-target="#project2Details" aria-expanded="false">
        <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
      </button>
    </div>

    <div class="collapse" id="project2Details">
      <div class="card card-body bg-light mt-3">
        <h6 class="fw-bold text-primary">
          <i class="fas fa-lightbulb me-2"></i>프로젝트 배경
        </h6>
        <p>입사 후 첫 프로젝트로, Kubernetes/Docker 지식이 전무한 상태에서 시작했습니다. 퇴근 후 스스로 학습해 CKA·AZ-204 자격증을 취득했고, 이를 기반으로 프로젝트를 완료한 뒤 사내에 지식을 전파했습니다.</p>

        <hr>

        <h6 class="fw-bold text-primary">
          <i class="fas fa-bullseye me-2"></i>핵심 과제
        </h6>

        <p><strong>1. CI/CD 파이프라인 구축</strong></p>
        <ul>
          <li>GitHub Actions를 통한 환경별 멀티 빌드 환경 구축 (사내 저장소 이용)</li>
          <li>ArgoCD 기반 안정적인 배포 환경 구축</li>
        </ul>

        <p><strong>2. Kubernetes 환경 구축</strong></p>
        <ul>
          <li>Kustomize + GitOps 방식의 yaml 관리</li>
          <li>Nginx Ingress 기반 라우팅 정책 수립 및 TLS 설정</li>
          <li>Pod 리소스 설정 및 Health Check 설정</li>
        </ul>

        <p><strong>3. 모니터링 환경 구축</strong></p>
        <ul>
          <li>Java Agent 기반 Azure Monitor 환경 구축</li>
          <li>Alert rule 및 Action Group 설정을 통한 실시간 알림 체계 구축</li>
          <li>Log sampling 적용을 통한 LAW 데이터 수집량 감소 및 비용 최적화</li>
        </ul>

        <p><strong>4. 비밀/권한 설정</strong></p>
        <ul>
          <li>Key Vault 연동을 통해 CI/CD 및 소스 코드 내 민감정보를 Secret으로 관리</li>
          <li>관리 ID / Federation 설정 및 RBAC 설정 관리</li>
        </ul>

        <hr>

        <h6 class="fw-bold text-primary">
          <i class="fas fa-chart-line me-2"></i>성과 지표
        </h6>
        <div class="table-responsive">
          <table class="table table-bordered table-sm">
            <thead class="table-light">
              <tr><th>항목</th><th>AS-IS</th><th>TO-BE</th></tr>
            </thead>
            <tbody>
              <tr><td>장애 복구 시간</td><td>30분</td><td>5분</td></tr>
              <tr><td>장애 건수</td><td>5회/년</td><td>1회/년</td></tr>
              <tr><td>배포 소요 시간</td><td>15분</td><td>5분</td></tr>
            </tbody>
          </table>
        </div>

        <div class="alert alert-warning mt-3">
          <strong><i class="fas fa-exclamation-triangle me-2"></i>가장 힘들었던 점</strong>
          <p class="mb-0 mt-2">AppGW의 X-Forwarded-For 헤더를 Nginx Ingress가 바라보는 AppGW IP로 덮어써서, 이를 우회하려 Ingress annotation에 헤더 재작성 설정을 추가한 적이 있습니다.

          이때 Azure 관리형 Nginx에서 중괄호가 금지 문법인 걸 몰라 설정이 조용히 롤백된 채 운영되다가, 약 10일 뒤 Azure 측 강제 재기동으로 nginx.conf 생성이 실패해 <strong>서비스 전면 404 장애</strong>로 번졌습니다.

          설정 제거로 긴급 복구했고 Client IP는 Azure Monitor 커스텀 헤더 수집으로 대체했습니다. 이 경험으로 관리형 서비스는 편리함 뒤에 제한의 영역이 있다는 것, 그 제약을 미리 확인해야 한다는 것을 배웠습니다.</p>
          <div class="mt-2">
            <a href="https://unggu.dev/lessons-learned/devops/proxy환경-X-Forwared-For/" target="_blank" class="d-inline-block me-3"><i class="fas fa-link me-1"></i>XFF 이슈 글 보기</a>
            <a href="https://unggu.dev/lessons-learned/devops/Azure-Ingress-이슈/" target="_blank" class="d-inline-block"><i class="fas fa-link me-1"></i>404 장애 글 보기</a>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

### 여행지니 - 사진 기반 여행 추천 시스템

<div class="card mb-5 border-info shadow-sm">
  <div class="card-header bg-info text-white">
    <div class="d-flex justify-content-between align-items-center">
      <div>
        <h5 class="mb-1 fw-bold">
          <i class="fas fa-map-marked-alt me-2"></i>여행을 지니다 + 여행을 추천하다
        </h5>
        <small>2024.05 ~ 2024.06 | SSAFY (2인) | Backend &amp; Frontend</small>
      </div>
      <span class="badge bg-light text-dark px-3 py-2">완료</span>
    </div>
  </div>

  <div class="card-body">
    <p class="mb-3">사진 기반 여행지 정보 추천 웹 서비스 (프로젝트 종료 후 개인 학습 목적으로 컨테이너화 및 클라우드 CI/CD 구축)</p>

    <div class="mb-4">
      <h6 class="text-muted mb-3">
        <i class="fas fa-layer-group me-2"></i>기술 스택
      </h6>
      <div class="p-3 bg-light rounded">
        <span class="badge bg-primary me-2 mb-2 px-3 py-2"><i class="fas fa-leaf me-1"></i>Spring Boot</span>
        <span class="badge bg-primary me-2 mb-2 px-3 py-2"><i class="fab fa-vuejs me-1"></i>Vue.js</span>
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fab fa-docker me-1"></i>Docker</span>
        <span class="badge bg-success me-2 mb-2 px-3 py-2"><i class="fas fa-sync-alt me-1"></i>Argo CD</span>
        <span class="badge bg-info me-2 mb-2 px-3 py-2"><i class="fas fa-database me-1"></i>MySQL</span>
      </div>
    </div>

    <div class="mb-3">
      <button class="btn btn-outline-primary w-100" type="button" data-bs-toggle="collapse" data-bs-target="#project3Details" aria-expanded="false">
        <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
      </button>
    </div>

    <div class="collapse" id="project3Details">
      <div class="card card-body bg-light mt-3">
        <h6 class="fw-bold text-info">
          <i class="fas fa-lightbulb me-2"></i>프로젝트 배경
        </h6>
        <p>SSAFY 과정 중 2인 팀 프로젝트로 FrontEnd/Backend를 분담해 개발했고, 종료 후 <strong>개인 학습 목적</strong>으로 기존 서비스를 컨테이너화해 클라우드 환경에 CI/CD 파이프라인을 구축했습니다.</p>

        <hr>

        <h6 class="fw-bold text-info">
          <i class="fas fa-bullseye me-2"></i>주요 개선사항
        </h6>
        <div class="table-responsive">
          <table class="table table-bordered table-sm">
            <thead class="table-light"><tr><th>항목</th><th>AS-IS</th><th>TO-BE</th></tr></thead>
            <tbody>
              <tr><td>아키텍처</td><td>전통적인 모놀리식</td><td>Docker 컨테이너화로 환경 일관성 확보</td></tr>
              <tr><td>배포</td><td>수동 배포</td><td>GitOps 기반 자동 배포</td></tr>
              <tr><td>실행 환경</td><td>단일 서버</td><td>Kubernetes 기반 오케스트레이션</td></tr>
            </tbody>
          </table>
        </div>

        <div class="alert alert-success mt-3">
          <strong><i class="fas fa-graduation-cap me-2"></i>학습 성과</strong>
          <p class="mb-0 mt-2">프로젝트를 단순히 끝내지 않고 실무에서 사용되는 DevOps 기술을 적용해 클라우드 네이티브 애플리케이션으로 발전시키며, 이후 Azure Migration 실무 역량 향상에 큰 도움이 되었습니다.</p>
        </div>

        <a href="https://github.com/JourneyJinni" class="btn btn-dark mt-3" target="_blank">
          <i class="fab fa-github me-2"></i>GitHub Repository
        </a>
      </div>
    </div>
  </div>
</div>

### 나주 버스 - iOS 앱

<div class="card mb-5 border-warning shadow-sm">
  <div class="card-header bg-warning text-dark">
    <div class="d-flex justify-content-between align-items-center">
      <div>
        <h5 class="mb-1 fw-bold">
          <i class="fas fa-bus me-2"></i>나주시 버스 도착 정보 제공 앱
        </h5>
        <small>2023.03 ~ 2023.09 개발 · 3년간 운영 | 개인 프로젝트 | iOS App</small>
      </div>
      <span class="badge bg-secondary px-3 py-2">운영 종료</span>
    </div>
  </div>

  <div class="card-body">
    <p class="mb-3">SwiftUI 기반 iOS 앱 — App Store 정식 배포 | 다운로드 5,000회 | 서버리스 아키텍처</p>

    <div class="mb-4">
      <h6 class="text-muted mb-3">
        <i class="fas fa-layer-group me-2"></i>기술 스택
      </h6>
      <div class="p-3 bg-light rounded">
        <span class="badge bg-info me-2 mb-2 px-3 py-2"><i class="fab fa-swift me-1"></i>SwiftUI</span>
        <span class="badge bg-info me-2 mb-2 px-3 py-2"><i class="fab fa-apple me-1"></i>iOS</span>
        <span class="badge bg-secondary me-2 mb-2 px-3 py-2"><i class="fas fa-database me-1"></i>CoreData</span>
      </div>
    </div>

    <div class="mb-3">
      <button class="btn btn-outline-primary w-100" type="button" data-bs-toggle="collapse" data-bs-target="#project4Details" aria-expanded="false">
        <i class="fas fa-chevron-down me-2"></i>상세 내용 보기
      </button>
    </div>

    <div class="collapse" id="project4Details">
      <div class="card card-body bg-light mt-3">
        <h6 class="fw-bold text-warning">
          <i class="fas fa-lightbulb me-2"></i>개발 동기
        </h6>
        <p>나주시에는 버스 정보를 제공하는 앱이 없어, 공공 API를 활용해 직접 개발했습니다. 버스 정류장·노선 검색과 도착까지 남은 시간, 노선의 현재 위치를 제공합니다.</p>

        <hr>

        <h6 class="fw-bold text-warning">
          <i class="fas fa-bullseye me-2"></i>기술적 특징
        </h6>

        <p><strong>서버리스 아키텍처</strong></p>
        <ul>
          <li>공공 API 직접 호출로 별도 백엔드 서버 불필요</li>
          <li>CoreData를 활용한 로컬 데이터 캐싱</li>
          <li>인프라 비용 0원으로 3년간 서비스 제공</li>
          <li>Google AdMob을 통한 수익화 실현</li>
        </ul>

        <p><strong>SwiftUI 활용</strong></p>
        <ul>
          <li>선언형 UI로 유지보수 용이</li>
          <li>iOS 네이티브 성능 최적화</li>
          <li>다크모드 자동 지원</li>
        </ul>

        <div class="alert alert-success mt-3">
          <strong><i class="fas fa-smile me-2"></i>느낀점</strong>
          <p class="mb-0 mt-2">실제 사용자가 있는 앱을 3년간 운영하며 버그 수정, 기능 개선, iOS 버전 업데이트 대응을 직접 겪었습니다.
          사용자 피드백을 반영해 서비스를 고쳐나가는 경험이 개발자로서의 첫걸음이 되었습니다.</p>
        </div>

        <div class="mt-3">
          <a href="https://apps.apple.com/kr/app/나주시-버스/id6459411077" class="btn btn-dark me-2" target="_blank">
            <i class="fab fa-app-store me-2"></i>App Store
          </a>
          <a href="https://github.com/unggu0704/naju-busInfo" class="btn btn-dark" target="_blank">
            <i class="fab fa-github me-2"></i>GitHub
          </a>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
// Collapse 열릴 때 chevron 아이콘 회전
document.addEventListener('DOMContentLoaded', function() {
  const collapseElements = document.querySelectorAll('[data-bs-toggle="collapse"]');

  collapseElements.forEach(function(element) {
    const targetId = element.getAttribute('data-bs-target');
    const targetElement = document.querySelector(targetId);

    if (targetElement) {
      targetElement.addEventListener('shown.bs.collapse', function() {
        const icon = element.querySelector('.fa-chevron-down');
        if (icon) {
          icon.classList.remove('fa-chevron-down');
          icon.classList.add('fa-chevron-up');
        }
      });

      targetElement.addEventListener('hidden.bs.collapse', function() {
        const icon = element.querySelector('.fa-chevron-up');
        if (icon) {
          icon.classList.remove('fa-chevron-up');
          icon.classList.add('fa-chevron-down');
        }
      });
    }
  });
});
</script>

---

## 🎓 학력 & 교육

<div class="card border-secondary shadow-sm mb-4">
  <div class="card-body">
    <div class="d-flex justify-content-between align-items-start">
      <div>
        <h6 class="mb-1 fw-bold">전북대학교 컴퓨터공학과</h6>
        <p class="mb-0 text-muted small">학점 3.98 / 4.5</p>
      </div>
      <span class="text-muted small">2024.02 졸업</span>
    </div>
  </div>
</div>

### 교육 과정

<div class="row g-3 mb-4">
  <div class="col-md-4">
    <div class="card h-100 border-primary">
      <div class="card-body">
        <div class="d-flex justify-content-between align-items-start mb-2">
          <h6 class="card-subtitle mb-0 text-primary">KTDS Digital Garage</h6>
          <span class="text-muted" style="font-size: 0.75rem;">2025.05 ~ 2025.07</span>
        </div>
        <p class="card-text small mb-0">5주간 사내 교육 + 2주 Sprint MVP 개발. Java Spring/React 풀스택, 6개 MSA 분리 및 AKS CI/CD 구축, Event Hub(Kafka) 기반 비동기 이벤트 파이프라인, Key Vault 연동 등 CSP 클라우드 아키텍처 패턴 학습</p>
      </div>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card h-100 border-success">
      <div class="card-body">
        <div class="d-flex justify-content-between align-items-start mb-2">
          <h6 class="card-subtitle mb-0 text-success">삼성청년SW아카데미 11기</h6>
          <span class="text-muted" style="font-size: 0.75rem;">2024.01 ~ 2024.07</span>
        </div>
        <p class="card-text small mb-0">알고리즘·자료구조 기반 Java 프로그래밍 역량 강화, 실무 환경과 동일한 방식의 자기주도형 팀 프로젝트 수행 (Spring Boot / Vue.js / MySQL / Docker)</p>
      </div>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card h-100 border-info">
      <div class="card-body">
        <div class="d-flex justify-content-between align-items-start mb-2">
          <h6 class="card-subtitle mb-0 text-info">NHN 아카데미</h6>
          <span class="text-muted" style="font-size: 0.75rem;">2023.08 ~ 2023.12</span>
        </div>
        <p class="card-text small mb-0">Java 심화 학습, Thread-Safe 개발 및 TDD 경험, Servlet/JSP 기반 Web Application 개발 경험</p>
      </div>
    </div>
  </div>
</div>

### 보유 자격증

<div class="row g-2 mb-4">
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-certificate fa-2x text-info me-3"></i>
          <div>
            <h6 class="mb-0 small">Certified Kubernetes Administrator (CKA)</h6>
            <small class="text-muted">2025.08 · The Linux Foundation</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-certificate fa-2x text-primary me-3"></i>
          <div>
            <h6 class="mb-0 small">Azure Developer Associate (AZ-204)</h6>
            <small class="text-muted">2025.02 · Microsoft</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-certificate fa-2x text-success me-3"></i>
          <div>
            <h6 class="mb-0 small">Azure Fundamentals (AZ-900)</h6>
            <small class="text-muted">2024.12 · Microsoft</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-database fa-2x text-warning me-3"></i>
          <div>
            <h6 class="mb-0 small">SQLD</h6>
            <small class="text-muted">2024.06 · 한국데이터산업진흥원</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-code fa-2x text-danger me-3"></i>
          <div>
            <h6 class="mb-0 small">정보처리기사</h6>
            <small class="text-muted">2023.06 · 한국산업인력공단</small>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div class="col-lg-4 col-md-6">
    <div class="card h-100 shadow-sm">
      <div class="card-body p-3">
        <div class="d-flex align-items-center">
          <i class="fas fa-language fa-2x text-secondary me-3"></i>
          <div>
            <h6 class="mb-0 small">TOEIC 845 / TOPCIT 수준3</h6>
            <small class="text-muted">2023.02 / 2023.05</small>
          </div>
        </div>
      </div>
    </div>
  </div>
</div>

---

## 📝 기술 블로그

<div class="list-group mb-3">
{% for post in site.posts limit:5 %}
  <a href="{{ post.url | relative_url }}" class="list-group-item list-group-item-action d-flex justify-content-between align-items-start">
    <div class="ms-2 me-auto">
      <div class="fw-bold">{{ post.title }}</div>
    </div>
    <span class="badge bg-secondary rounded-pill">{{ post.date | date: "%Y.%m.%d" }}</span>
  </a>
{% endfor %}
</div>

<a href="/archives/" class="btn btn-outline-secondary">
  <i class="fas fa-archive"></i> 전체 포스트 보기
</a>

---

## 📞 Contact

<div class="text-center mb-4">
  <div class="row g-3 justify-content-center">
    <div class="col-6 col-md-3">
      <a href="mailto:unggu556@naver.com" class="text-decoration-none d-block">
        <div class="card h-100 border-primary shadow-sm hover-shadow">
          <div class="card-body text-center py-4">
            <i class="fas fa-envelope fa-2x text-primary mb-2"></i>
            <p class="mb-0 small fw-bold">Email</p>
          </div>
        </div>
      </a>
    </div>
    <div class="col-6 col-md-3">
      <a href="https://github.com/unggu0704" class="text-decoration-none d-block" target="_blank">
        <div class="card h-100 border-dark shadow-sm hover-shadow">
          <div class="card-body text-center py-4">
            <i class="fab fa-github fa-2x text-dark mb-2"></i>
            <p class="mb-0 small fw-bold">GitHub</p>
          </div>
        </div>
      </a>
    </div>
    <div class="col-6 col-md-3">
      <a href="https://www.linkedin.com/in/규형-김-5b5b10299" class="text-decoration-none d-block" target="_blank">
        <div class="card h-100 border-info shadow-sm hover-shadow">
          <div class="card-body text-center py-4">
            <i class="fab fa-linkedin fa-2x text-info mb-2"></i>
            <p class="mb-0 small fw-bold">LinkedIn</p>
          </div>
        </div>
      </a>
    </div>
    <div class="col-6 col-md-3">
      <a href="https://unggu.dev" class="text-decoration-none d-block" target="_blank">
        <div class="card h-100 border-success shadow-sm hover-shadow">
          <div class="card-body text-center py-4">
            <i class="fas fa-blog fa-2x text-success mb-2"></i>
            <p class="mb-0 small fw-bold">Blog</p>
          </div>
        </div>
      </a>
    </div>
  </div>
</div>

<style>
.hover-shadow:hover {
  transform: translateY(-5px);
  transition: all 0.3s ease;
  box-shadow: 0 0.5rem 1rem rgba(0, 0, 0, 0.15) !important;
}
</style>

---

> 함께 성장할 수 있는 기회가 있다면 언제든 연락 주세요! 😊
{: .prompt-tip }
