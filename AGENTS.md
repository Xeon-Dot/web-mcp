# AGENTS.md

Xeon MCP 소개 사이트 — `index.html` 단일 파일(인라인 CSS/JS) 정적 페이지. 빌드·패키지·자동 테스트 없음. UI 문구와 문서는 한국어.

## 실행과 검증

- 로컬 서빙: `python3 -m http.server 8080`
- 자동 테스트 없음. 검증 = Playwright 헤드리스 브라우저 실행 검사 + 정적 grep(구조/문구/URL).
- **root 환경 주의**: Chrome이 sandbox 거부로 기동 실패(`Running as root without --no-sandbox`). `playwright-cli`와 playwright MCP는 이 환경에서 동작 안 함. 해법: `playwright-core`로 `/opt/google/chrome/chrome`을 `args: ["--no-sandbox", "--disable-dev-shm-usage"]`로 직접 실행.

## 테마 시스템 (가장 쉽게 깨지는 부분)

- 이중 테마: OS 자동 전환(`prefers-color-scheme`) + 수동 토글(`html[data-theme="light|dark"`, `localStorage["theme"]`, head 인라인 스크립트가 렌더 전에 적용 — FOUC 방지).
- 테마 종속 스타일을 추가/수정할 때는 **두 패턴 모두** 선언해야 함. 하나만 쓰면 수동 토글 또는 자동 전환이 깨짐:
  - `@media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) … }`
  - `:root[data-theme="dark"] …`
- 노랑(`--color-accent`) 배경 위 텍스트는 `--color-on-accent`(양 모드 모두 어두움) 사용. `--color-text-strong`은 다크 모드에서 흰색이라 안 보임.

## 자산과 고정 문자열

- 로고/파비콘은 cdn.xeon.kr SVG: `Xeon-logo-blacktext-tran.svg`=라이트, `Xeon-logo-whitetext-tran.svg`=다크 (`.logo-light-text`/`.logo-dark-text`로 전환).
- Google Fonts 패밀리명은 `+` 인코딩 필수: `IBM+Plex+Mono`, `IBM+Plex+Sans`. `IBMPlexMono`처럼 쓰면 HTTP 400으로 폰트가 아예 안 로드됨.
- `https://mcp.xeon.kr/searxng`는 hero code 블록, 서버 카드, README 3곳에서 정확히 일치해야 함.
- 포지셔닝: "여러 HTTP MCP를 무료로 사용하는 플랫폼". **"게이트웨이" 문구 금지.**

## 레포 관례

- 설계 문서 `docs/superpowers/specs/2026-09-22-xeon-mcp-site-design.md`가 콘텐츠·디자인 결정의 근거. 변경 시 이 문서와 모순되지 않게.
- 커밋 메시지는 영문 conventional style (`feat:` / `fix:` / `docs:` / `chore:`).
- `.fva/`는 FVA 도구의 인덱스 바이너리이며 커밋 대상 — 도구가 갱신하면 `chore: update FVA index state`로 커밋.
