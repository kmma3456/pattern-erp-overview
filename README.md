# 패턴제작업체 맞춤형 ERP

패턴 제작 업무(거래·출고·정산)를 한곳에서 처리하는 **맞춤형 업무 ERP**입니다.

> 소스코드는 private입니다. 이 문서는 채용·협업 검토용 공개 개요입니다.  
거래내역, 거래처, 매출 내보내기, 연차, 출퇴근, 전표, 메시지 발송까지 권한별로 나눕니다.

React + TypeScript + Express + Prisma + MySQL.

[기능 상세](docs/FEATURES.md) · [아키텍처](docs/ARCHITECTURE.md)

---

## 한눈에 보기

| | |
| --- | --- |
| **역할** | Full-stack (UI · API · DB · 배포) |
| **스택** | React 19, TypeScript, Vite, Express, Prisma, MySQL |
| **핵심** | 권한 기반 ERP, 엑셀 매출취합, 마일리지 정산, 연차/출퇴근 |
| **배포** | Nginx + pm2 |

```text
직원 / 실장 / 대표
        ↓
웹앱 (React)  →  REST API (Express)  →  MySQL (Prisma)
                      ├─ 엑셀 / ZIP 내보내기
                      ├─ SMS 발송
                      └─ GPS 출퇴근
```

---

## 왜 만들었나

엑셀과 메신저로 흩어져 있던 거래·출고·연차·근태를 **역할이 나뉜 하나의 ERP**로 모았습니다.

- 직원은 거래 입력, 필터, 본인 연차/출퇴근
- 실장은 매출취합, 마일리지 차감, 승인
- 대표는 사용자, 백업, 전표 결재까지

화면은 화려한 대시보드보다 **빨리 찾고, 덜 틀리게** 맞췄습니다.

---

## 주요 기능

### 매출 · 거래

- 거래내역 등록/수정, 리스트 인라인 수정
- 거래처 다중 선택 필터 (`전체(N개)` / `N개 선택`)
- 기간·담당자·단가/계산서/입금 등 다른 필터와 연동된 업체 수
- 매출취합 엑셀, 거래처별 ZIP
- 마일리지 적립/사용, 보내기 시 잔액 한도 내 차감

### 인사 · 근태

- 대표 / 실장 / 직원 권한 (사용자관리는 대표만)
- 연차·반차 신청/승인/반려, **주말·공휴일 제외 근무일 차감**
- 부여 대비 잔여 연차 (`잔여/부여`)
- GPS 지오펜스 출퇴근 (지점별 좌표)
- 복지내역서 · 지출결의서 + 전자서명

### 운영

- 거래처 마스터, 출력 의뢰 관리
- 출고/포장 완료 SMS (알리고 · 솔라피 · SENS · Webhook)
- 사내 게시판, 데이터 백업

화면 단위 설명은 [docs/FEATURES.md](docs/FEATURES.md)에 있습니다.

---

## 권한

| 구분 | 대표 | 실장 | 직원 |
| --- | :---: | :---: | :---: |
| 대시보드 / 거래 / 연차 / 출퇴근 | O | O | O |
| 매출취합 내보내기 · 마일리지 차감 | O | O | |
| 연차·전표 승인 | O | O | |
| 사용자관리 · 백업 | O | | |

직급(차장·과장·대리 등)은 표시용이고, 실제 제어는 **대표 / 실장 / 직원**입니다.

---

## 기술 스택

**Frontend**

![React](https://img.shields.io/badge/React_19-61DAFB?style=flat-square&logo=react&logoColor=black)
![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white)
![Vite](https://img.shields.io/badge/Vite-646CFF?style=flat-square&logo=vite&logoColor=white)
![React Router](https://img.shields.io/badge/React_Router-CA4245?style=flat-square&logo=reactrouter&logoColor=white)

**Backend & Data**

![Express](https://img.shields.io/badge/Express-000000?style=flat-square&logo=express&logoColor=white)
![Prisma](https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white)
![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white)

ExcelJS · xlsx-populate · JSZip · Playwright · Nginx · pm2

---

## 구조

```text
src/                 웹앱 (페이지 · 권한 · API 클라이언트)
  pages/             거래, 연차, 출퇴근, 출력, 전표, 메시지
  components/        공통 필터, 모달, 페이지네이션
  app/               라우팅, 인증, 도메인 타입/계산
server/              Express REST API + Prisma
prisma/              스키마 · 마이그레이션 · 시드
docs/                기능 / 아키텍처
```

데이터 흐름은 [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md)를 보면 됩니다.

---

## 구현에서 신경 쓴 점

1. **연차 일수**  
   토/일과 한국 공휴일은 차감하지 않습니다. 반차는 `0.5`일, DB는 `Decimal`입니다.

2. **매출 보내기 × 마일리지**  
   같은 기간에 이미 차감한 업체는 다시 깎지 않고, 공급가 한도 안에서만 사용 처리합니다.

3. **거래처 필터**  
   다른 조건을 걸면 셀렉트에 `전체(87개)`처럼 **지금 조건에 걸리는 업체 수**가 붙습니다.

4. **출퇴근**  
   소속 지점 좌표 반경 안에서만 체크인/아웃이 됩니다.

5. **내보내기**  
   거래처 미선택 시 업체별 파일을 ZIP으로, 한 곳만 고르면 엑셀 한 파일입니다.

---

## 로컬 실행

Node.js 20+, MySQL 8

```bash
cp .env.example .env
# DATABASE_URL 을 본인 MySQL에 맞게 수정

npm install
npx prisma migrate dev
npm run dev
```

- 웹: `http://localhost:5173`
- API: `http://localhost:8787`

로컬 테스트용 사용자는 Prisma 시드로 생성합니다. **로그인 정보는 공개 저장소에 포함하지 않습니다.**

운영 DB 접속 정보, 업로드 파일, SMS·메시지 API 키 등은 `.env`로만 관리합니다.

---

## 환경 변수

`.env.example`을 복사해 사용합니다.

- `DATABASE_URL` — MySQL
- `VITE_ATTENDANCE_*` — 출퇴근 지오펜스
- `MESSAGE_PROVIDER` — `MOCK` / `ALIGO` / `SOLAPI` / `NCP_SENS` / `WEBHOOK`

---

## 문서

| 문서 | 내용 |
| --- | --- |
| [docs/FEATURES.md](docs/FEATURES.md) | 화면별 기능, 권한, 업무 흐름 |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | 레이어, API, 데이터 모델, 배포 |
