---
name: knowledge-agent
description: Notion 기반의 지능형 독서 기록 및 관리 스킬. gog CLI 스타일의 인터페이스를 지향하며 Brave Search와 Gemini를 활용한 자동화 워크플로우를 제공합니다.
metadata: {"clawdbot":{"emoji":"📚","requires":{"services":["brave-search", "gemini-api", "notion-api"]}}}
---

# 📚 Knowledge Agent: Book Management

이 스킬은 사용자가 자연어로 독서 기록을 관리하고, 외부 API를 통해 도서 정보를 수집하며, AI 분석을 통해 독후감 기록 및 도서 추천을 수행하는 기능을 제공합니다. 모든 데이터는 사용자의 Notion Database에 구조화되어 저장됩니다.

## 1. 도서 정보 자동 수집 (ISBN -> Notion)
사용자가 ISBN을 입력하면 도서 정보를 검색하여 Notion에 저장하거나 읽은 횟수를 업데이트합니다.

**워크플로우:**
- **Step 0 (생성)**: Notion에 db가 있는지 확인하고 존제하지 않을시  (`isbn`, `title`, `author`, `publisher`, `status` (unread/reading/completed), `read_count`, 'summary', 'reviews', 'review_date' 'rating', 'is_favorate' 초깃값 false, keyword(" "," "," "))의 저장구조로 DB 생성.
- **Step 1 (입력)**: 사용자가 Telegram을 통해 ISBN 입력, Notion isbn 속성에 저장.
- **Step 2 (검색)**: 저장되어있는 isbn을 사용해 Brave Search API로 제목, 저자, 출판사, 요약 및 키워드 검색.
- **Step 3 (정제)**: Gemini AI가 검색 결과중 제목, 저자, 출판사, 요약과 키워드만 찾아내 Notion 속성에 맞게 데이터 업데이트.
- **Step 4 (저장)**: Notion DB에 저장. 기존 ISBN 존재 시 `read_count`를 1 증가 및 저장 완료 메세지.



## 2. 독후감 기록 및 AI 분석
책 제목으로 도서를 특정하여 독후감을 작성하고 AI 감정 분석 결과를 함께 저장합니다.

**워크플로우:**
- **Step 0 (입력)**: 사용자가 telegram으로 책 재목 입력.
- **Step 1 (검색)**: 사용자가 입력한 제목으로 Notion DB 내 'Title' 항목 검색. 중복 시 ISBN으로 최종 선택 요청.
- **step 2 (입력)**: 사용자가 독후감, 별점을 입력한다.
- **Step 2 (기록)**: 별점, 작성 날짜를 포함하여 Notion의 `reviews`, 'rating', 'review date' 항목에 기록.



## 3. 관심 도서 및 즐겨찾기 관리
나중에 읽고 싶은 책을 태그로 관리합니다.




---

## 🛠️ Notes
- **에러 처리**: 검색 결과가 없거나 API 오류 발생 시 사용자에게 다시 입력을 제안합니다.