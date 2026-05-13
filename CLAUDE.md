# CLAUDE.md

이 파일은 Claude Code(claude.ai/code)가 이 저장소에서 작업할 때 참고하는 안내 문서입니다.

## 프로젝트 개요

영수증(이미지/PDF)을 업로드하면 **Upstage Vision LLM**이 자동으로 내용을 파싱·구조화하여 지출 데이터를 생성하는 경량 웹 애플리케이션. DB 없이 JSON 파일로 운영하는 1인 MVP.

## 기술 스택

| 레이어 | 기술 |
|--------|------|
| 프론트엔드 | React 18 + Vite 5 + TailwindCSS 3 + Axios |
| 백엔드 | Python FastAPI 0.111 + LangChain 0.2 |
| OCR | Upstage `document-digitization-vision` (`langchain-upstage` 경유) |
| 데이터 저장 | `backend/data/expenses.json` (JSON 파일, DB 없음) |
| 배포 | Vercel (프론트 정적 빌드 + 백엔드 서버리스 via Mangum) |

## 프로젝트 구조 (구축 예정)

```
receipt-tracker/
├── frontend/
│   └── src/
│       ├── pages/          # Dashboard.jsx, UploadPage.jsx, ExpenseDetail.jsx
│       ├── components/     # DropZone, ParsePreview, ExpenseCard, SummaryCard,
│       │                   # FilterBar, Badge, Modal, Toast
│       └── api/axios.js    # Axios 인스턴스 (baseURL: VITE_API_BASE_URL)
├── backend/
│   ├── main.py             # FastAPI 앱, CORS, 라우터 등록
│   ├── routers/            # upload.py, expenses.py, summary.py
│   ├── services/
│   │   ├── ocr_service.py      # LangChain + ChatUpstage + JsonOutputParser
│   │   └── storage_service.py  # expenses.json 읽기/쓰기
│   └── data/expenses.json
└── vercel.json
```

## 개발 명령어

### 백엔드
```bash
cd backend
python -m venv venv
venv\Scripts\activate          # Windows
pip install -r requirements.txt
uvicorn main:app --reload      # http://localhost:8000/docs
```

### 프론트엔드
```bash
cd frontend
npm install
npm run dev                    # http://localhost:5173
npm run build
```

## 환경변수

| 변수명 | 사용 위치 | 비고 |
|--------|----------|------|
| `UPSTAGE_API_KEY` | 백엔드 | Upstage API 인증 키 |
| `VITE_API_BASE_URL` | 프론트 빌드 시 | 미설정 시 `http://localhost:8000` 폴백 |
| `DATA_FILE_PATH` | 백엔드 | `VERCEL=1` 감지 시 자동으로 `/tmp/expenses.json` 사용 |

## API 엔드포인트

| 메서드 | 경로 | 설명 |
|--------|------|------|
| POST | `/api/upload` | 영수증 파일 업로드 → OCR 파싱 → JSON 반환 |
| GET | `/api/expenses` | 지출 목록 조회 (`from`, `to` 날짜 필터) |
| DELETE | `/api/expenses/{id}` | 지출 항목 삭제 |
| PUT | `/api/expenses/{id}` | 지출 항목 수정 |
| GET | `/api/summary` | 합계 통계 (`month` 필터) |

## 데이터 스키마

```json
{
  "id": "uuid-v4",
  "created_at": "ISO8601",
  "store_name": "string",
  "receipt_date": "YYYY-MM-DD",
  "receipt_time": "HH:MM | null",
  "category": "식료품|외식|교통|쇼핑|의료|기타",
  "items": [{ "name": "", "quantity": 0, "unit_price": 0, "total_price": 0 }],
  "subtotal": 0,
  "discount": 0,
  "tax": 0,
  "total_amount": 0,
  "payment_method": "string | null",
  "raw_image_path": "uploads/..."
}
```

## 핵심 아키텍처 결정사항

- **PDF 처리**: `pdf2image` + Poppler로 이미지 변환 후 Base64 인코딩하여 Vision LLM에 전달. Vercel 환경에서는 `/tmp` 경로 사용.
- **OCR 파싱**: LangChain `ChatUpstage` + `JsonOutputParser` 조합. 시스템 프롬프트에 JSON 스키마를 명시하여 구조화된 응답 강제.
- **Vercel 데이터 영속성**: 서버리스 파일시스템 비지속 문제로 `localStorage` 병행 저장 적용. 장기 운영 시 Railway/Supabase 전환 고려.
- **파일 검증**: MIME 타입 + 크기(10MB 제한)를 서버 측에서 재검증.

## UI 디자인 토큰

- 주요 색상: `indigo-600` / 페이지 배경: `gray-50` / 카드 배경: `white`
- Toast: `fixed bottom-4 right-4`, 3초 자동 소멸
- 카드: `rounded-xl border border-gray-200 shadow-sm hover:shadow-md`
- 폰트: Pretendard → Noto Sans KR 폴백

## 테스트 이미지

`images/` 폴더에 테스트용 영수증 이미지(이마트, 스타벅스, CU, 롯데백화점 등)와 PDF가 있음.

## PRD 문서 동시에 업데이트
* Source Code가 변경되거나 라이브러리 버전이 변경되면 반드시 @PRD_영수증_지출관리앱.md도 같이 업데이트 합니다.
* 기능 구현이 완료되면 @PRD_영수증_지출관리앱.md 문서에 있는 수락 기준 및 완료 기준의 Check Box에 완료된 사항들도 모두 체크표시 하세요.