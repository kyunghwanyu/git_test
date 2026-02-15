# 미국주식 알림 웹앱 (MVP)

이 저장소는 **1인 전용 미국주식 모니터링/알림 웹앱**의 MVP 설계 및 구현 계획을 담고 있습니다.

## 핵심 목표

1. **관심 종목 모니터링 + 실시간 알림**
   - 관심 티커를 1분 주기로 모니터링
   - 사용자가 설정한 가격/지표/복합 조건 충족 시 즉시 텔레그램 알림 발송
2. **급등주 Top10 발굴 + 리포트 알림**
   - 매일 KST 기준 **오전 6:30, 오후 5:00** 배치 수행
   - 미국주식 전체 유니버스를 대상으로 점수화 후 Top10 전송
3. **1인 접근 제어 로그인**
   - 퍼블릭 배포 환경에서 ID/PW 1개 계정으로 접근 제어

---

## 기술 스택

| 영역 | 기술 |
|---|---|
| Backend | Python (데이터 수집/지표 계산/스케줄러/알림) |
| Frontend | Next.js 16, React 19, TypeScript, TailwindCSS 4 |
| 데이터/분석 | pandas, numpy, scipy, scikit-learn |
| 데이터 소스 | yahoofinance, webull |
| DB/Auth | Supabase, NextAuth v5, JWT |
| 메시징 | Telegram Bot API |

---

## 시스템 아키텍처 (MVP)

- **Frontend (Next.js)**
  - 로그인
  - 관심 종목 CRUD
  - 알림 조건 CRUD
  - 최근 알림 로그 / 일간 리포트 조회
- **Backend Worker (Python)**
  - 1분 주기 실시간 모니터링 루프
  - 지표 계산 + 조건 평가 엔진
  - 텔레그램 즉시 발송
- **Batch Scheduler (Python/Cron)**
  - KST 06:30, 17:00 급등주 Top10 산출
  - 리포트 저장 + 텔레그램 전송
- **Supabase**
  - watchlist, alert_rules, alert_events, daily_picks_reports 등 저장

---

## 기능 요구사항

## 1) 관심 종목 모니터링

- 모니터링 주기: **1분**
- 지원 조건(초기):
  - 가격 조건 (>, <, >=, <=)
  - 지표 조건 (RSI, MACD, MA, 거래량 등)
  - 복합 조건 (AND/OR)
- 동작:
  1. 관심 종목 조회
  2. 시세/지표 계산
  3. 룰 엔진 평가
  4. 중복/쿨다운 필터
  5. 텔레그램 알림 + DB 기록
- 알림 중복 방지:
  - 동일 조건 재발송 쿨다운 적용 (예: 5분)

## 2) 급등주 Top10 발굴

- 실행 시간: **KST 06:30, 17:00**
- 대상 유니버스: **미국주식 전체**
- 점수화 기준(초기):
  - 모멘텀, 거래량 급증, 추세/변동성 지표 조합
- 결과:
  - Top10 티커 + 핵심 지표 + 점수 요약
  - 텔레그램 리포트 발송
  - DB 저장 후 웹에서 열람 가능

## 3) 로그인

- 방식: **단일 ID/PW (1계정)**
- 구현:
  - NextAuth Credentials Provider
  - 비밀번호 해시 비교(bcrypt)
  - 미들웨어로 보호 라우트 접근 제한

---

## 데이터 모델 (초안)

- `watchlist`
  - id, ticker, enabled, created_at
- `alert_rules`
  - id, watchlist_id, rule_json, cooldown_sec, enabled
- `indicator_snapshots`
  - id, ticker, ts, price, rsi, macd, ma20, ma50, volume
- `alert_events`
  - id, rule_id, ticker, triggered_at, payload_json, sent_ok
- `daily_picks_reports`
  - id, run_at, picks_json, sent_ok
- `app_settings`
  - key, value_json

---

## 운영/보안 원칙

- 비밀번호 평문 저장 금지
- 모든 시크릿은 `.env`로 관리
- 텔레그램 발송 실패 시 재시도 및 실패 로그 기록
- 외부 데이터 소스 장애 대비 재시도/대체 소스 전략

---

## 개발 단계

1. **기초 인프라**: DB 스키마/환경변수/공통 로깅
2. **인증/UI 뼈대**: 로그인 + 대시보드 + CRUD 화면
3. **실시간 엔진**: 1분 모니터링 + 룰 엔진 + 텔레그램
4. **배치 엔진**: 2회/일 Top10 산출 + 리포트
5. **안정화**: 테스트/장애복구/운영 스크립트

---

## Quick Start (예정)

```bash
# Python deps
pip install -r requirements.txt

# Frontend deps
cd frontend && npm install

# env 설정
cp .env.example .env

# backend worker
python run.py

# frontend
cd frontend && npm run dev

# batch (single run)
python orchestrator.py once
```

> 현재 저장소는 초기 문서 중심 단계이며, 위 명령은 구현 진행에 맞춰 구체화됩니다.
