# Knowledge Agent Workflows

## 1. 책 정보 자동 수집 (바코드 → ISBN)

### 개요
사용자가 책의 바코드를 촬영하면, ISBN을 추출하고 인터넷에서 책 정보를 자동으로 검색하여 Notion에 저장합니다.

### 워크플로우

```
Google Chat 채널
     ↓
[사용자 바코드 촬영]
     ↓
[바코드 이미지 전달]
     ↓
Note20 (openclaw)
     ↓
[이미지 분석 → ISBN 추출]
     ↑
Gemini API (Vision)
     ↓
[ISBN 확인 및 검증]
     ↓
[Brave Search API로 도서 정보 검색]
     ↑
Brave Search API
     ↓
[검색 결과 파싱]
- 제목, 저자, 출판사
- 출판일, ISBN
- 설명, 표지 이미지 URL 등
     ↓
[Gemini AI로 정보 요약 및 정제]
     ↑
Gemini API
     ↓
[Notion API를 통해 저장]
     ↑
Notion Database
     ↓
[완료 알림 전송]
     ↑
Google Chat 채널
```

### 세부 단계

| 단계 | 담당 | 입력 | 출력 | 설명 |
|------|------|------|------|------|
| 1 | Google Chat | 바코드 이미지 | 바코드 이미지 파일 | 사용자가 책 바코드를 촬영해서 전송 |
| 2 | Gemini Vision | 이미지 | ISBN 코드 추출 | Gemini의 Vision 모델로 바코드 인식 → ISBN 추출 |
| 3 | Brave Search | ISBN | 도서 검색 결과 | ISBN으로 도서 정보 검색 (Google Books, 예스24 등) |
| 4 | Gemini API | 검색 결과 | 정제된 메타데이터 | 검색 결과에서 중요 정보 추출 및 요약 |
| 5 | Notion API | 메타데이터 | Notion DB 저장 | 추출된 정보를 개인 Notion 라이브러리에 저장 |
| 6 | Google Chat | 완료 메시지 | 사용자 피드백 | 저장 완료 알림 및 링크 전송 |

### 저장 데이터 구조

```json
{
  "isbn": "xxx-xxxxxxxx",
  "title": "책 제목",
  "author": "저자명",
  "publisher": "출판사",
  "publication_date": "2024-01-15",
  "description": "책 설명...",
  "cover_image_url": "https://...",
  "pages": 320,
  "read_count": 1,
  "saved_at": "2026-02-14T10:30:00Z",
  "status": "unread",
  "rating": null,
  "review": null
}
```

### ISBN 중복 처리

ISBN이 이미 저장소에 있는 경우, 오류로 처리하지 않고 **책 읽은 횟수(read_count)를 1 증가**시킵니다.
예를 들어:
- 첫 저장: `read_count: 1`
- 같은 책 바코드 재촬영: `read_count: 2` (자동 업데이트)

### 에러 처리

- **바코드 인식 실패** → "바코드를 인식할 수 없습니다. 다시 촬영해주세요."
- **검색 결과 없음** → 수동 입력 모드 제안
- **API 오류** → 재시도 또는 수동 입력

---

## 2. 독후감 기록

### 개요
사용자가 책 제목으로 검색해서 독후감을 작성합니다. 책이 중복되는 경우에만 ISBN으로 확인합니다. 각 책마다 여러 번의 독후감을 별도로 저장할 수 있습니다.

### 워크플로우

```
Google Chat 채널
     ↓
[사용자 독후감 작성 요청]
(예: "책 제목: '어린왕자' 독후감 작성")
     ↓
Note20 (openclaw)
     ↓
[책 제목으로 Notion에서 검색]
     ↑
Notion Database
     ↓
[ 검색 결과 ]
     ├─ 결과 1건: 해당 책 ISBN 확정
     │   ↓
     │   [사용자 독후감 입력 수신]
     │
     └─ 결과 2건 이상: 중복 확인 필요
         ↓
         [사용자에게 ISBN 선택 요청]
         (예: "1. ISBN 978-xxx (저자1), 2. ISBN 978-yyy (저자2)")
         ↓
         [사용자 ISBN 선택]
         ↓
         [사용자 독후감 입력 수신]
     ↓
[Gemini AI로 독후감 분석]
- 감정 분석 (긍정/부정/중립)
- 주요 키워드 추출
     ↑
Gemini API
     ↓
[독후감 메타데이터 작성]
- 작성 날짜, 감정 분석 결과
- 별점, 추천 여부
     ↓
[Notion API로 저장]
     ↑
Notion Database
     ↓
[저장 완료 알림]
     ↑
Google Chat 채널
```

### 저장 데이터 구조

```json
{
  "isbn": "978-1234567890",
  "title": "책 제목",
  "content": "인터넷에서 검색한 리뷰 (네이버 책, 교보문고, Google Books 등)",
  "reviews": [
    {
      "id": "review_20260214_001",
      "content": "사용자가 작성한 독후감 본문...",
      "rating": 4.5,
      "emotion": "긍정적",
      "keywords": ["주제1", "주제2", "주제3"],
      "written_at": "2026-02-14T14:30:00Z",
      "reading_date": "2026-02-14",
      "recommend": true
    },
    {
      "id": "review_20260217_002",
      "content": "재독 후 작성한 독후감...",
      "rating": 5,
      "emotion": "긍정적",
      "keywords": ["주제1", "새로운주제"],
      "written_at": "2026-02-17T10:15:00Z",
      "reading_date": "2026-02-17",
      "recommend": true
    }
  ]
}
```

### 특징

- **책 제목 검색**: ISBN 입력 없이 책 제목만으로 검색 (편의성 향상)
- **중복 처리**: 같은 제목 책이 여러 개면 ISBN으로 선택
- **다중 독후감**: 같은 책에 여러 번 독후감 작성 가능 (재독시에도 기록)
- **별점 시스템**: 1~5점 또는 0.5 단위로 평가
- **감정 분석**: Gemini AI로 자동 분석 (긍정/중립/부정)
- **키워드 추출**: 책의 인상적인 부분 자동 추출

## 3. 책 추천

### 개요
사용자가 읽은 책의 키워드를 바탕으로 비슷한 주제의 책을 추천합니다. 
**방식 1:** 특정 책을 선택해 비슷한 책 추천
**방식 2:** 읽은 책들을 종합하여 취향에 맞는 책 추천

### 워크플로우 - 방식 1: 특정 책 기반 추천

```
Google Chat 채널
     ↓
[사용자 추천 요청]
(예: "'어린왕자' 같은 책 추천해줘")
     ↓
Note20 (openclaw)
     ↓
[책 제목으로 Notion에서 검색]
     ↑
Notion Database
     ↓
[대상 책의 keywords 추출]
(예: ["철학", "우화", "성장이야기"])
     ↓
[선택한 책과 비슷한 책 검색]
- 저장소 내 모든 책과 키워드 비교
- 유사도 점수 계산
     ↓
[Gemini AI로 유사도 분석]
- 키워드 유사도 점수 계산
- 상위 N개 추천 도서 선정
     ↑
Gemini API
     ↓
[추천 결과 시각화]
- 추천 도서 목록 + 유사도 점수
- 각 책의 평점, 상태 표시
     ↓
[Google Chat으로 결과 전송]
     ↑
Google Chat 채널
```

### 워크플로우 - 방식 2: 읽은 책들 기반 추천

```
Google Chat 채널
     ↓
[사용자 추천 요청]
(예: "지금까지 내가 읽은 책들 기반으로 추천해줄 책이 뭐가 있어?")
     ↓
Note20 (openclaw)
     ↓
[사용자의 모든 읽은 책 조회]
(status: "completed" 또는 "reading")
     ↑
Notion Database
     ↓
[전체 책의 키워드 종합]
- 모든 읽은 책의 keywords 수집
- 키워드 빈도 계산
- 사용자 취향 프로필 생성
     ↓
[Gemini AI로 취향 분석]
- 키워드 기반 사용자 프로필 구성
- 상위 관심사 도출
     ↑
Gemini API
     ↓
[저장소 내 책 비교]
- 읽지 않은 책들과 대조
- 유사도 점수 계산
     ↓
[상위 N개 추천 도서 선정]
- 유사도 기준 정렬
- 이미 읽은 책 제외
     ↓
[추천 이유 생성]
- "당신이 즐겨 읽는 철학 + 우화 장르입니다"
     ↓
[Google Chat으로 결과 전송]
     ↑
Google Chat 채널
```

### 저장 데이터 구조

```json
{
  "recommendation_request": {
    "book_title": "어린왕자",
    "book_isbn": "978-xxx",
    "search_keywords": ["철학", "우화", "성장이야기"],
    "requested_at": "2026-02-14T15:00:00Z"
  },
  "recommendation_results": [
    {
      "rank": 1,
      "isbn": "978-yyy",
      "title": "추천 책 제목",
      "author": "저자명",
      "similarity_score": 0.95,
      "common_keywords": ["철학", "우화"],
      "user_rating": 4.5,
      "status": "completed",
      "reason": "철학과 우화라는 공통 키워드가 있으며 성장 이야기 포함"
    },
    {
      "rank": 2,
      "isbn": "978-zzz",
      "title": "추천 책 제목 2",
      "author": "저자명 2",
      "similarity_score": 0.82,
      "common_keywords": ["성장이야기"],
      "user_rating": 3.8,
      "status": "unread",
      "reason": "성장이야기라는 공통 주제"
    }
  ]
}
```

### 특징

- **키워드 기반 추천**: 읽은 책의 키워드로 유사 책 검색
- **유사도 점수**: AI가 계산한 0~1 범위의 유사도 점수
- **이유 제시**: 왜 추천되었는지 설명 
- **사용자 평점 반영**: 추천 책의 사용자 평점 표시
- **다양한 순위**: 상위 5~10개 도서 추천

## 4. 책 검색

### 개요
사용자가 특정 주제나 관심사를 입력하면, Brave Search API로 해당 주제의 책을 검색하고 상위 3개 결과만 화면에 표시합니다.

### 워크플로우

```
Google Chat 채널
     ↓
[사용자 책 검색 요청]
(예: "판타지 소설 추천해줘" 또는 "SF 책 찾아줘")
     ↓
Note20 (openclaw)
     ↓
[주제 키워드 추출]
(예: "판타지 소설")
     ↓
[Brave Search API로 도서 검색]
Search Query: "판타지 소설 추천 책"
     ↑
Brave Search API
     ↓
[검색 결과 파싱]
- 도서 제목, 저자, ISBN
- 설명, 링크, 평점
     ↓
[상위 3개 결과만 필터링]
(유사도/인기도 기준)
     ↓
[Gemini AI로 결과 정제]
- 중복 제거
- 메타데이터 표준화
     ↑
Gemini API
     ↓
[검색 결과 포맷팅]
- 도서 정보 + 썸네일
- 구매/저장 옵션
     ↓
[Google Chat으로 결과 표시]
     ↑
Google Chat 채널
```

### 저장 데이터 구조

```json
{
  "search_query": "판타지 소설",
  "search_timestamp": "2026-02-14T15:30:00Z",
  "results": [
    {
      "rank": 1,
      "title": "책 제목",
      "author": "저자명",
      "isbn": "978-1234567890",
      "description": "책 설명...",
      "cover_image_url": "https://...",
      "rating": 4.7,
      "source": "교보문고",
      "purchase_link": "https://...",
      "action": "읽음/관심/저장"
    },
    {
      "rank": 2,
      "title": "책 제목 2",
      "author": "저자명 2",
      "isbn": "978-9876543210",
      "description": "책 설명...",
      "cover_image_url": "https://...",
      "rating": 4.5,
      "source": "네이버 책",
      "purchase_link": "https://...",
      "action": "읽음/관심/저장"
    },
    {
      "rank": 3,
      "title": "책 제목 3",
      "author": "저자명 3",
      "isbn": "978-1111111111",
      "description": "책 설명...",
      "cover_image_url": "https://...",
      "rating": 4.3,
      "source": "알라딘",
      "purchase_link": "https://...",
      "action": "읽음/관심/저장"
    }
  ]
}
```

### 특징

- **3개 제한**: 사용자 편의성을 위해 상위 3개 결과만 표시
- **다양한 출처**: 여러 도서 플랫폼에서 검색
- **빠른 응답**: 검색 결과를 즉시 화면에 표시
- **바로 저장 가능**: 검색 결과에서 바로 저장 또는 독서 기록
- **평점 포함**: 기존 사용자/플랫폼 평점 표시

## 5. 관심 도서 관리 (즐겨찾기)

### 개요
사용자가 관심 있는 책을 제목이나 태그로 검색하고 즐겨찾기에 등록합니다. 나중에 읽고 싶은 책들을 체계적으로 관리할 수 있습니다.

### 워크플로우

```
Google Chat 채널
     ↓
[사용자 검색 요청]
(예: "제목: '해리포터'" 또는 "태그: #판타지 즐겨찾기")
     ↓
Note20 (openclaw)
     ↓
[ 검색 방식 분기 ]
     ├─ 제목 검색: Notion 저장소에서 검색
     │   ↑
     │   Notion Database
     │   ↓
     └─ 태그 검색: keywords 필드에서 태그 매칭
         ↑
         Notion Database
     ↓
[검색 결과 표시]
- 도서 제목, 저자, 설명
- 이미 즐찾 여부 표시
     ↓
[사용자 즐찾 등록 요청]
     ↓
[Notion API로 즐겨찾기 데이터 저장]
     ↑
Notion Database
     ↓
[완료 알림 + 즐찾 목록 표시]
     ↑
Google Chat 채널
```

### 저장 데이터 구조

```json
{
  "isbn": "978-1234567890",
  "title": "책 제목",
  "author": "저자명",
  "is_favorite": true,
  "favorite_added_at": "2026-02-14T16:00:00Z",
  "favorite_tags": ["판타지", "모험", "청소년문학"],
  "priority": 1,
  "notes": "사용자가 남긴 메모 (선택사항)"
}
```

### 즐찾 목록 조회

```
Google Chat 채널
     ↓
[사용자 즐겨찾기 목록 요청]
(예: "내 즐겨찾기 목록 보여줘" 또는 "즐겨찾기: #판타지 필터")
     ↓
Note20 (openclaw)
     ↓
[사용자의 즐겨찾기 책 목록 조회]
(is_favorite: true 필터)
     ↑
Notion Database
     ↓
[ 필터링 (선택) ]
- 태그별 필터
- 우선순위별 정렬
     ↓
[목록 포맷팅 및 표시]
- 책 정보 (제목, 저자)
- 즐겨찾기 추가 날짜
- 우선순위
     ↓
[Google Chat으로 전송]
     ↑
Google Chat 채널
```

### 특징

- **제목 검색**: 정확한 책 제목으로 검색
- **태그 검색**: 관심사(판타지, 철학, 경제 등) 태그로 검색
- **즐겨찾기 우선순위**: 읽고 싶은 순서 설정 가능
- **메모 추가**: 각 책에 개인 메모 추가 가능
- **태그 기반 필터**: 즐겨찾기 목록을 태그로 필터링
- **중복 방지**: 이미 즐겨찾기한 책 표시


 