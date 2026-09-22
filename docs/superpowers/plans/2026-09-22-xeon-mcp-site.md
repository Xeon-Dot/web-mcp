# Xeon MCP 소개 사이트 Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** opencode.ai 무드의 미니멀 단일 HTML 소개 사이트로 Xeon MCP(무료 HTTP MCP 플랫폼)를 소개하는 `index.html`을 새로 작성한다.

**Architecture:** 정적 단일 `index.html` (인라인 CSS + JS). 섹션은 Nav → Hero → 특징 → 서버 목록 → Footer 순서, 얇은 보더로 구분. 서버 데이터는 HTML에 하드코딩. JS는 복사 버튼·스크롤 nav·모바일 메뉴만 담당.

**Tech Stack:** 순수 HTML/CSS/JS, Google Fonts (IBM Plex Sans + IBM Plex Mono). 빌드 도구 없음.

**Spec:** `docs/superpowers/specs/2026-09-22-xeon-mcp-site-design.md`

## Global Constraints

- 산출물은 `index.html` 단일 파일 (인라인 CSS/JS). 외부 의존성은 Google Fonts 링크 1개만.
- 포지셔닝 문구: "여러 HTTP MCP를 무료로 사용하는 플랫폼" — "게이트웨이" 문구 금지.
- SearXNG URL은 정확히 `https://mcp.xeon.kr/searxng`.
- 폰트: IBM Plex Sans (본문), IBM Plex Mono (URL·라벨).
- 컨테이너 `max-width: 67.5rem`, 섹션 `border-top: 1px` 보더 구분, 본문 행간 180~200%.
- 버튼 라운드 4px. 라이트/다크는 `prefers-color-scheme` 자동.
- 섹션 구성: Hero / 특징 / 서버 목록 / Footer 4개 + Nav. 연결 가이드 섹션 금지.
- 서버 목록: SearXNG 실표시 + Coming Soon 플레이스홀더. JS 레지스트리 금지 (YAGNI).
- 자동 테스트 없음. 각 태스크는 로컬 서빙 수동 검증으로 통과 판정.

## Review Focus

스펙이 명시하지 않았지만 실제 사용에서 걸릴 수 있는 항목:

1. **클립보드 API 실패 (비보안 컨텍스트, file://로 열었을 때)** — 복사 버튼이 죽지 않고 텍스트 선택 fallback이 뜨기를 기대. → Task 5 Step에서 file:// 직접 열기 검증 포함.
2. **라이트/다크 자동 전환** — OS 설정에 따라 배경/텍스트가 명확히 바뀌어야 함. 과도하게 대비 약하면 읽기 불가. → Task 6 검증 단계에서 두 모드 모두 확인.
3. **모바일 레이아웃 깨짐 (375px)** — 히어로 URL 블록과 서버 카드가 가로 스크롤 없이 접히기를 기대. → Task 6에서 375px 폭 확인.
4. **모바일 메뉴가 열린 채로 스크롤/닫힘 상태 불일치** — 햄버거 토글이 aria-expanded와 시각 상태를 같이 유지하기를 기대. → Task 5에서 토글 검증.
5. **앵커 내비게이션이 sticky Nav에 가림** — CTA 클릭 시 섹션 제목이 Nav 뒤로 숨지 않기를 기대. → Task 5에서 앵커 클릭 후 위치 확인.

---

### Task 1: 골격 + Nav + Hero

**Files:**
- Create: `index.html`

**Interfaces:**
- Consumes: 없음 (첫 태스크)
- Produces: `<nav>`, `<section class="hero">` 마크업과 CSS 변수 체계 (`--color-*`, `--font-*`). 이후 태스크는 같은 CSS 변수를 그대로 사용한다.

- [ ] **Step 1: HTML 골격과 기본 스타일 작성**

`index.html` 전체 내용을 다음과 같이 작성한다:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>Xeon MCP — 무료 HTTP MCP 플랫폼</title>
  <meta name="description" content="Xeon MCP는 여러 HTTP MCP 서버를 무료로 사용할 수 있는 플랫폼입니다. 설치 없이 URL만으로 AI 클라이언트에 연결하세요." />
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link href="https://fonts.googleapis.com/css2?family=IBMPlexMono:wght@400;500;600&family=IBMPlexSans:wght@400;500;600;700&display=swap" rel="stylesheet" />
<style>
/* ===== Design Tokens ===== */
:root {
  --font-sans: "IBM Plex Sans", -apple-system, BlinkMacSystemFont, "Segoe UI", sans-serif;
  --font-mono: "IBM Plex Mono", ui-monospace, "SF Mono", Menlo, monospace;
  --color-background: hsl(0, 20%, 99%);
  --color-background-weak: hsl(0, 8%, 97%);
  --color-background-weak-hover: hsl(0, 8%, 94%);
  --color-background-strong: hsl(0, 5%, 12%);
  --color-background-strong-hover: hsl(0, 5%, 18%);
  --color-background-interactive-weaker: hsl(64, 74%, 95%);
  --color-text: hsl(0, 1%, 39%);
  --color-text-weak: hsl(0, 1%, 60%);
  --color-text-strong: hsl(0, 5%, 12%);
  --color-text-inverted: hsl(0, 20%, 99%);
  --color-border: hsl(30, 2%, 81%);
  --color-border-weak: hsla(0, 100%, 3%, 0.12);
  --color-icon: hsl(0, 1%, 55%);
  --color-accent: hsl(62, 84%, 88%);
  --space-section: 4rem;
  --pad-x: 1.5rem;
}
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: hsl(0, 9%, 7%);
    --color-background-weak: hsl(0, 6%, 10%);
    --color-background-weak-hover: hsl(0, 6%, 15%);
    --color-background-strong: hsl(0, 15%, 94%);
    --color-background-strong-hover: hsl(0, 15%, 97%);
    --color-background-interactive-weaker: hsl(60, 20%, 8%);
    --color-text: hsl(0, 4%, 71%);
    --color-text-weak: hsl(0, 2%, 49%);
    --color-text-strong: hsl(0, 15%, 94%);
    --color-text-inverted: hsl(0, 9%, 7%);
    --color-border: hsl(0, 3%, 28%);
    --color-border-weak: hsl(0, 4%, 23%);
    --color-icon: hsl(10, 3%, 43%);
  }
}

/* ===== Reset & Base ===== */
*, *::before, *::after { margin: 0; padding: 0; box-sizing: border-box; }
html { scroll-behavior: smooth; }
body {
  font-family: var(--font-sans);
  background: var(--color-background);
  color: var(--color-text);
  line-height: 1.9;
  -webkit-font-smoothing: antialiased;
}
a { color: inherit; text-decoration: none; }
button { font-family: inherit; cursor: pointer; }
.container { max-width: 67.5rem; margin: 0 auto; padding: 0 var(--pad-x); }

/* ===== Nav ===== */
.nav {
  position: sticky; top: 0; z-index: 50;
  background: var(--color-background);
  border-bottom: 1px solid transparent;
  transition: border-color 0.2s;
}
.nav.scrolled { border-bottom-color: var(--color-border-weak); }
.nav-inner {
  display: flex; align-items: center; justify-content: space-between;
  height: 80px;
}
.logo {
  display: flex; align-items: center; gap: 10px;
  font-weight: 700; font-size: 1.1rem; color: var(--color-text-strong);
}
.logo-mark {
  width: 28px; height: 28px; border-radius: 6px;
  background: var(--color-background-strong);
  color: var(--color-text-inverted);
  display: flex; align-items: center; justify-content: center;
  font-family: var(--font-mono); font-weight: 600; font-size: 0.9rem;
}
.nav-links { display: flex; align-items: center; gap: 32px; list-style: none; }
.nav-links a { font-size: 0.9rem; color: var(--color-text-weak); }
.nav-links a:hover { color: var(--color-text-strong); text-decoration: underline; text-underline-offset: 4px; }
.btn {
  display: inline-flex; align-items: center; gap: 8px;
  background: var(--color-background-strong);
  color: var(--color-text-inverted);
  padding: 8px 16px; border: none; border-radius: 4px;
  font-size: 0.9rem; font-weight: 500;
  transition: background 0.15s;
}
.btn:hover { background: var(--color-background-strong-hover); }
.btn-ghost {
  background: transparent; color: var(--color-text-strong);
  border: 1px solid var(--color-border);
}
.btn-ghost:hover { background: var(--color-background-weak); }
.nav-toggle {
  display: none; background: none; border: none;
  width: 40px; height: 40px; color: var(--color-icon);
}
.nav-toggle svg { width: 22px; height: 22px; }

/* ===== Hero ===== */
.hero { padding: calc(var(--space-section) * 1.5) 0 var(--space-section); }
.hero-label {
  font-family: var(--font-mono); font-size: 0.8rem; font-weight: 500;
  color: var(--color-text-weak); letter-spacing: 0.05em;
  margin-bottom: 20px;
}
.hero h1 {
  font-size: clamp(2rem, 5vw, 2.6rem); font-weight: 700;
  color: var(--color-text-strong); line-height: 1.3;
  letter-spacing: -0.02em;
  max-width: 20ch;
}
.hero p {
  margin-top: 20px; max-width: 46rem;
  color: var(--color-text); line-height: 200%;
}
.hero-actions { display: flex; gap: 12px; margin-top: 32px; flex-wrap: wrap; }

/* URL 복사 블록 (Task 4에서 인터랙션 추가) */
.url-block {
  margin-top: 40px;
  display: flex; align-items: stretch;
  border: 1px solid var(--color-border); border-radius: 6px;
  background: var(--color-background-weak);
  overflow: hidden;
  max-width: 40rem;
}
.url-block code {
  font-family: var(--font-mono); font-size: 0.95rem;
  color: var(--color-text-strong);
  padding: 14px 18px; flex: 1;
  overflow-x: auto; white-space: nowrap;
}
.url-copy {
  border: none; border-left: 1px solid var(--color-border);
  background: transparent; color: var(--color-text-weak);
  padding: 0 18px; font-size: 0.85rem; font-weight: 500;
  display: flex; align-items: center; gap: 6px;
  transition: background 0.15s;
}
.url-copy:hover { background: var(--color-background-weak-hover); color: var(--color-text-strong); }
.url-copy[data-copied] { color: var(--color-text-strong); background: var(--color-accent); }

@media (max-width: 48rem) {
  .nav-links {
    display: none;
    position: absolute; top: 80px; left: 0; right: 0;
    flex-direction: column; align-items: stretch; gap: 0;
    background: var(--color-background);
    border-bottom: 1px solid var(--color-border-weak);
    padding: 8px 0;
  }
  .nav-links.open { display: flex; }
  .nav-links li a { display: block; padding: 14px var(--pad-x); }
  .nav-links .btn { margin: 8px var(--pad-x); justify-content: center; }
  .nav-toggle { display: flex; align-items: center; justify-content: center; }
}
</style>
</head>
<body>

<nav class="nav" id="nav">
  <div class="container nav-inner">
    <a class="logo" href="#">
      <span class="logo-mark">X</span>
      Xeon MCP
    </a>
    <button class="nav-toggle" id="nav-toggle" aria-label="메뉴 열기" aria-expanded="false" aria-controls="nav-links">
      <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2" stroke-linecap="round">
        <line x1="4" y1="7" x2="20" y2="7"></line>
        <line x1="4" y1="12" x2="20" y2="12"></line>
        <line x1="4" y1="17" x2="20" y2="17"></line>
      </svg>
    </button>
    <ul class="nav-links" id="nav-links">
      <li><a href="#features">특징</a></li>
      <li><a href="#servers">서버</a></li>
      <li><a class="btn" href="#servers">시작하기</a></li>
    </ul>
  </div>
</nav>

<header class="hero">
  <div class="container">
    <div class="hero-label">$ xeon-mcp</div>
    <h1>여러 HTTP MCP를<br />무료로 사용하는 플랫폼</h1>
    <p>
      Xeon MCP는 AI 에이전트와 MCP 클라이언트를 위한 무료 HTTP MCP 허브입니다.
      설치나 별도 설정 없이 URL만 복사해 원하는 클라이언트에 붙여넣으면
      바로 사용할 수 있습니다.
    </p>
    <div class="hero-actions">
      <a class="btn" href="#servers">서버 둘러보기</a>
      <a class="btn btn-ghost" href="#features">특징 보기</a>
    </div>
    <div class="url-block">
      <code id="hero-url">https://mcp.xeon.kr/searxng</code>
      <button class="url-copy" data-copy="#hero-url" type="button">복사</button>
    </div>
  </div>
</header>

<!-- Task 3: 특징 섹션 -->
<!-- Task 4: 서버 목록 섹션 + Footer -->

<script>
/* Task 5: 인터랙션 */
</script>
</body>
</html>
```

- [ ] **Step 2: 로컬 서빙으로 렌더링 확인**

Run: `python3 -m http.server 8080` (백그라운드)
Expected: 브라우저 `http://localhost:8080`에서 히어로까지 렌더링, 콘솔 에러 없음.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add nav and hero skeleton for Xeon MCP site"
```

---

### Task 2: 특징 섹션

**Files:**
- Modify: `index.html` (`<!-- Task 3: 특징 섹션 -->` 주석 위치에 마크업 추가, `<style>`에 규칙 추가)

**Interfaces:**
- Consumes: Task 1의 CSS 변수(`--color-*`, `--font-mono`), `.container`, `.hero-label` 패턴.
- Produces: `<section class="features" id="features">`. Task 3/5는 이 섹션의 존재(앵커 `#features`)를 기대한다.

- [ ] **Step 1: CSS 추가**

`<style>`의 닫힘 직전(미디어 쿼리 앞)에 추가:

```css
/* ===== Features ===== */
.section { padding: var(--space-section) 0; border-top: 1px solid var(--color-border-weak); }
.section-label {
  font-family: var(--font-mono); font-size: 0.8rem; font-weight: 500;
  color: var(--color-text-weak); letter-spacing: 0.05em;
  margin-bottom: 16px;
}
.section h2 {
  font-size: clamp(1.4rem, 3vw, 1.75rem); font-weight: 700;
  color: var(--color-text-strong); line-height: 1.4;
  max-width: 24ch;
}
.features-grid {
  display: grid; grid-template-columns: repeat(auto-fit, minmax(15rem, 1fr));
  gap: 1px; margin-top: 40px;
  background: var(--color-border-weak);
  border: 1px solid var(--color-border-weak);
  border-radius: 6px; overflow: hidden;
}
.feature-card { background: var(--color-background); padding: 24px; }
.feature-card .icon {
  font-family: var(--font-mono); font-size: 0.85rem; font-weight: 600;
  color: var(--color-text-weak); margin-bottom: 14px;
}
.feature-card h3 {
  font-size: 1rem; font-weight: 600;
  color: var(--color-text-strong); margin-bottom: 8px;
}
.feature-card p { font-size: 0.9rem; line-height: 180%; color: var(--color-text); }
```

- [ ] **Step 2: 마크업 추가**

`<!-- Task 3: 특징 섹션 -->` 주석을 다음으로 교체:

```html
<section class="section features" id="features">
  <div class="container">
    <div class="section-label">// features</div>
    <h2>바로 쓸 수 있는 무료 MCP 허브</h2>
    <div class="features-grid">
      <div class="feature-card">
        <div class="icon">01</div>
        <h3>완전 무료</h3>
        <p>모든 HTTP MCP 서버를 비용 없이 사용할 수 있습니다. 요금제나 가입 절차 없이 바로 연결하세요.</p>
      </div>
      <div class="feature-card">
        <div class="icon">02</div>
        <h3>설치 없이 URL만</h3>
        <p>HTTP 방식이라 별도 런타임이나 빌드가 필요하지 않습니다. URL을 복사해 클라이언트에 붙여넣으면 끝입니다.</p>
      </div>
      <div class="feature-card">
        <div class="icon">03</div>
        <h3>어느 클라이언트와든</h3>
        <p>Claude Desktop, Cursor, OpenCode 등 표준 MCP를 지원하는 모든 클라이언트에서 동일하게 동작합니다.</p>
      </div>
      <div class="feature-card">
        <div class="icon">04</div>
        <h3>계속 늘어나는 서버</h3>
        <p>새로운 MCP 서버가 지속적으로 추가됩니다. 지금은 SearXNG 웹 검색을 시작으로 준비 중인 서버가 많습니다.</p>
      </div>
    </div>
  </div>
</section>
```

- [ ] **Step 3: 브라우저 확인**

Run: `http://localhost:8080` 새로고침
Expected: 히어로 아래에 4카드 그리드가 보더 그리드로 표시, 폭에 따라 4→2→1 컬럼 접힘. 콘솔 에러 없음.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add features section"
```

---

### Task 3: 서버 목록 섹션

**Files:**
- Modify: `index.html` (`<!-- Task 4: 서버 목록 섹션 + Footer -->` 주석 위치, `<style>`)

**Interfaces:**
- Consumes: Task 2의 `.section`, `.section-label` 클래스.
- Produces: `<section class="section" id="servers">`와 SearXNG 카드의 `data-copy` 속성 (Task 5의 복사 JS가 `data-copy` 셀렉터로 이 요소들을 찾는다).

- [ ] **Step 1: CSS 추가**

`<style>` 닫힘 직전에 추가:

```css
/* ===== Servers ===== */
.server-list { display: flex; flex-direction: column; gap: 16px; margin-top: 40px; }
.server-card {
  border: 1px solid var(--color-border-weak); border-radius: 6px;
  padding: 24px;
  display: flex; align-items: center; justify-content: space-between;
  gap: 20px; flex-wrap: wrap;
}
.server-info { display: flex; align-items: center; gap: 16px; min-width: 0; }
.server-icon {
  width: 44px; height: 44px; border-radius: 6px; flex-shrink: 0;
  background: var(--color-background-weak);
  border: 1px solid var(--color-border-weak);
  display: flex; align-items: center; justify-content: center;
  font-family: var(--font-mono); font-weight: 600; color: var(--color-text-strong);
}
.server-meta { min-width: 0; }
.server-name {
  display: flex; align-items: center; gap: 10px;
  font-weight: 600; color: var(--color-text-strong);
}
.badge {
  font-family: var(--font-mono); font-size: 0.7rem; font-weight: 500;
  padding: 2px 8px; border-radius: 100px;
  border: 1px solid var(--color-border);
  color: var(--color-text-weak);
}
.badge-live {
  background: var(--color-accent); border-color: transparent;
  color: hsl(0, 5%, 12%);
}
.server-desc { font-size: 0.875rem; color: var(--color-text-weak); line-height: 170%; }
.server-url {
  font-family: var(--font-mono); font-size: 0.8rem;
  color: var(--color-text); margin-top: 4px;
  overflow-x: auto; white-space: nowrap;
}
.server-actions { display: flex; gap: 8px; flex-shrink: 0; }
.server-card.planned { opacity: 0.55; }
.server-card.planned .server-name { color: var(--color-text-weak); font-weight: 500; }
@media (max-width: 40rem) {
  .server-card { flex-direction: column; align-items: stretch; }
  .server-actions .btn { justify-content: center; }
}
```

- [ ] **Step 2: 마크업 추가**

`<!-- Task 4: 서버 목록 섹션 + Footer -->` 주석을 다음으로 교체:

```html
<section class="section" id="servers">
  <div class="container">
    <div class="section-label">// servers</div>
    <h2>지금 연결할 수 있는 MCP 서버</h2>
    <div class="server-list">
      <div class="server-card">
        <div class="server-info">
          <div class="server-icon">SX</div>
          <div class="server-meta">
            <div class="server-name">SearXNG <span class="badge badge-live">LIVE</span></div>
            <div class="server-desc">프라이버시 중심 메타 검색 엔진으로 웹 검색 결과를 제공합니다.</div>
            <div class="server-url" id="searxng-url">https://mcp.xeon.kr/searxng</div>
          </div>
        </div>
        <div class="server-actions">
          <button class="btn" data-copy="#searxng-url" type="button">URL 복사</button>
        </div>
      </div>
      <div class="server-card planned">
        <div class="server-info">
          <div class="server-icon">··</div>
          <div class="server-meta">
            <div class="server-name">다음 서버 <span class="badge">COMING SOON</span></div>
            <div class="server-desc">웹 검색에 이어 더 많은 HTTP MCP 서버가 준비 중입니다.</div>
          </div>
        </div>
      </div>
    </div>
  </div>
</section>

<footer class="section footer">
  <div class="container">
    <div class="footer-inner">
      <div>
        <a class="logo" href="#">
          <span class="logo-mark">X</span>
          Xeon MCP
        </a>
        <p class="footer-desc">여러 HTTP MCP를 무료로 사용하는 플랫폼.</p>
      </div>
      <ul class="footer-links">
        <li><a href="#features">특징</a></li>
        <li><a href="#servers">서버</a></li>
        <li><a href="https://modelcontextprotocol.io" target="_blank" rel="noopener">Model Context Protocol</a></li>
      </ul>
    </div>
    <div class="footer-legal">
      Built on the Model Context Protocol by Anthropic.
    </div>
  </div>
</footer>
```

Footer용 CSS도 `<style>`에 추가:

```css
/* ===== Footer ===== */
.footer { padding-bottom: calc(var(--space-section) * 1.2); }
.footer-inner {
  display: flex; justify-content: space-between; gap: 32px; flex-wrap: wrap;
  padding-top: 8px;
}
.footer-desc { font-size: 0.875rem; color: var(--color-text-weak); margin-top: 12px; }
.footer-links { list-style: none; display: flex; gap: 24px; flex-wrap: wrap; }
.footer-links a { font-size: 0.875rem; color: var(--color-text-weak); }
.footer-links a:hover { color: var(--color-text-strong); text-decoration: underline; text-underline-offset: 4px; }
.footer-legal {
  margin-top: 40px; padding-top: 24px;
  border-top: 1px solid var(--color-border-weak);
  font-family: var(--font-mono); font-size: 0.75rem; color: var(--color-text-weak);
}
```

- [ ] **Step 3: 브라우저 확인**

Run: `http://localhost:8080` 새로고침
Expected: SearXNG LIVE 카드 + 흐릿한 Coming Soon 카드 + Footer 표시. `#servers` 앵커 이동 시 제목이 sticky Nav에 가리지 않음 (가려지면 Task 5에서 `scroll-margin-top` 추가).

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add server list and footer sections"
```

---

### Task 4: 인터랙션 (복사 · Nav 스크롤 · 모바일 메뉴)

**Files:**
- Modify: `index.html` (`<script>` 블록, `<style>`)

**Interfaces:**
- Consumes: Task 1의 `#nav`, `#nav-toggle`, `#nav-links`, `#hero-url`; Task 3의 `[data-copy]` 버튼과 `#searxng-url`.
- Produces: 동작하는 복사 버튼(클립보드 API + file:// fallback), `.nav.scrolled` 토글, `#nav-links.open` 토글.

- [ ] **Step 1: 앵커 오프셋 CSS 추가**

`<style>` 닫힘 직전에 추가 (Review Focus #5):

```css
section[id], header[id] { scroll-margin-top: 96px; }
```

- [ ] **Step 2: 스크립트 구현**

`<script>` 블록 내용을 다음으로 교체:

```js
// URL 복사
function fallbackCopy(text) {
  const ta = document.createElement("textarea");
  ta.value = text;
  ta.style.position = "fixed";
  ta.style.opacity = "0";
  document.body.appendChild(ta);
  ta.select();
  let ok = false;
  try { ok = document.execCommand("copy"); } catch (e) { ok = false; }
  document.body.removeChild(ta);
  return ok;
}

async function copyText(text) {
  if (navigator.clipboard && window.isSecureContext) {
    try {
      await navigator.clipboard.writeText(text);
      return true;
    } catch (e) { /* fall through */ }
  }
  return fallbackCopy(text);
}

document.querySelectorAll("[data-copy]").forEach((btn) => {
  const original = btn.textContent;
  btn.addEventListener("click", async () => {
    const target = document.querySelector(btn.dataset.copy);
    if (!target) return;
    const ok = await copyText(target.textContent.trim());
    if (ok) {
      btn.textContent = "복사됨 ✓";
      btn.setAttribute("data-copied", "");
      setTimeout(() => {
        btn.textContent = original;
        btn.removeAttribute("data-copied");
      }, 2000);
    } else {
      // 클립보드 실패: 텍스트를 선택해 사용자가 수동 복사 가능
      const range = document.createRange();
      range.selectNodeContents(target);
      const sel = window.getSelection();
      sel.removeAllRanges();
      sel.addRange(range);
      btn.textContent = "텍스트 선택됨";
      setTimeout(() => { btn.textContent = original; }, 2000);
    }
  });
});

// Nav 스크롤 상태
const nav = document.getElementById("nav");
const onScroll = () => nav.classList.toggle("scrolled", window.scrollY > 8);
window.addEventListener("scroll", onScroll, { passive: true });
onScroll();

// 모바일 메뉴
const toggle = document.getElementById("nav-toggle");
const links = document.getElementById("nav-links");
toggle.addEventListener("click", () => {
  const open = links.classList.toggle("open");
  toggle.setAttribute("aria-expanded", String(open));
  toggle.setAttribute("aria-label", open ? "메뉴 닫기" : "메뉴 열기");
});
links.addEventListener("click", (e) => {
  if (e.target.closest("a")) {
    links.classList.remove("open");
    toggle.setAttribute("aria-expanded", "false");
  }
});
```

- [ ] **Step 3: 동작 검증 (Review Focus #1, #4, #5)**

1. `http://localhost:8080`에서 URL 복사 버튼 클립 → 클립보드에 `https://mcp.xeon.kr/searxng` 확인, 버튼이 "복사됨 ✓"로 2초 변했다 복귀.
2. `file://`로 `index.html` 직접 열기 → 복사 버튼 클릭 → fallback이 동작(복사됨 또는 "텍스트 선택됨")하고 페이지가 죽지 않음.
3. 스크롤 8px 넘기면 Nav 하단 보더 나타남, 올리면 제거.
4. 375px 폭에서 햄버거 클릭 → 메뉴 열림(aria-expanded=true), 링크 클릭 → 메뉴 닫힘 + 앵커 이동, 제목이 Nav에 가리지 않음.

- [ ] **Step 4: Commit**

```bash
git add index.html
git commit -m "feat: add copy, scroll, and mobile menu interactions"
```

---

### Task 5: 최종 QA + README 갱신

**Files:**
- Modify: `README.md`
- Verify: `index.html`

**Interfaces:**
- Consumes: Task 1~4의 완성된 페이지.
- Produces: 스펙의 검증 항목을 전부 통과한 상태와 갱신된 README.

- [ ] **Step 1: 전체 QA 체크 (Review Focus #2, #3)**

브라우저 개발자 도구에서 확인:
1. OS 라이트 모드 / 다크 모드 각각에서 배경·텍스트 대비가 읽을 만큼 충분한지 (폰트가 IBM Plex로 로드되는지).
2. 375px 폭에서 가로 스크롤 발생하지 않는지, 히어로 URL 블록이 줄바꿈/스크롤로 처리되는지.
3. `file://` 직접 열기에서도 폰트·레이아웃·인터랙션 모두 동작하는지.
4. 브라우저 콘솔에 에러·경고 없음.
5. `<title>`, `meta description`, `lang="ko"` 존재.

- [ ] **Step 2: README 갱신**

`README.md`를 다음 내용으로 교체:

```markdown
# web-mcp

**Xeon MCP** — 여러 HTTP MCP를 무료로 사용하는 플랫폼.

AI 에이전트와 MCP 클라이언트를 위한 무료 HTTP MCP 허브의 소개 페이지입니다.
현재 SearXNG 웹 검색 서버를 제공하며, 추가 서버가 준비 중입니다.

- SearXNG: https://mcp.xeon.kr/searxng

## 시작하기

```bash
open index.html
```

또는 로컬 서버로 실행:

```bash
python3 -m http.server 8080
```

## 기술 스택

- 단일 HTML 파일 (Vanilla JS + CSS)
- IBM Plex Sans / IBM Plex Mono
- `prefers-color-scheme` 기반 라이트/다크 자동 전환
- 완전 반응형 (데스크톱 · 태블릿 · 모바일)

---

Built on the [Model Context Protocol](https://modelcontextprotocol.io) by Anthropic.
```

- [ ] **Step 3: Commit**

```bash
git add README.md index.html
git commit -m "docs: update README for free HTTP MCP platform positioning"
```

---

### Task 6: 스펙 대조 최종 리뷰

**Files:**
- Verify: `docs/superpowers/specs/2026-09-22-xeon-mcp-site-design.md` vs `index.html`

- [ ] **Step 1: 스펙 체크리스트 대조**

스펙의 "섹션 구성", "디자인 시스템", "인터랙션", "오류 처리" 항목을 하나씩 `index.html`에서 찾아 확인:
- [ ] Nav(sticky) / Hero / 특징 4카드 / 서버 목록(SearXNG + Coming Soon) / Footer 5요소
- [ ] IBM Plex Sans/Mono, `max-width: 67.5rem`, 섹션 보더, 라이트/다크 자동
- [ ] URL 복사(클립보드 + fallback), Nav 스크롤, 모바일 메뉴, 스무스 앵커
- [ ] "게이트웨이" 문구 없음, SearXNG URL 정확, 연결 가이드 섹션 없음

- [ ] **Step 2: 커밋 (변경이 있을 경우만)**

```bash
git add -A
git commit -m "fix: align index.html with design spec"
```
