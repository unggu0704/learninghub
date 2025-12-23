---
title: About
icon: fas fa-info-circle
order: 4
---

# Chirpy 테마 디자인 가이드

> 이 페이지는 Jekyll Chirpy 테마에서 사용 가능한 모든 디자인 요소를 정리한 가이드입니다.
> 필요한 섹션을 복사해서 사용하세요!
{: .prompt-tip }

---

## 📌 1. Prompt 스타일 (핵심 기능)

Chirpy 테마의 가장 강력한 디자인 요소입니다. 4가지 타입이 있습니다.

> 유용한 팁이나 성공 사례를 표시할 때 사용합니다.
{: .prompt-tip }

> 추가 정보나 참고사항을 표시할 때 사용합니다.
{: .prompt-info }

> 주의가 필요하거나 중요한 내용을 강조할 때 사용합니다.
{: .prompt-warning }

> 위험하거나 치명적인 오류, 절대 하지 말아야 할 것을 표시합니다.
{: .prompt-danger }

### 사용법
```markdown
> 여기에 내용을 작성합니다.
{: .prompt-tip }
```

**주의사항:**
- `>` blockquote 바로 다음 줄에 `{: .prompt-타입 }` 작성
- 빈 줄이 있으면 안 됨!

---

## 🎨 2. Bootstrap 카드 레이아웃

### 기본 카드
<div class="card mb-3">
  <div class="card-body">
    <h5 class="card-title">카드 제목</h5>
    <p class="card-text">카드 설명입니다. 프로젝트나 기술 스택을 깔끔하게 정리할 수 있습니다.</p>
    <a href="#" class="btn btn-primary">자세히 보기</a>
  </div>
</div>

### 그리드 카드 (3컬럼)
<div class="row g-3 mb-4">
  <div class="col-md-4">
    <div class="card h-100">
      <div class="card-body">
        <h5 class="card-title">Backend</h5>
        <span class="badge bg-primary me-1">Java</span>
        <span class="badge bg-primary me-1">Spring</span>
        <span class="badge bg-primary">JPA</span>
      </div>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card h-100">
      <div class="card-body">
        <h5 class="card-title">DevOps</h5>
        <span class="badge bg-success me-1">Docker</span>
        <span class="badge bg-success me-1">K8s</span>
        <span class="badge bg-success">AKS</span>
      </div>
    </div>
  </div>
  <div class="col-md-4">
    <div class="card h-100">
      <div class="card-body">
        <h5 class="card-title">Cloud</h5>
        <span class="badge bg-info me-1">Azure</span>
        <span class="badge bg-info">AZ-204</span>
      </div>
    </div>
  </div>
</div>

### 사용법
```html
<div class="card">
  <div class="card-body">
    <h5 class="card-title">제목</h5>
    <p class="card-text">내용</p>
  </div>
</div>
```

---

## 🏷️ 3. 뱃지 (Badge)

<div class="mb-3">
  <span class="badge bg-primary me-1">Primary</span>
  <span class="badge bg-secondary me-1">Secondary</span>
  <span class="badge bg-success me-1">Success</span>
  <span class="badge bg-danger me-1">Danger</span>
  <span class="badge bg-warning text-dark me-1">Warning</span>
  <span class="badge bg-info me-1">Info</span>
  <span class="badge bg-dark">Dark</span>
</div>

### 사용법
```html
<span class="badge bg-primary">Java</span>
<span class="badge bg-success">Docker</span>
```

---

## 🔽 4. Details (접기/펼치기)

<details>
  <summary><strong>📦 클릭해서 내용 보기</strong></summary>

  <div class="mt-3">
    <h4>숨겨진 내용</h4>
    <ul>
      <li>상세 설명 1</li>
      <li>상세 설명 2</li>
      <li>Markdown **문법**도 사용 가능</li>
    </ul>

    ```java
    // 코드 블록도 포함 가능
    public class Example {
        public static void main(String[] args) {
            System.out.println("Hello!");
        }
    }
    ```
  </div>
</details>

### 사용법
```markdown
<details>
  <summary><strong>제목</strong></summary>

  <div class="mt-3">
    내용 (Markdown 사용 가능)
  </div>
</details>
```

---

## 🖼️ 5. 이미지 스타일링

### 기본 이미지
```markdown
![설명](/assets/img/image.png)
```

### 크기 지정
```markdown
![설명](/assets/img/image.png){: width="400" }
```

### 둥근 모서리 + 그림자
```markdown
![설명](/assets/img/image.png){: .rounded .shadow width="300" }
```

### 캡션 추가
```markdown
![설명](/assets/img/image.png)
_이미지 캡션_
```

---

## 🔘 6. 버튼

<div class="d-flex gap-2 mb-3">
  <a href="#" class="btn btn-primary">Primary</a>
  <a href="#" class="btn btn-secondary">Secondary</a>
  <a href="#" class="btn btn-success">Success</a>
  <a href="#" class="btn btn-outline-primary">Outline</a>
</div>

### 사용법
```html
<a href="#" class="btn btn-primary">버튼 텍스트</a>
<a href="#" class="btn btn-outline-primary">외곽선 버튼</a>
```

---

## 📊 7. 테이블

### 기본 테이블
| 기간 | 역할 | 기술 스택 |
|------|------|-----------|
| 2023 | Backend | Spring Boot, JPA |
| 2024 | DevOps | Kubernetes, AKS |
| 2025 | Cloud | Azure, AZ-204 |

### 사용법
```markdown
| 헤더1 | 헤더2 | 헤더3 |
|-------|-------|-------|
| 내용1 | 내용2 | 내용3 |
```

---

## 💬 8. 인용구 (Blockquote)

### 기본 인용구
> 일반적인 인용문입니다.

### Prompt와 조합
> **중요:** Prompt 스타일을 적용하지 않은 기본 blockquote입니다.

---

## 📝 9. Liquid 템플릿

### 최근 포스트 목록
{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url | relative_url }}) - `{{ post.date | date: "%Y.%m.%d" }}`
{% endfor %}

### 사용법
```liquid
{% raw %}{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }})
{% endfor %}{% endraw %}
```

### 조건문
```liquid
{% raw %}{% if page.url contains 'about' %}
About 페이지입니다!
{% endif %}{% endraw %}
```

---

## 🎯 10. 실전 예제: 프로필 섹션

<div class="d-flex align-items-center mb-4 p-3 border rounded">
  <img src="/assets/img/favicons/unggu.jpg" class="rounded-circle me-3" width="80" alt="Profile">
  <div>
    <h4 class="mb-1">김규형 (Unggu)</h4>
    <p class="text-muted mb-0">Backend Developer & DevOps Engineer</p>
  </div>
</div>

### 사용법
```html
<div class="d-flex align-items-center mb-4 p-3 border rounded">
  <img src="/path/to/image.jpg" class="rounded-circle me-3" width="80">
  <div>
    <h4 class="mb-1">이름</h4>
    <p class="text-muted mb-0">직함</p>
  </div>
</div>
```

---

## 📱 11. 연락처 버튼

<div class="d-flex gap-2 flex-wrap mb-3">
  <a href="mailto:example@example.com" class="btn btn-outline-primary">
    <i class="fas fa-envelope"></i> Email
  </a>
  <a href="https://github.com" class="btn btn-outline-dark">
    <i class="fab fa-github"></i> GitHub
  </a>
  <a href="https://linkedin.com" class="btn btn-outline-info">
    <i class="fab fa-linkedin"></i> LinkedIn
  </a>
</div>

### 사용법
```html
<a href="mailto:email@example.com" class="btn btn-outline-primary">
  <i class="fas fa-envelope"></i> Email
</a>
```

---

## ⚡ 12. 유용한 Bootstrap 클래스

### 간격 (Spacing)
- `mb-3` : margin-bottom (1~5)
- `mt-4` : margin-top
- `p-3` : padding (전체)
- `me-2` : margin-end (오른쪽)

### 정렬
- `d-flex` : Flexbox
- `align-items-center` : 수직 중앙 정렬
- `justify-content-between` : 양쪽 정렬
- `text-center` : 텍스트 중앙 정렬

### 색상
- `text-muted` : 회색 텍스트
- `text-primary` : 파란색
- `bg-light` : 밝은 배경

### 반응형 그리드
- `col-md-6` : 중간 화면 이상에서 50% 너비
- `col-lg-4` : 큰 화면에서 33% 너비
- `row g-3` : 그리드 간격

---

## 🚨 주의사항

> **Kramdown 속성 문법 (`{: }`)은 반드시 요소 바로 다음 줄에!**
{: .prompt-warning }

```markdown
✅ 올바른 예시:
> 내용
{: .prompt-tip }

❌ 잘못된 예시:
> 내용

{: .prompt-tip }  ← 빈 줄이 있으면 안 됨!
```

> **HTML 블록 안에서 Markdown 사용 시 빈 줄 필요**
{: .prompt-info }

```html
<div>

Markdown **문법**이 작동함

</div>
```

---

## 📚 참고 자료

- [Jekyll 공식 문서](https://jekyllrb.com/docs/)
- [Chirpy 테마 문서](https://github.com/cotes2020/jekyll-theme-chirpy)
- [Bootstrap 5 문서](https://getbootstrap.com/docs/5.3/)
- [Kramdown 문법](https://kramdown.gettalong.org/syntax.html)
- [Font Awesome 아이콘](https://fontawesome.com/icons)

---

> 이 가이드를 참고하여 멋진 About 페이지를 만들어보세요!
{: .prompt-tip }
