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
