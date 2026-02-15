## 기술 스택

| 영역 | 기술 |
|---|---|
| Backend | supabase |
| Frontend | Next.js 16, React 19, TypeScript, TailwindCSS 4 |
| ML/분석 | scikit-learn (RF, GB, HGB, LR), pandas, numpy, scipy |
| 데이터 | yahoofinance, webull |
| 인증 | NextAuth v5, JWT |

---

## Quick Start

```bash
# 설치
pip install -r requirements.txt
cd frontend && npm install

# 환경 변수
cp .env.example .env
# abclab_API_KEY 등 설정

# 실행
python run.py          # 백엔드 (포트 5001)
cd frontend && npm run dev  # 프론트엔드 (포트 3000)

# 전체 데이터 업데이트
python orchestrator.py once
```
