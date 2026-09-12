# n8n-mcp 분석 및 수익화 전략 (한국어)

> 저장소 분석 + 활용 방향 + 수익화 전략을 정리한 문서입니다.
> 작성일: 2026-09-12

## 저장소 정보

| 항목 | 내용 |
|---|---|
| **내 포크** | https://github.com/bmshin94/n8n-mcp |
| **원본 저장소** | https://github.com/czlonkowski/n8n-mcp |
| **npm 패키지** | https://www.npmjs.com/package/n8n-mcp |
| **공식 대시보드** | https://dashboard.n8n-mcp.com |
| **스킬 저장소** | https://github.com/czlonkowski/n8n-skills |
| **라이선스** | MIT |
| **분석 시점 버전** | 2.84.2 (n8n 2.38.2 대응) |
| **스타 / 포크** | 22,875 / 3,639 (2026-09 기준) |

---

## 1. 이게 뭐하는 프로젝트인가

한 줄 요약: **AI에게 n8n 사용법을 통째로 가르쳐주는 MCP 서버**

- **n8n** — 블록을 연결해 업무를 자동화하는 워크플로우 도구
- **MCP (Model Context Protocol)** — AI에게 외부 도구를 연결해주는 표준 규격
- **n8n-mcp** — 이 둘을 잇는 다리

### 해결하는 문제

이 도구가 없으면 AI는 n8n 노드 이름과 파라미터를 추측해서 만들어내고, 결과적으로
동작하지 않는 워크플로우가 나온다. n8n-mcp는 실제 노드 정의를 DB로 들고 있어서
AI가 정확한 정보를 조회하고, 만들기 전에 검증까지 할 수 있게 해준다.

### 폴더 구조 핵심

| 위치 | 내용 |
|---|---|
| `data/nodes.db` | 66MB SQLite DB — 노드 2,755개(코어 832 + 커뮤니티 1,923) + 템플릿 2,352개 |
| `data/skills/` | AI 교육용 스킬팩 15종 (agents, expression-syntax, error-handling, code-python 등) |
| `src/mcp/` | MCP 서버 본체, 도구 정의(`tools.ts`, `tools-n8n-manager.ts`), 도구별 문서 |
| `src/services/` | 검증기, 자동수정기, diff 엔진, n8n API 클라이언트, 보안 감사 |
| `src/loaders`, `parsers`, `mappers` | n8n 패키지 → 파싱 → DB 저장 파이프라인 |
| `src/mcp-engine.ts` | 다른 서비스에 내장하기 위한 임베딩 API |
| `src/http-server.ts` | HTTP 모드 (외부 언어에서 호출 가능) |
| `tests/` | 테스트 6,524개 |

---

## 2. 제공하는 MCP 도구 (총 28개)

### 오프라인 그룹 — 설정 없이 바로 사용

| 도구 | 용도 |
|---|---|
| `search_nodes` | 노드 검색 |
| `get_node` | 노드 상세 조회 (minimal / standard / full 3단계) |
| `validate_node` | 노드 설정 검증 |
| `validate_workflow` | 워크플로우 전체 검증 |
| `search_templates` | 템플릿 검색 (키워드 / 메타데이터 / 작업별 / 노드별) |
| `get_template` | 템플릿 상세 조회 |
| `tools_documentation` | 도구 사용법 안내 |

### 관리 그룹 (`n8n_*`) — n8n API 설정 필요, 21개

워크플로우 CRUD, 부분 업데이트, 실행 이력, 테스트, 버전 관리, 자동 수정,
템플릿 배포, 자격증명 관리, 데이터테이블, 폴더, 인스턴스 보안 감사 등.

핵심 도구:
- `n8n_create_workflow` — 실제 n8n 인스턴스에 워크플로우 생성
- `n8n_update_partial_workflow` — 변경분만 전송해 토큰 80~90% 절약
- `n8n_autofix_workflow` — 13가지 유형의 오류 자동 수정
- `n8n_audit_instance` — 인스턴스 보안 점검
- `n8n_workflow_versions` — 버전 이력 및 롤백

---

## 3. 플러그인 / 스킬 / MCP — 정체는?

본질은 **MCP 서버**이지만 세 가지 형태를 모두 지원한다.

| 형태 | 근거 | 설명 |
|---|---|---|
| MCP 서버 | `src/mcp/server.ts` | 본체. 28개 도구 제공 |
| 플러그인(확장) | `manifest.json` | Claude Desktop용 MCPB 번들로 패키징 가능 |
| 스킬 | `data/skills/`, `src/mcp/skills/registry.ts` | `skill://n8n-mcp/...` URI로 마크다운 문서 제공 |

개념 정리:
- **MCP** = AI에게 *손*(실행 능력)을 달아주는 것
- **스킬** = AI에게 *지식*(읽을 문서)을 주는 것
- **플러그인** = 그것을 포장한 배포 패키지

n8n-mcp는 손과 지식을 동시에 제공하기 때문에 강력하다.

---

## 4. 설치 및 사용법

### 방법 1 — 클라우드 (설치 없음)
https://dashboard.n8n-mcp.com 가입 → API 키 발급. 무료 티어 하루 100회 호출.

### 방법 2 — npx (권장)
```bash
claude mcp add n8n-mcp \
  -e MCP_MODE=stdio \
  -e LOG_LEVEL=error \
  -e DISABLE_CONSOLE_OUTPUT=true \
  -- npx n8n-mcp
```

n8n 인스턴스까지 제어하려면:
```bash
  -e N8N_API_URL=https://your-n8n-instance.com \
  -e N8N_API_KEY=your-api-key \
```

### 방법 3 — Docker
이미지 약 280MB. 저장소의 `docker-compose.yml` 사용.

### 방법 4 — 소스 빌드 (개발용)
```bash
npm install
npm run build     # TypeScript 컴파일
npm run rebuild   # 노드 DB 재생성 (2~3분 소요)
npm start         # stdio 모드 실행
```

### 실제 사용
별도 명령어가 없다. 평소처럼 자연어로 요청하면 AI가 내부적으로 도구를 호출한다.

> "구글시트 읽어서 슬랙에 요약 보내는 워크플로우 만들어줘"

권장 흐름: 템플릿 검색 → 노드 조회 → 설정 → 검증 → 배포

---

## 5. API 토큰이 필요한가

| 토큰 | 필수 여부 | 용도 |
|---|---|---|
| (없음) | — | 노드 검색, 문서 조회, 검증, 템플릿 — **토큰 없이 전부 동작** |
| `N8N_API_KEY` | 선택 | 내 n8n 인스턴스에 직접 생성/수정 |
| `AUTH_TOKEN` | HTTP 모드만 | 서버를 외부에 노출할 때 쓰는 접근 토큰 |
| `N8N_MCP_ACCESS_TOKEN` | 선택 | n8n 인스턴스 레벨 MCP 기능(Agents 등) |

DB가 로컬에 포함돼 있어 **인터넷 없이도** 검색·검증이 동작한다.

주의: `AUTH_TOKEN`의 기본값(`your-secure-token-here`)을 그대로 사용하면 안 된다.

---

## 6. 왜 인기가 있는가

2025년 6월 생성 이후 1년 3개월 만에 스타 22,875개.

1. **타이밍** — MCP 등장 시점과 n8n 성장기가 겹침
2. **명확한 문제 해결** — AI가 노드를 지어내는 문제를 실제로 해결
3. **품질** — 테스트 6,524개, 271KB 분량 CHANGELOG, 거의 매일 업데이트
4. **낮은 진입장벽** — `npx` 한 줄
5. **즉시 체감되는 성과** — 자발적 입소문

---

## 7. 로컬 에이전트 구축에 활용하기

### A. 그대로 연결해서 사용
MCP로 연결하면 로컬 에이전트가 곧바로 n8n 전문가가 된다.

### B. 코드에 내장 — `src/mcp-engine.ts`
```typescript
const engine = new N8NMCPEngine({
  additionalTools: [...]   // 커스텀 도구 추가 가능
});
```
- 커스텀 도구 추가 지원
- 멀티테넌트 지원 (`ENABLE_MULTI_TENANT=true`)
- 인증·과금·요청 제한은 감싸는 서비스가 담당하도록 설계됨

### C. 참고 교재로 활용
직접 MCP 서버를 만들 때 좋은 레퍼런스다.
- 도구 설계 (`src/mcp/tools.ts`)
- 토큰 절약 설계 (detail 3단계, diff 기반 업데이트)
- 검증 로직 계층화 (`src/services/`)

---

## 8. React / PHP로 만들 수 있는가

### React
- MCP 서버 본체는 **불가**. 브라우저에서는 파일·DB·서버 프로세스를 다룰 수 없다.
- 단, UI는 가능하다. 저장소에도 이미 `ui-apps/` 폴더가 있다.
- 역할 분담: **React = 프론트엔드, Node.js = MCP 엔진**

### PHP
- MCP는 언어 중립적 프로토콜이므로 **구현 자체는 가능**하다.
- 그러나 진짜 어려운 부분은 언어가 아니다:

| 어려운 부분 | 이유 |
|---|---|
| 노드 2,755개 DB 구축 | n8n이 JS 패키지라 파싱에 Node.js 필요 |
| 검증 로직 | `src/services/`에만 수십 개 파일 |
| 템플릿 2,352개 수집 | 크롤링 + 가공 파이프라인 필요 |
| 최신 n8n 추적 | 지속적인 유지보수 부담 |

### 권장 구조 — 재구현하지 말고 얹기
```
[React 화면] → [PHP/자체 백엔드 API] → [n8n-mcp HTTP 서버] → [n8n]
   직접 구현        직접 구현              그대로 사용
```
`npm run start:http`로 HTTP 모드를 띄우면 어떤 언어에서든 HTTP로 호출할 수 있다.

---

## 9. 수익화 전략

### 9.1 현실 인식 — 경쟁 환경

저장소 내 `docs/competitive-analysis-july-2026.md`에 따르면, n8n이 **공식 MCP 서버를
제품에 내장**했다. 따라서 "n8n-mcp를 그대로 호스팅해서 판매"하는 모델은 경쟁력이 없다.

### 9.2 살아남은 차별점 (= 판매 가능한 영역)

| 영역 | 우위 | 격차 |
|---|---|---|
| 검증 신뢰도 | n8n-mcp | 망가진 워크플로우 5개 중 공식은 5개 모두 통과시킴, n8n-mcp는 5개 모두 검출 |
| 템플릿 | n8n-mcp | 2,352개 vs 공식 0개 |
| 커뮤니티 노드 | n8n-mcp | 1,923개 지원 vs 공식은 검증 불가 |
| 큰 필드 수정 | n8n-mcp | Code 노드 한 줄 수정 시 1,174자 vs 220자 (5.3배) |
| 자격증명 / 감사 / 버전관리 | n8n-mcp | 공식에는 없음 |
| 멀티 인스턴스 (SaaS) | n8n-mcp | 공식은 1:1 고정 |

공식이 우위인 영역: 네이티브 통합, 드래프트/발행, 프로젝트·폴더 지정, pin-data 테스트.

### 9.3 수익화 5단계

#### 1단계 — 자동화 구축 대행
- 투자금 0원, 즉시 시작 가능
- 타겟: 이커머스 셀러, 병원·학원, 마케팅 대행사, 스타트업
- 단가: 건당 30~300만원
- **월 유지보수 계약을 반드시 함께 제안** (반복 수익 확보)
- 주의: 템플릿 사용 시 원작자 표기 의무 (README의 MANDATORY ATTRIBUTION)

#### 2단계 — 교육 콘텐츠
- 한국어 콘텐츠가 거의 없는 블루오션
- 유튜브 → 전자책 → 온라인 강의 → 유료 커뮤니티 순으로 확장
- 교육은 1단계 대행의 마케팅 채널 역할도 한다

#### 3단계 — 한국 특화 스킬팩 / 노드
- `data/skills/`의 15개 스킬팩은 전부 영어·글로벌 기준
- 카카오 알림톡, 네이버 스마트스토어, 국내 결제(토스/아임포트) 노드가 비어 있음
- 스킬팩은 마크다운 문서라 진입장벽이 낮다 (`src/mcp/skills/registry.ts`가 자동 로드)
- 직접 판매보다 포지셔닝·인지도 확보 수단으로 활용

#### 4단계 — 니치 특화 SaaS
- 기술적 기반이 이미 준비돼 있다: `src/types/instance-context.ts`의 `InstanceContext`가
  사용자별 `n8nApiUrl` / `n8nApiKey` / `instanceId`를 받고, `ENABLE_MULTI_TENANT=true`로
  멀티테넌트 모드가 켜진다
- 범용이 아닌 업종 특화로 접근 (이커머스, 병원, 부동산, 마케팅 대행사)
- 월 구독 3~30만원
- 리스크: 개발 기간 3~6개월. **고객을 먼저 확보한 뒤 시작할 것**

#### 5단계 — 관리형 운영(MSP)
공식 MCP가 제공하지 않는 운영 기능을 상품화한다.

| 기능 | 상품화 |
|---|---|
| `n8n_audit_instance` | 월간 보안 점검 리포트 |
| `n8n_workflow_versions` | 버전 관리 및 롤백 보장 |
| `n8n_executions` | 장애 모니터링 및 알림 |
| `n8n_autofix_workflow` | 자동 복구 |
| 멀티 인스턴스 | 다지점·다법인 통합 관리 |

패키지 예시: 베이직 월 30만 / 스탠다드 월 60만 / 프리미엄 월 100만원.
반복 수익 구조이고 이탈률이 낮다.

### 9.4 90일 실행 플랜

**1~30일 — 실력과 증거 확보**
- 본인 자동화 5개 직접 제작
- 과정을 블로그·영상으로 기록 (포트폴리오)
- 한국어 스킬팩 1개 시범 제작

**31~60일 — 첫 매출**
- 지인·커뮤니티 대상 반값 구축 3건
- 각 건에 유지보수 월 계약 부착
- 고객 불편 지점 기록 (4단계 SaaS의 근거 데이터)

**61~90일 — 확장**
- 전자책 또는 영상 시리즈 출시
- 정가 수주 전환
- MSP 패키지를 기존 고객에게 업셀

### 9.5 법적 체크리스트

| 항목 | 상태 | 조치 |
|---|---|---|
| 라이선스 | MIT | 상업적 이용 자유 |
| 저작권 표시 | 의무 | 재배포 시 `LICENSE` 유지 |
| 템플릿 출처 | 의무 | 납품물에 원작자 표기 |
| n8n 본체 | 확인 필요 | n8n은 Sustainable Use License — 재판매형 SaaS는 별도 검토 |
| 고객 데이터 | 중요 | API 키 보관 시 보안·개인정보 책임 발생 |
| 텔레메트리 | 참고 | 익명 사용 통계 수집(opt-in) — 기업 고객에 사전 고지 |

### 9.6 피해야 할 접근

- 그대로 복사해 호스팅 판매 (공식 무료 + 본가 무료 티어와 경쟁 불가)
- "n8n-mcp 대안" 포지션 (본가가 거의 매일 업데이트)
- 고객 없이 SaaS부터 개발
- 가격 경쟁
- 고객의 운영 중인 워크플로우를 AI로 직접 수정 (배상 책임 위험)

### 9.7 결론

**1단계(구축 대행) + 5단계(관리형 운영)** 조합을 권장한다.

- 투자금 없이 즉시 매출 발생
- 공식 MCP가 다루지 않는 운영·감사·버전관리 영역
- 반복 수익이 누적되는 구조
- 진행하면서 3·4단계의 재료가 자연스럽게 축적됨

핵심 원칙: **도구를 팔지 말고, 도구로 만든 결과와 안정적인 운영을 판다.**

---

## 10. 안전 주의사항

README에 명시된 경고:

- 운영 중인 워크플로우를 AI로 직접 수정하지 말 것
- 작업 전 반드시 복사본 생성
- 개발 환경에서 먼저 테스트
- 중요 워크플로우는 백업 내보내기
- 배포 전 반드시 검증

---

## 참고 문서

- 자체 호스팅: `docs/SELF_HOSTING.md`
- Claude Code 연동: `docs/CLAUDE_CODE_SETUP.md`
- n8n 배포: `docs/N8N_DEPLOYMENT.md`
- 공식 MCP 연동: `docs/OFFICIAL_MCP_SETUP.md`
- 경쟁 분석: `docs/competitive-analysis-july-2026.md`
- 보안 강화: `docs/SECURITY_HARDENING.md`
- 워크플로우 diff 예제: `docs/workflow-diff-examples.md`
