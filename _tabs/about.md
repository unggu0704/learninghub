# Jekyll / Chirpy 포트폴리오 문법 테스트 문서

> 이 문서는 **일반 Markdown vs Jekyll(Chirpy) 확장 문법**이 실제로 어떻게 다르게 동작하는지 확인하기 위한 테스트용 문서입니다.
> 그대로 `about.md` 또는 `_tabs/portfolio.md` 에 붙여서 결과를 확인하세요.

---

## 0. Front Matter 테스트 (필수)

```yaml
---
title: Portfolio
icon: fas fa-briefcase
order: 2
---
```

* 위 영역은 **Markdown이 아님**
* 페이지 메타데이터 + 테마 제어용

---

## 1. 일반 Markdown 렌더링 테스트

### 1-1. 기본 요소

* 리스트 아이템 1
* 리스트 아이템 2
* **Bold 텍스트**
* *Italic 텍스트*
* `inline code`

> 인용문 테스트

---

## 2. Markdown + HTML 혼합 테스트 (중요)

> Jekyll은 Markdown 내부에 **HTML을 그대로 허용**합니다.

<div class="card">
  <h3>JourneyJinni</h3>
  <p><strong>설명:</strong> AKS 기반 GitOps 아키텍처 서비스</p>
  <ul>
    <li>Argo CD</li>
    <li>Kubernetes</li>
    <li>Azure Container Registry</li>
  </ul>
</div>

---

## 3. Chirpy 전용 스타일 문법 테스트

### 3-1. Prompt 스타일

{: .prompt-tip }
이 프로젝트는 실제 운영 환경을 고려하여 설계되었습니다.

{: .prompt-info }

* CI/CD: GitHub Actions
* Deployment: Argo CD

{: .prompt-warning }
실제 서비스 환경에서는 보안 설정이 추가됩니다.

---

## 4. 이미지 + 캡션 테스트

![Sample Image](/assets/img/sample.png){: width="400" }

> 이미지 아래 설명 텍스트

---

## 5. 코드 블록 + 하이라이팅

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: sample-app
```

```bash
kubectl get pods
```

---

## 6. Liquid 문법 테스트 (Jekyll 전용)

### 6-1. 페이지 메타데이터 출력

* 페이지 제목: **{{ page.title }}**
* 현재 URL: **{{ page.url }}**
* 사이트 이름: **{{ site.title }}**

### 6-2. 조건문 테스트

{% if page.url contains 'portfolio' %}
이 문장은 portfolio 페이지에서만 보입니다.
{% endif %}

---

## 7. 반복문 테스트 (포스트 목록)

{% for post in site.posts limit:3 %}

* {{ post.title }} ({{ post.date | date: "%Y-%m-%d" }})
  {% endfor %}

---

## 8. 테이블 테스트 (경력 요약)

| 기간   | 역할      | 기술                  |
| ---- | ------- | ------------------- |
| 2023 | Backend | Spring Boot, JPA    |
| 2024 | DevOps  | Kubernetes, Argo CD |

---

## 9. 앵커 링크 테스트

* [프로젝트 섹션으로 이동](#2-markdown--html-혼합-테스트-중요)

---

## 10. 실패/비권장 문법 테스트

```md
::: warning
이 문법은 일부 Markdown 엔진에서만 동작합니다.
:::
```

> 위 문법은 **Chirpy에서는 동작하지 않을 수 있음**

---

## 11. 결론

* 이 문서는 **Markdown + HTML + Liquid + 테마 규칙**이 혼합된 구조입니다.
* GitHub README에서는 동일하게 렌더링되지 않습니다.
* 포트폴리오 작성 시 **HTML + prompt 스타일 적극 활용**을 권장합니다.
