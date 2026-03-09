# 컴강의 플랫폼 설계안
> 모바일 퍼스트 · 서버비 0원 · 1억명 동시접속 · 보안 철저

---

## 1. 핵심 목표

| 목표 | 방법 |
|------|------|
| 서버비 0원 | 정적 사이트(Cloudflare Pages 무료) |
| 1억명 동시접속 | CDN 엣지 캐싱 — 서버 없음, 요청마다 파일 전달 |
| 빠른 속도 | 전 세계 300+ CDN 노드, PWA 캐시 |
| 보안 철저 | HTTPS 강제, CSP 헤더, 백엔드 없어 공격면 최소 |
| 유지보수 쉽게 | 파일 하나(index.html) + 강의 JSON — 비개발자도 수정 가능 |
| 모바일 퍼스트 | 375px 기준 설계, 터치 친화적 UI |

---

## 2. 아키텍처

```
[사용자 스마트폰]
      │ HTTPS
      ▼
[Cloudflare CDN] ── 전 세계 엣지 캐싱
      │ 캐시 히트 (99%)
      ▼
[정적 파일 (HTML/CSS/JS/JSON)]
      │
      ├── 영상: YouTube 임베드 (구글 CDN, 무료)
      ├── 검색: 클라이언트 JS (서버 불필요)
      └── 진도 저장: localStorage (서버 불필요)
```

**서버가 0개** → 서버비 0원, 해킹 대상 없음

---

## 3. 호스팅 스택

| 역할 | 서비스 | 비용 |
|------|--------|------|
| 정적 호스팅 | Cloudflare Pages | **무료** |
| CDN | Cloudflare (300+ 노드) | **무료** |
| 영상 | YouTube | **무료** |
| 도메인 | Cloudflare DNS | **무료** |
| HTTPS/TLS | Cloudflare 자동 | **무료** |
| **합계** | | **월 0원** |

---

## 4. 파일 구조

```
/
├── index.html          ← 전체 앱 (단일 파일)
├── sw.js               ← 서비스워커 (오프라인/캐시)
├── manifest.json       ← PWA 설정
└── data/
    └── courses.json    ← 강의 목록 (여기만 수정하면 강의 추가)
```

### 강의 추가 방법 (비개발자도 가능)
`data/courses.json` 파일만 편집:
```json
{
  "courses": [
    {
      "id": 1,
      "title": "파이썬 기초",
      "category": "프로그래밍",
      "level": "입문",
      "duration": "2시간",
      "youtubeId": "YOUTUBE_VIDEO_ID",
      "thumbnail": "https://img.youtube.com/vi/VIDEO_ID/mqdefault.jpg",
      "description": "파이썬을 처음 배우는 분들을 위한 강의"
    }
  ]
}
```

---

## 5. 모바일 퍼스트 UI 구조

```
┌─────────────────┐
│  🖥 컴강의       │  ← 헤더 (고정)
│  [검색창]       │
├─────────────────┤
│ [전체][프로그래밍]│  ← 카테고리 탭 (가로 스크롤)
│ [네트워크][보안] │
├─────────────────┤
│ ┌─────────────┐ │
│ │ 썸네일      │ │  ← 강의 카드
│ │ 파이썬 기초 │ │
│ │ ⭐ 입문 2h  │ │
│ └─────────────┘ │
│ ┌─────────────┐ │
│ │ ...         │ │
│ └─────────────┘ │
├─────────────────┤
│ 홈  검색  내강의 │  ← 하단 네비 (고정)
└─────────────────┘
```

---

## 6. 보안 설계

```
Content-Security-Policy:
  default-src 'self';
  frame-src https://www.youtube.com;
  img-src 'self' https://img.youtube.com data:;
  script-src 'self' 'unsafe-inline';
  style-src 'self' 'unsafe-inline';

X-Frame-Options: DENY
X-Content-Type-Options: nosniff
Referrer-Policy: no-referrer
Permissions-Policy: camera=(), microphone=(), geolocation=()
```

Cloudflare Pages의 `_headers` 파일로 자동 적용

---

## 7. 성능 목표

| 지표 | 목표 | 방법 |
|------|------|------|
| LCP | < 1.5s | CDN 캐시, 이미지 lazy load |
| FID | < 50ms | JS 최소화, 인라인 크리티컬 CSS |
| CLS | < 0.1 | 카드 크기 고정 |
| Lighthouse | 95+ | PWA + 정적 최적화 |

---

## 8. 오프라인 지원 (PWA)

- 서비스워커가 HTML/CSS/JS/JSON 캐시
- 인터넷 없어도 이전에 본 강의 목록 접근 가능
- 홈 화면에 앱처럼 설치 가능 (Android/iOS)

---

## 9. 진도 관리 (서버 없이)

```js
// localStorage로 진도 저장
localStorage.setItem(`progress_${courseId}`, JSON.stringify({
  watched: true,
  timestamp: Date.now()
}))
```

---

## 10. 배포 절차

1. GitHub에 이 저장소 push
2. Cloudflare Pages에서 GitHub 연결
3. 빌드 명령: 없음 (정적 파일)
4. 완료 → 전 세계 CDN 자동 배포

**이후 강의 추가:** `courses.json` 수정 후 git push → 자동 배포
