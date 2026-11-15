# 가결 (GAGYEOL) — AI 기반 재무업무 자동화 서비스

> 영수증 한 장으로 양식 작성부터 결재까지 — 학생 단체 회계 처리를 자동화하다.

2026 단국대학교 소프트웨어학과 캡스톤 페스티벌 졸업작품 · 팀 **레로레로**

[![시연 영상](https://img.youtube.com/vi/85RA-XWsSbU/hqdefault.jpg)](https://youtu.be/85RA-XWsSbU)

▶ **시연 영상**: https://youtu.be/85RA-XWsSbU (3분 38초)

---

## 제출 자료

| 자료 | 위치 |
|------|------|
| 소스 코드 — 백엔드 (Spring Boot) | [`backend/`](https://github.com/dku-gagyeol/GAGYEOL_BE) → [dku-gagyeol/GAGYEOL_BE](https://github.com/dku-gagyeol/GAGYEOL_BE) |
| 소스 코드 — 프론트엔드 (Next.js) | [`frontend/`](https://github.com/dku-gagyeol/GAGYEOL_FE) → [dku-gagyeol/GAGYEOL_FE](https://github.com/dku-gagyeol/GAGYEOL_FE) |
| 결과보고서 PDF | [docs/가결_결과보고서_레로레로.pdf](docs/가결_결과보고서_레로레로.pdf) |
| 포스터 PDF / 이미지 (90×140cm) | [PDF](docs/가결_포스터_레로레로.pdf) · [JPG](docs/가결_포스터_레로레로.jpg) |
| 발표자료 PDF | [docs/가결_발표자료_레로레로.pdf](docs/가결_발표자료_레로레로.pdf) |
| 시연 영상 | https://youtu.be/85RA-XWsSbU |

> `backend/`, `frontend/`는 원본 저장소를 가리키는 Git 서브모듈입니다. GitHub 화면에서는 폴더를 누르면 원본 저장소로 이동합니다.

## 팀원

| 이름 | 학번 | 역할 |
|------|------|------|
| 고동민 | 32200129 | 백엔드 개발 — API 설계·구현, AI/RAG 연동, 결재 로직, 배포 |
| 김아름 | 32230857 | 프론트엔드 개발 — 화면 설계·구현, API 연동, 사용자 플로우 |

## 작품 개요

학생 단체(동아리·학생회)의 예산 집행 과정에서 반복되는 수작업 — 영수증 확인, 양식 작성, 회칙 확인, 결재 상신 — 을 AI로 자동화한 웹 서비스입니다.

1. **증빙 업로드 · AI 분석** — Upstage Information Extract로 영수증·거래명세서에서 결제수단·금액·날짜를 추출
2. **양식 추천 · 자동 작성** — 결제 유형에 맞는 양식지를 추천하고, Apache POI로 DOCX/XLSX 필드를 채우며 서술형 항목은 GPT-4o가 작성
3. **회칙 참조 (RAG)** — 회칙 PDF를 임베딩(text-embedding-3-small)해 pgvector에 저장하고, 관련 조항을 양식 설명(사용 조건·첨부 의무 등)에 반영
4. **필수 항목 누락 검사** — 자동으로 채워지지 않은 필수 항목과 증빙 연결 여부를 결재 상신 전에 확인
5. **다단계 결재** — 역할 순서(approval_order)에 따른 순차 결재, 비관적 락(PESSIMISTIC_WRITE)으로 동시 승인 충돌 방지, 반려 건 재결재
6. **수입지출관리대장 자동 작성** — 통장 거래내역(.xls)을 Apache POI로 직접 읽어 월별 시트에 나눠 기입하고 수입·지출 합계와 잔액을 반영

## 기술 스택

| 영역 | 기술 |
|------|------|
| Backend | Spring Boot 3.4.4, Java 17, JPA, Flyway |
| Frontend | Next.js 16, React 19, TypeScript |
| Database | PostgreSQL 16 + pgvector |
| AI | OpenAI GPT-4o, text-embedding-3-small, Upstage Information Extract |
| 문서 처리 | Apache POI, Apache PDFBox |
| 인증 · 배포 | JWT, Docker, Docker Compose |

## 시스템 구조

```
사용자(Browser)
   │
   ▼
Frontend (Next.js) ── REST API · Bearer JWT ──▶ Backend (Spring Boot)
                                                   ├ AI 서비스 (증빙 분석 · 양식 분석)
                                                   ├ RAG 엔진 (회칙 조항 검색)
                                                   └ 문서 처리 (POI · PDFBox)
                                                        │
                        ┌───────────────────────────────┼──────────────────────────┐
                        ▼                               ▼                          ▼
              PostgreSQL + pgvector               파일 스토리지         OpenAI · Upstage API
```

## 실행 방법

```bash
git clone --recurse-submodules https://github.com/kodongmin/GAGYEOL-capstone.git
cd GAGYEOL-capstone
```

**Backend** — 자세한 내용은 [`backend/README.md`](https://github.com/dku-gagyeol/GAGYEOL_BE)

```bash
cd backend
cp .env.example .env
# .env에 OPENAI_API_KEY, UPSTAGE_API_KEY 입력

# 방법 1) Docker Compose로 DB + 서버 함께 실행 (권장)
docker-compose up --build        # API: http://localhost:8080

# 방법 2) DB만 Docker로 띄우고 서버는 로컬에서 실행
docker-compose up db
./gradlew bootRun
```

그 밖의 환경 변수(`SPRING_DATASOURCE_URL`, `SPRING_DATASOURCE_USERNAME`, `SPRING_DATASOURCE_PASSWORD`, `JWT_SECRET`)는 backend README의 "환경 변수" 표를 참고하세요.

**Frontend** — 자세한 내용은 [`frontend/README.md`](https://github.com/dku-gagyeol/GAGYEOL_FE)

```bash
cd frontend
npm install
cp .env.example .env.local
# .env.local에 NEXT_PUBLIC_API_URL(백엔드 API 주소) 입력 — 로컬 백엔드라면 http://localhost:8080
npm run dev                      # http://localhost:3000
```

API 키는 저장소에 포함되어 있지 않습니다. 각자 발급한 키를 `.env`에 넣어 실행해 주세요.
