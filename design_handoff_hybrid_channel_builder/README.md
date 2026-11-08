# Handoff: 하이브리드 웹 빌더 (오너/방문자 듀얼 모드 채널)

## Overview
채널(= 사용자가 만드는 소규모 웹사이트) 빌더 서비스입니다. 하나의 앱이 두 가지 계정 타입을 서비스합니다.

- **오너(Managing 계정)** — 채널을 만들고, 페이지를 구성하고, UI 컴포넌트를 배치·편집하고, 예약/게시판/상품을 운영하며 대시보드를 봅니다.
- **방문자(Profile 계정)** — 채널을 구독·열람하고, 콘텐츠를 아카이브하고, 예약을 요청·변경하고, 게시판에 글을 쓰고, 알림을 받습니다. 비공개 채널은 접근 요청이 필요합니다.

레퍼런스 목업 채널은 강남의 피트니스 채널 "스카이짐"이며, 예약 페르소나는 **퍼스널 트레이너 레슨 스케줄**입니다. 대표색은 메탈릭 실버 `#c9ced6`이고, 오너가 HEX로 바꿀 수 있습니다.

핵심 UI 규칙: **하나의 코드가 폭에 따라 두 개의 다른 UI 언어로 렌더링됩니다.** 데스크톱 폭에서는 웹 UI(3분할 2.5 : 5 : 2.5), 모바일 폭에서는 모바일 앱 UI(하단 탭바 + 바텀시트 + 전체화면 상세). 전환 임계값은 **900px**입니다.

## About the Design Files
이 번들의 HTML 파일은 **디자인 레퍼런스**입니다. 의도한 화면과 동작을 보여주는 프로토타입이며, 그대로 프로덕션에 복사해 쓰는 코드가 아닙니다.

해야 할 일은 이 HTML 디자인을 **대상 코드베이스의 기존 환경에서 다시 구현**하는 것입니다(React/Next.js, Vue, SwiftUI, 네이티브 등 — 그 코드베이스의 확립된 패턴과 라이브러리를 사용). 아직 환경이 없다면 프로젝트에 가장 적합한 프레임워크를 고른 뒤 그 위에 구현하십시오. 이 프로젝트는 Neon Postgres 연결이 목표이므로 아래 "Backend: Neon Postgres" 섹션에 스키마와 API 계약을 함께 실었습니다.

프로토타입의 모든 데이터는 목업이며 클라이언트 메모리에만 존재합니다. 이미지는 사용자가 드래그&드롭으로 채우는 플레이스홀더(`image-slot.js`)입니다.

## Fidelity
**High-fidelity (hifi)** — 색, 타이포, 간격, 인터랙션이 최종안입니다. 아래 명시된 값 그대로 픽셀 단위로 재현하되, 대상 코드베이스의 기존 컴포넌트 라이브러리로 조립하십시오. 값은 모두 Nocturne 디자인 시스템 토큰에서 나왔습니다(Design Tokens 섹션 참고).

---

## 전역 셸 (Global shell)

### 상단 시스템 바 (프로토타입 전용 + 프로덕션 겸용)
높이 최소 52px, 배경 `#1b1d2b`, 하단 보더 `1px solid #3f424d`, 패딩 `8.4px 16.8px`, 자식 간격 `16.8px`.

좌 → 우 순서:
1. 브랜드 마크 — 22×22px, radius 6px, 보더 `1px solid #968ae0`, 글자 "B" 11px `#b5abfc`; 옆에 "BUILDER" 12.5px, letter-spacing .04em, `#9397ab`.
2. **모드 스위처** — pill 세그먼트 컨트롤. 컨테이너: 패딩 2.8px, 보더 `1px solid #3f424d`, radius 999px, 배경 `#161826`. 옵션 "오너 모드" / "방문자 모드": 패딩 `5.6px 14px`, 12.5px, radius 999px. 선택됨 = 배경 `#2b2741` / 글자 `#e7e5fe`; 비선택 = 배경 transparent / 글자 `#9397ab`.
   - 프로덕션에서는 이 스위처가 계정 타입에 따라 결정됩니다. 프로토타입에서는 데모 목적의 수동 전환입니다.
3. (오너 모드일 때만) **편집 방식 토글** — "편집 방식 · A 바텀시트" / "· B 인스펙터". 보더 `1px solid #3f424d`, 글자 `#9397ab` 11.5px, 패딩 `5.6px 11.2px`, radius 999px. 두 가지 편집 인터랙션 안을 비교하기 위한 것으로, 프로덕션에서는 **하나를 골라 제거**하는 것이 맞습니다(권장: 모바일은 항상 A, 데스크톱은 B).
4. (오너 모드일 때만) **채널 편집 / 편집 종료** 버튼 — 보더 `1px solid #968ae0`, radius 999px, 패딩 `5.6px 14px`, 12.5px. 편집 중 = 배경 `#2b2741` 글자 `#e7e5fe`; 아님 = transparent / `#b5abfc`.
5. 우측 뷰포트 인디케이터 — 6px 점(`#968ae0`) + 11.5px `#75798c` 텍스트. 데스크톱: "데스크톱 웹 UI · 2.5 : 5 : 2.5", 모바일: "모바일 앱 UI · {width}px". **프로토타입 전용 — 프로덕션에서 제거**.

### 데스크톱 레이아웃 (창 폭 ≥ 900px)
시스템 바 아래 전체 높이를 CSS grid로 3열: `grid-template-columns: 2.5fr 5fr 2.5fr`. 세 열은 각각 독립 스크롤(`overflow-y: auto; min-height: 0`). 1269px 창에서 실측 317 / 635 / 317px.

- **좌측 (2.5)** — 정체성 패널. 배경 `#1b1d2b`, 우측 보더 `1px solid #3f424d`, 패딩 16.8px, 섹션 간격 16.8px. 오너면 대시보드, 방문자면 프로필.
- **중앙 (5)** — 채널 본체. 배경 `#161826`. 채널 헤더 + 페이지 내비 + 컴포넌트 스택. 하단 패딩 120px(플로팅 바 여유).
- **우측 (2.5)** — 상세/인스펙터. 배경 `#1b1d2b`, 좌측 보더 `1px solid #3f424d`. 세 가지 상태: 콘텐츠 상세 / 편집 인스펙터 / 빈 상태.

### 모바일 레이아웃 (창 폭 < 900px)
- grid가 `1fr` 단일 열로 붕괴.
- 중앙 열만 기본 표시. 좌측 정체성 패널은 하단 탭 "내 정보"(방문자) / "대시보드"(오너)를 눌렀을 때 중앙을 **대체**해 표시.
- 우측 열은 `position: absolute; inset: 0; z-index: 30` 전체화면 오버레이가 되며, 콘텐츠 상세를 열었을 때만 나타납니다.
- **하단 탭바** — `position: absolute; left/right/bottom: 0; z-index: 34`, 배경 `rgba(27,29,43,0.96)` + `backdrop-filter: blur(12px)`, 상단 보더 `1px solid #3f424d`, 패딩 `5.6px 8.4px 11.2px`. 탭 4개 균등 분할(`flex: 1 1 0`), 각 탭은 18×18px radius 5px 아이콘 자리(보더 1.5px currentColor) + 10.5px 라벨, 세로 스택 간격 4px, 패딩 `8.4px 0`. 선택 = 대표색 / 아이콘 opacity 1, 비선택 = `#75798c` / opacity .6. 탭 높이 포함 실질 안전영역 약 86px.
  - 오너 탭: 채널 · 예약 · 게시판 · 대시보드
  - 방문자 탭: 홈 · 예약 · 게시판 · 내 정보
- 히트 타깃은 44px 이상 유지.

---

## Screens / Views

### 1. 채널 뷰 (중앙 열) — 방문자
**목적**: 채널을 열람하고 콘텐츠를 탭해 상세를 봅니다.

**채널 헤더** — `position: sticky; top: 0; z-index: 12`, 배경 `rgba(22,24,38,0.92)` + blur(10px), 하단 보더 `1px solid #3f424d`.
- 1행 (패딩 `11.2px 22.4px`, 간격 11.2px): 채널 아이콘 30×30 radius 8 → 채널명 15px weight 500 letter-spacing -.01em **대표색**, 그 아래 서브도메인 10.5px `#75798c` (`skygym.mysite.co.kr`) → 우측 "구독하기 / 구독 중" 버튼(보더 1px 대표색, radius 999px, 패딩 `5.6px 14px`, 12px; 구독 중 = 배경 `#2b2741` 글자 `#e7e5fe`).
- 2행 페이지 내비 (패딩 `0 22.4px 11.2px`, 간격 5.6px, `overflow-x: auto`): 페이지 탭 버튼, 패딩 `5.6px 12px`, 12.5px. **메뉴바 스타일 3종에 따라 형태가 달라짐**:
  - `top` — 헤더가 `position: relative` (스크롤과 함께 올라감). 탭은 radius 999px, 선택 시 보더 1px 대표색 + 글자 대표색, 비선택 보더 transparent + `#9397ab`.
  - `top-pinned` (기본) — 헤더 `position: sticky`. 탭 형태는 `top`과 동일.
  - `button` — 탭이 radius 8px 박스형: 선택 = 배경 `#2b2741` + 보더 1px 대표색, 비선택 = 배경 transparent + 보더 `1px solid #3f424d`.

**컴포넌트 스택** — 세로 flex, 간격 33.6px, 패딩 `22.4px 22.4px 120px`. 각 컴포넌트는 `<section>`, radius 14px.

기본 페이지 구성(목업):
| 페이지 | role | 컴포넌트 |
| --- | --- | --- |
| 홈 | home | hero, grid, trainers, pricing, gallery, info |
| 예약 | — | calendar, list |
| 레슨 | main | grid, faq |
| 게시판 | — | board |
| 스토어 | — | products |

### 2. UI 컴포넌트 11종 (스펙)
모든 카드 공통: 배경 `#232532`, 보더 `1px solid #3f424d`, radius 8px. 섹션 제목 `h2` 21px weight 500 letter-spacing -.01em. 구분선은 `1px solid rgba(233,233,237,0.08)`.

1. **hero** — min-height 320px, radius 14px, overflow hidden, 내용은 하단 정렬. 배경 이미지 슬롯이 `position: absolute; inset: 0`을 채우고, 그 위에 그라디언트 `linear-gradient(to top, rgba(22,24,38,.94) 0%, rgba(22,24,38,.35) 55%, rgba(22,24,38,.1) 100%)` (pointer-events none). 텍스트 블록 패딩 `28px 28px 24px`, 간격 11.2px: kicker 11px letter-spacing .14em uppercase 대표색 → h1 34px weight 500 line-height 1.15 letter-spacing -.02em max-width 15em → 본문 14px `#b2b6ca` line-height 1.6 max-width 34em → 버튼 2개("레슨 예약하기" = 보더 1px 대표색/글자 대표색, "채널 소개" = 보더 `1px solid #3f424d`/글자 `#b2b6ca`; 둘 다 radius 8px 패딩 `9px 18px` 13px).
   - 목업 카피: kicker "스카이짐", h1 "몸을 바꾸는 가장 확실한 60일", 본문 "강남 도산대로, 4명의 트레이너와 12개 프로그램. 첫 상담은 무료입니다."
2. **grid** (블로그·인스타형 포스트) — 제목 행(제목 + 우측 메타 11.5px `#75798c` "4개의 글") + `repeat(auto-fill, minmax(220px, 1fr))` gap 16.8px. 카드: 이미지 높이 132px → 패딩 12px 간격 5.6px: kicker 10.5px uppercase letter-spacing .06em 대표색 / 제목 14px weight 500 line-height 1.35 / 발췌 12px `#9397ab` line-height 1.55 / 메타 10.5px `#595d6c`. 카드 전체 클릭 → 상세.
3. **list** — 행마다 좌측 kicker 12.5px 대표색(고정폭 74px) / 제목 13.5px + 발췌 11.5px `#75798c` / 우측 메타 11.5px `#9397ab`. 행 패딩 `12px 0`, 하단 보더 구분선.
4. **trainers** — `minmax(190px, 1fr)` gap 16.8px. 카드 패딩 16.8px 간격 11.2px: 원형 62px 사진 슬롯 → 이름 14.5px weight 500 / 전문 11.5px 대표색 / 소개 12px `#9397ab`.
5. **calendar (예약)** — 아래 "예약 플로우" 섹션 참고.
6. **products** — `minmax(230px, 1fr)`. 카드: 이미지 120px → 이름 13.5px / 가격 15px 대표색 / "재고 N" 11px `#75798c` / "장바구니" 버튼(보더 `1px solid #3f424d`, radius 6px, 패딩 `7px 11.2px`, 12px).
7. **pricing** — `minmax(200px, 1fr)`. 카드 패딩 16.8px: kicker 11px uppercase letter-spacing .08em `#75798c` / 플랜명 15px / 가격 25px weight 500 대표색 letter-spacing -.02em / 설명 12px `#9397ab` line-height 1.6.
8. **board** — 제목 행에 "글쓰기" 버튼(보더 1px 대표색, radius 8px, 패딩 `7px 14px`, 12.5px). 안내문 11.5px `#75798c`: "이 채널을 구독한 사람은 누구나 글을 쓸 수 있습니다". 글 카드 패딩 16.8px 간격 8.4px: 아바타 24px 원(`#3f424d`) + 작성자 12px `#b2b6ca` + 시간 11px `#595d6c` + 우측 태그 pill(보더 `1px solid #3f424d`, 10.5px, `#9397ab`, 패딩 `2px 7px`) → 제목 14.5px weight 500 → 발췌 12.5px `#9397ab` → 하단 액션 행 11.5px `#75798c` 간격 16.8px: "댓글 N", "보관 N", "링크 공유"(button, 보더 없음).
9. **gallery** — `minmax(150px, 1fr)` gap 8.4px, 각 항목 `aspect-ratio: 1/1`, radius 8px 이미지 슬롯.
10. **info (위치·영업시간)** — 카드 하나, 패딩 22.4px, `repeat(auto-fit, minmax(220px, 1fr))` gap 16.8px. 각 항목: kicker 11px uppercase letter-spacing .08em `#75798c` / 본문 13.5px line-height 1.6 `white-space: pre-line`.
11. **faq** — 행마다 질문 13.5px + 답 12.5px `#9397ab` line-height 1.6, 패딩 `12px 0`, 하단 구분선.

### 3. 좌측 패널 — 오너 대시보드
- 헤더: 42px 원형 아바타 + 이름 14px("스카이짐 · 정민호 관장") + 서브도메인 11.5px `#75798c`(ellipsis).
- **지표 그리드** 2열 gap 8.4px. 카드: 보더 `1px solid #3f424d`, radius 8px, 패딩 11.2px, 배경 `#232532`. 라벨 10.5px uppercase letter-spacing .05em `#75798c` / 값 19px weight 500 / 델타 10.5px(양수 강조는 `#b5abfc`, 중립은 `#75798c`).
  - 오늘 예약 11건 (어제 +2) · 이번주 예약 58건 (지난주 +7) · 방문자 412 (조회 1,908) · 구독자 236 (이번주 +9) · 게시판 4 · 12 (신규 글 · 댓글) · 이번달 매출 8,940,000 (결제 34건)
- **오늘 예약** 리스트: 시간 12.5px 대표색(고정폭 46px) / 이름 12.5px + 종류 11px `#75798c` / 상태 pill 10.5px. 행 패딩 `8.4px 0` + 하단 구분선.
- **접근 요청 대기**: 카드 안에 이름 12.5px + 사유 11px `#75798c` + "승인" 버튼(보더 1px `#968ae0`, 글자 `#b5abfc`, radius 6px, 패딩 `4px 9px`, 11px).
- **알림 센터**: 5px 점(신규 `#968ae0`, 읽음 `#595d6c`) + 12px `#b2b6ca` 본문, 뒤에 " · {경과}" `#595d6c`.

### 4. 좌측 패널 — 방문자 프로필
- 헤더: 42px 원형 아바타 + "한지민" 14px + "@jimin · 프로필 계정" 11.5px.
- **3열 통계** gap 8.4px, 중앙 정렬: 값 17px weight 500 / 라벨 10.5px `#75798c`. (구독 채널 7 · 보관 콘텐츠 24 · 예약 2)
- **내 예약** 카드: 제목 12.5px + 상태 pill(배경 `#2b2741`, 글자 `#d2cefd`, 10.5px) / 일시 11.5px `#9397ab` / "변경"·"취소" 버튼(보더 `1px solid #3f424d`, radius 6px, 패딩 `4px 9px`, 11px).
- **아카이브**: 카테고리 필터 pill 4개(전체 / 채널 / 콘텐츠 / 상품, 패딩 `4px 11.2px`, 11.5px) + 항목 행(34px radius 6px 썸네일 + 제목 12.5px + 메타 11px `#75798c`), 클릭 → 상세 또는 채널 이동.
- **알림 · 메시지**: 오너와 같은 형식.

### 5. 우측 패널 — 콘텐츠 상세
데스크톱은 우측 열 인라인, 모바일은 전체화면. 진입 애니메이션 `riseIn .18s ease` (opacity 0→1, translateY 8px→0).
- sticky 헤더: 좌측 종류 라벨 11px uppercase letter-spacing .08em `#75798c` + 우측 ✕ 버튼(16px `#9397ab`). 패딩 `11.2px 16.8px`, 하단 보더.
- 이미지 180px.
- 본문 영역 패딩 16.8px 간격 11.2px: kicker 10.5px uppercase 대표색 → h2 21px weight 500 line-height 1.25 → 메타 11.5px `#595d6c` → (상품이면) 가격 박스: 카드 스타일 패딩 11.2px, 가격 18px 대표색 + "재고 N" 11.5px `#75798c` → 본문 13.5px `#b2b6ca` line-height 1.75 `white-space: pre-line`.
- (게시글이면) **댓글 섹션**: 상단 구분선 + 패딩탑 16.8px. "댓글 N" 11px uppercase `#75798c`. 댓글: 22px 원형 아바타 + 작성자 12px `#b2b6ca` + " · {경과}" `#595d6c` + 본문 12.5px `#9397ab`. 입력 행: input(배경 `#161826`, 보더 `1px solid #3f424d`, radius 6px, 패딩 `8px 10px`, 12.5px) + "등록" 버튼(보더 1px 대표색).
- 하단 액션 행(상단 구분선): "보관", "링크 공유", (예약 가능하면) "예약하기". 앞 두 개는 `#3f424d` 보더, 마지막은 대표색 보더. radius 6px 패딩 `8px 12px` 12px.
- 빈 상태(데스크톱, 상세·인스펙터 모두 없을 때): 패딩 22.4px. "상세 보기" 11px uppercase `#75798c` + 설명 12.5px `#595d6c` line-height 1.7 + 추천 항목 버튼 3개(카드 배경, 좌측 정렬, radius 8px 패딩 11.2px 12.5px).

### 6. 오너 편집 모드
"채널 편집"을 켜면:
- 채널 헤더에 "편집 중" 배지 — 보더 `1px dashed #5d5294`, radius 999px, 패딩 `4px 9px`, 11px `#b5abfc`.
- 각 컴포넌트에 **인라인 툴** 노출. 좌상단(`top: -14px; left: 0`) 타입 라벨 pill: 배경 `#232532`, 보더 `1px solid #5d5294`, 10.5px `#b5abfc`, 패딩 `2px 9px`. 우상단(`top: -14px; right: 0`) 툴 pill 그룹: ↑ / ↓ / 삭제, 배경 `#232532`, 보더 `1px solid #5d5294`, radius 999px, 패딩 3px, 각 버튼 11px `#b2b6ca` 패딩 `2px 7px`.
- 선택된 컴포넌트는 `outline: 2px solid #968ae0; outline-offset: 6px`.
- 스택 맨 아래 추가 힌트 버튼: 보더 `1px dashed #5d5294`, radius 14px, 배경 `rgba(43,39,65,0.35)`, 글자 `#b5abfc`, 패딩 22.4px, 13px, "+ 이 페이지에 UI 컴포넌트 추가".

#### 안 A — 플로팅 바 + 바텀시트 (모바일 앱 방식)
- **FAB** — `position: absolute; right: 22.4px; bottom: 28px(데스크톱) / 86px(모바일); z-index: 36`. 52×52px 원, 보더 `1px solid #968ae0`, 배경 `#2b2741`, 글자 `#d2cefd` 20px, `box-shadow: 0 16px 40px rgba(0,0,0,.65)`. 아이콘 ✎ ↔ × 로 바 접기/펼치기.
- **플로팅 바** — 하단 중앙(`left: 50%; translateX(-50%)`), bottom 28px / 86px, z-index 35. 패딩 8.4px, 보더 `1px solid #5d5294`, radius 999px, 배경 `rgba(35,37,50,0.96)` + blur(12px), 같은 그림자, `max-width: calc(100% - 44px)`, `overflow-x: auto`, 진입 `riseIn .16s`. 액션 4개: "+ 컴포넌트" / "채널 설정" / "페이지 설정" / "새 글". 각 버튼 radius 999px 패딩 `8.4px 14px` 12.5px; 열려 있는 시트에 해당하면 배경 `#2b2741` 글자 `#e7e5fe`, 아니면 transparent / `#b2b6ca`.
- **바텀시트** — 백드롭 `position: fixed; inset: 0; z-index: 50; background: rgba(9,10,16,.6)`, `fadeIn .14s`, 하단 중앙 정렬. 시트: 폭 100% `max-width: 560px`, `max-height: 82vh`, 스크롤, 배경 `#232532`, 보더 `1px solid #595d6c`(하단 없음), radius `14px 14px 0 0`, 그림자 `0 16px 40px rgba(0,0,0,.65)`, 패딩 `16.8px 16.8px 28px`, 애니메이션 `sheetUp .22s cubic-bezier(.2,.8,.2,1)`. 상단에 38×4px radius 999px 그랩바(`#3f424d`), 그 아래 제목 16px weight 500 + "닫기" 12.5px `#75798c`. 백드롭 클릭으로 닫힘(시트 내부 클릭은 stopPropagation).

#### 안 B — 우측 인스펙터 (데스크톱 방식)
데스크톱에서만 활성. 컴포넌트를 클릭하면 우측 열이 인스펙터로 바뀝니다(상세보다 우선).
- 헤더 "인스펙터" 11px uppercase `#b5abfc` + ✕.
- 선택 컴포넌트 타입명 15px weight 500.
- "제목" 텍스트 입력(라이브로 컴포넌트 제목 변경).
- "순서" — "위로" / "아래로" 반반 버튼.
- "여백" — 좁게 / 보통 / 넓게 세그먼트.
- "컴포넌트 삭제" 버튼.
- 구분선 아래 바로가기: "채널 설정 열기" / "페이지 설정 열기" / "컴포넌트 추가" (좌측 정렬, 보더 `1px solid #3f424d`, radius 6px, 패딩 8px, 12px).

### 7. 시트: UI 컴포넌트 추가
`repeat(auto-fill, minmax(150px, 1fr))` gap 8.4px. 항목 버튼: 배경 `#161826`, 보더 `1px solid #3f424d`, radius 8px, 패딩 12px, 좌측 정렬. 라벨 13px + 힌트 11px `#75798c` line-height 1.45.

| 라벨 | 힌트 | type |
| --- | --- | --- |
| 히어로 / 커버 | 큰 이미지와 문구 | hero |
| 콘텐츠 그리드 | 블로그·인스타형 포스트 | grid |
| 콘텐츠 리스트 | 한 줄씩 쌓는 목록 | list |
| 트레이너 소개 | 인물 카드 | trainers |
| 예약 · 캘린더 | 날짜 → 시간 선택 | calendar |
| 상품 리스트 | 가격 · 수량 | products |
| 요금제 | 플랜 비교 | pricing |
| 게시판 | 구독자 글쓰기 · 댓글 | board |
| 갤러리 | 이미지 4~12장 | gallery |
| 위치 · 영업시간 | 주소와 시간 | info |
| FAQ | 질문과 답 | faq |

추가하면 현재 페이지 컴포넌트 배열 **끝에** 붙고, 시트가 닫히고 토스트 "{타입명} 컴포넌트를 추가했습니다".

### 8. 시트: 채널 설정
- **채널명(서브도메인)** — input + 옆에 고정 텍스트 ".mysite.co.kr" 12px `#75798c`. 검증 정규식 `/^[a-z0-9-]{3,20}$/`, 입력은 자동 소문자화. 통과 시 메시지 11.5px `#b5abfc` "사용할 수 있습니다 — {slug}.mysite.co.kr", 실패 시 `#9397ab` "영소문자, 숫자, 하이픈 3~20자". (프로덕션에서는 서버 중복 확인 필요.)
- **채널 표시 이름** — 텍스트 input.
- **채널 대표 색 (HEX)** — 34×34px radius 6px 색 미리보기 + monospace input(`/^#([0-9a-f]{3}|[0-9a-f]{6})$/i` 통과 시 즉시 반영) + 스와치 6개(26×26px radius 6px; 현재 선택은 `2px solid #e9e9ed` 보더, 나머지 `1px solid #3f424d`): `#c9ced6`(기본, 메탈릭 실버) `#a8b0bd` `#8f98a6` `#c9b48a` `#8aa7c9` `#968ae0`.
- **메뉴바 스타일** — 3분할 세그먼트: 상단 고정 없음 / 상단 고정 / 버튼형.
- **채널 아이콘** — 46px radius 8px 이미지 슬롯 + 안내문 11.5px `#595d6c` "이미지를 끌어다 놓거나 눌러서 선택합니다."
- **로딩 애니메이션** — 3분할 세그먼트: 스피너 / 펄스 / 바 + "미리보기" 버튼(보더 1px `#968ae0`, 글자 `#b5abfc`). 미리보기는 1.5초간 전체화면 오버레이(`position: fixed; inset: 0; z-index: 60`, 배경 `#161826`, `fadeIn .12s`)를 띄우고, 안에 로더 + 채널명 12px letter-spacing .1em `#75798c`.
  - 스피너: 40px 원, 보더 `2px solid #3f424d`, `border-top-color` 대표색, `spin .8s linear infinite`.
  - 펄스: 40px 컨테이너 안에 10px 대표색 원 + 링(보더 2px 대표색) `pulseRing 1.4s ease-out infinite` (opacity .25 scale .8 → opacity 0 scale 1.6).
  - 바: 140×3px radius 999px 트랙(`#3f424d`) 안에서 40% 폭 대표색 바가 좌우 왕복.
- **저장** 버튼(보더 1px 대표색, radius 8px, 패딩 11.2px, 13px). 검증 실패 시 토스트 "채널명을 확인하세요".

### 9. 시트: 페이지 설정
- **페이지 이름** input + 검증(같은 채널 내 중복 불가). 통과 "이 채널에서 사용할 수 있는 이름입니다", 실패 "같은 이름의 페이지가 이미 있습니다".
- **페이지 역할** 3분할 세그먼트: 홈 / 메인 / 랜딩. **role은 채널 내에서 유일** — 다른 페이지가 같은 role을 갖고 있으면 자동으로 해제됩니다.
- **페이지 순서** — 페이지 행 목록. 각 행: 배경 `#161826`, 보더 `1px solid #3f424d`, radius 6px, 패딩 8.4px. 이름 12.5px(현재 페이지는 `#e9e9ed`, 나머지 `#9397ab`) + role 11px `#595d6c` + ↑ / ↓ 버튼.
- **페이지 배경 이미지** — 100%×92px radius 8px 슬롯, 플레이스홀더 "배경 이미지 놓기 — 비우면 대표 색 그라디언트".
- 하단: "+ 새 페이지"(반반) / "저장"(반반, 대표색 보더).

### 10. 시트: 새 글 쓰기
- 글 타입 3분할 세그먼트: 글 / 사진 / 상품.
- 제목 input(패딩 11.2px, 14px).
- 본문 textarea(rows 5, 13px, line-height 1.65, `resize: vertical`).
- 타입이 **상품**일 때만 가격 input(flex 1) + 수량 input(96px) 행 추가.
- 이미지 슬롯 100%×110px radius 8px.
- "게시" 버튼. 제목이 비면 토스트 "제목을 입력하세요". 성공 시 게시판 배열 **앞에** 삽입 + 게시판 페이지로 이동 + 토스트(오너 "공지를 게시했습니다" / 방문자 "게시 요청을 보냈습니다"). 오너 글의 태그는 "공지", 방문자는 "자유".

### 11. 예약 플로우 (calendar 컴포넌트)
목업 기준 월: **2026년 8월**.

- 제목 행: "레슨 예약" + 우측 "2026년 8월" 12px `#9397ab`.
- **월 그리드 카드**(패딩 16.8px, 간격 11.2px): 요일 머리글 7열 gap 5.6px, 10.5px `#75798c` 중앙 정렬(일 월 화 수 목 금 토). 날짜 셀 7열 gap 5.6px, 각 셀 `aspect-ratio: 1/1` radius 6px, 12.5px, 안에 날짜 + 4px 점.
  - 선택: 배경 `#2b2741`, 글자 `#e7e5fe`, 보더 1px 대표색, 점 대표색.
  - 예약 가능: 배경 `#161826`, 글자 `#b2b6ca`, 보더 `1px solid #3f424d`, 점 `#5d5294`.
  - 불가(과거/휴무): 배경 transparent, 글자 `#3f424d`, 점 transparent, 클릭 무효.
  - 목업 규칙: 1–5일은 과거, `(day + 6) % 7 === 0`인 날은 휴무(일요일). 프로덕션에서는 서버의 가용 슬롯으로 대체.
- **시간 슬롯** — 헤더 12.5px `#b2b6ca`: "8월 {일}일 · 시간을 고르세요" 또는 "· {시간} 선택됨". 그리드 `minmax(150px, 1fr)` gap 8.4px. 슬롯 버튼: radius 8px, 패딩 11.2px, 좌측 정렬, 시간 14px + "트레이너 · 잔여 N" 11px `#75798c`.
  - 선택 = 배경 `#2b2741` + 보더 1px 대표색 / 가능 = 배경 `#232532` + 보더 `1px solid #3f424d` / 마감 = 배경 transparent + 글자 `#595d6c`, 클릭 무효, 라벨 "마감".
  - 목업 슬롯: 07:00 김도훈 / 09:00 박서연 / 11:00 이준호 / 14:00 김도훈 / 18:00 박서연 / 20:00 이준호. 잔여 = `(선택일 + index) % 4`, 0이면 마감.
- **예약 옵션 폼** (상단 구분선 + 패딩탑 16.8px, wrap flex, 간격 11.2px, 하단 정렬): 예약자명(150px) / 인원(number, 78px, min 1) / 요청 사항(flex 1 1 180px, placeholder "예: 어깨 재활 위주로 부탁드립니다") / CTA 버튼.
  - 라벨 11px `#75798c`, input 배경 `#161826` 보더 `1px solid #3f424d` radius 6px 패딩 `8px 10px` 12.5px.
  - CTA 라벨은 모드별로 다름: 방문자 "예약 요청", 오너 "예약 열기".
  - 시간 미선택 시 토스트 "시간을 먼저 선택하세요". 성공 시 방문자 "예약 요청을 보냈습니다 · 8월 {일}일 {시간}", 오너 "8월 {일}일 {시간} 슬롯을 열었습니다".

**다른 페르소나로의 확장** — 예약 데이터 모델은 페르소나 중립으로 설계되어 있습니다. 슬롯이 (자원 × 시간창 × 정원 × 옵션) 튜플이므로:
- 퍼스널 트레이너: 자원 = 트레이너, 정원 1–4
- 플라워샵: 자원 = 픽업 창, 옵션 = 꽃 종류/색/포장 선택
- 극장: 자원 = 상영관/좌석, 정원 = 좌석 수
- 페스티벌: 자원 = 회차/구역, 정원 = 대량
`reservation_option` 테이블이 이 페르소나별 옵션(라벨, 타입, 선택지, 필수 여부)을 데이터로 흡수합니다.

### 12. 토스트
`position: fixed; left: 50%; translateX(-50%); top: 68px; z-index: 70`. 배경 `#2b2741`, 보더 `1px solid #5d5294`, 글자 `#e7e5fe`, radius 999px, 패딩 `9px 18px`, 12.5px, 그림자 `0 16px 40px rgba(0,0,0,.65)`, `riseIn .16s`. 표시 시간 **2400ms**, 새 토스트가 오면 타이머 리셋.

---

## Interactions & Behavior

### 내비게이션
- 페이지 탭 클릭 → 현재 페이지 변경 + 모바일 탭 동기화 + 열린 상세/선택 해제.
- 콘텐츠 클릭 → 데스크톱은 우측 열에 상세, 모바일은 전체화면 상세. 상세를 열면 편집 선택은 해제됨(상호 배타).
- 모바일 탭바 "내 정보"/"대시보드" → 좌측 정체성 패널이 중앙을 대체.
- hero의 "레슨 예약하기", 상세의 "예약하기" → 예약 페이지로 이동.

### 애니메이션
| 이름 | 값 | 쓰임 |
| --- | --- | --- |
| `sheetUp` | `.22s cubic-bezier(.2,.8,.2,1)`, translateY 100%→0 | 바텀시트 진입 |
| `fadeIn` | `.14s ease` (백드롭) / `.12s` (로더) | 오버레이 |
| `riseIn` | `.16s–.18s ease`, opacity 0→1 + translateY 8px→0 | 상세, 플로팅 바, 토스트 |
| `spin` | `.8s linear infinite` | 로딩 스피너 |
| `pulseRing` | `1.4s ease-out infinite` | 로딩 펄스 |

### 검증 규칙
- 서브도메인: `^[a-z0-9-]{3,20}$`, 소문자 강제, 서버 중복 확인 필요.
- HEX 색: `^#([0-9a-f]{3}|[0-9a-f]{6})$` (대소문자 무시), 통과 시 즉시 미리보기 반영.
- 페이지 이름: 비어 있지 않고, 같은 채널 내 중복 불가.
- 게시글: 제목 필수.
- 예약: 날짜 + 시간 슬롯 모두 선택돼야 제출 가능. 마감 슬롯·과거·휴무일은 클릭 자체가 막힘.
- 댓글: 공백만이면 거부.

### 반응형
단일 임계값 **900px**. 스크롤 소유권이 바뀜에 유의: 데스크톱은 세 열이 각자 스크롤, 모바일은 중앙(또는 오버레이) 하나만 스크롤. 문서 전체는 절대 스크롤하지 않음(`height: 100vh; overflow: hidden`).

### 접근성 (프로덕션에서 반드시 보강)
프로토타입에는 아래가 빠져 있습니다.
- 시트/오버레이 포커스 트랩 + Esc 닫기 + `role="dialog" aria-modal="true"`.
- 캘린더 키보드 내비게이션(방향키 이동, 불가 날짜 `aria-disabled`).
- 모드/메뉴/역할 세그먼트에 `role="radiogroup"`.
- 토스트에 `role="status" aria-live="polite"`.
- `:focus-visible { outline: 2px solid var(--color-accent); outline-offset: 2px; }` — Nocturne 규칙.
- 아이콘 전용 버튼(✕, ↑, ↓, FAB)에 접근 가능한 이름.

---

## State Management

프로토타입의 상태(그대로 옮기면 됩니다):

| 키 | 타입 | 설명 |
| --- | --- | --- |
| `mode` | 'owner' \| 'visitor' | 프로덕션에서는 계정 타입 + 채널 소유 여부로 파생 |
| `editing` | boolean | 오너 편집 모드 |
| `editStyle` | 'sheet' \| 'inspector' | 편집 인터랙션 안 (프로덕션에서는 하나 고정) |
| `fabOpen` | boolean | 플로팅 바 펼침 |
| `page` | pageId | 현재 페이지 |
| `mobileTab` | string | 모바일 하단 탭 |
| `sheet` | null \| 'components' \| 'channel' \| 'page' \| 'write' | 열린 바텀시트 |
| `selected` | compId \| null | 선택된 컴포넌트 (상세와 상호 배타) |
| `detailId` | recordId \| null | 열린 상세 |
| `subscribed` | boolean | 구독 상태 |
| `pc`, `slug`, `name`, `menuStyle`, `loader` | string | 채널 설정 (초안과 확정본 분리: `*Draft`는 시트 안 임시값, "저장"에서 커밋) |
| `pages` | Page[] | `{ id, label, role, comps: [{ id, type, title? }] }` |
| `resDate`, `resSlot`, `resName`, `resQty`, `resNote` | | 예약 초안 |
| `boardPosts`, `comments` | [] | 게시판 데이터 |
| `postType`, `postTitle`, `postBody`, `postPrice`, `postQty` | | 글쓰기 초안 |
| `archiveCat`, `pad` | string | 필터, 컴포넌트 여백 |
| `vw` | number | 창 폭 (resize 리스너) |

주의할 불변식:
1. **상세와 인스펙터는 같은 우측 열을 공유** — 인스펙터가 우선. 하나를 열면 다른 하나를 닫습니다.
2. **role은 채널 내 유일** — 새 role을 지정하면 기존 보유 페이지의 role이 해제됩니다.
3. **시트 초안은 저장 전까지 커밋되지 않음** — 취소해도 채널 설정이 변하지 않아야 합니다.
4. **방문자 모드로 전환하면** 편집 모드·시트·선택이 모두 해제됩니다.

---

## Backend: Neon Postgres

Neon(서버리스 Postgres) 연결은 서버 측에서만 하십시오 — 브라우저에서 커넥션 문자열을 노출하면 안 됩니다. 권장 구성: **Next.js App Router + Route Handlers/Server Actions + `@neondatabase/serverless` + Drizzle ORM**. `DATABASE_URL`은 서버 환경변수로만 두고, 개발/프리뷰는 Neon 브랜치를 쓰면 편합니다.

### 스키마 (제안)

```sql
-- 계정: 하나의 유저가 두 역할을 겸할 수 있음
create table account (
  id            uuid primary key default gen_random_uuid(),
  email         text unique not null,
  password_hash text,
  kind          text not null default 'profile',   -- 'managing' | 'profile'
  display_name  text not null,
  avatar_url    text,
  created_at    timestamptz not null default now()
);

create table channel (
  id            uuid primary key default gen_random_uuid(),
  owner_id      uuid not null references account(id) on delete cascade,
  slug          text unique not null,              -- 서브도메인. ^[a-z0-9-]{3,20}$
  name          text not null,
  category      text not null,                     -- UI 템플릿 = 카테고리
  primary_color text not null default '#c9ced6',
  menu_style    text not null default 'top-pinned',-- 'top' | 'top-pinned' | 'button'
  icon_url      text,
  loader        text not null default 'spin',      -- 'spin' | 'pulse' | 'bar'
  visibility    text not null default 'public',    -- 'public' | 'private'
  created_at    timestamptz not null default now()
);
create index on channel(owner_id);

create table page (
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  label      text not null,
  slug       text not null,
  role       text,                                 -- 'home' | 'main' | 'landing' | null
  position   int  not null default 0,
  bg_image_url text,
  unique (channel_id, label),
  unique (channel_id, slug)
);
-- role은 채널 내 유일
create unique index page_role_unique on page(channel_id, role) where role is not null;

create table component (
  id         uuid primary key default gen_random_uuid(),
  page_id    uuid not null references page(id) on delete cascade,
  type       text not null,                        -- hero|grid|list|trainers|calendar|products|pricing|board|gallery|info|faq
  position   int  not null default 0,
  title      text,
  config     jsonb not null default '{}'::jsonb,   -- 여백, 열 수, 표시 옵션 등
  created_at timestamptz not null default now()
);
create index on component(page_id, position);

-- 콘텐츠(블로그/인스타형 포스트 + 상품 + 트레이너 프로필까지 한 테이블)
create table post (
  id           uuid primary key default gen_random_uuid(),
  channel_id   uuid not null references channel(id) on delete cascade,
  component_id uuid references component(id) on delete set null,
  author_id    uuid references account(id) on delete set null,
  kind         text not null default 'content',    -- 'content' | 'product' | 'person' | 'board'
  kicker       text,
  title        text not null,
  excerpt      text,
  body         text,
  cover_url    text,
  price_cents  bigint,                             -- kind='product'
  currency     text default 'KRW',
  stock_qty    int,
  status       text not null default 'published',  -- 'draft'|'pending'|'published'|'archived'
  view_count   int  not null default 0,
  published_at timestamptz default now()
);
create index on post(channel_id, kind, status, published_at desc);

create table post_image (
  id       uuid primary key default gen_random_uuid(),
  post_id  uuid not null references post(id) on delete cascade,
  url      text not null,
  position int not null default 0
);

create table comment (
  id         uuid primary key default gen_random_uuid(),
  post_id    uuid not null references post(id) on delete cascade,
  author_id  uuid not null references account(id) on delete cascade,
  body       text not null,
  created_at timestamptz not null default now()
);
create index on comment(post_id, created_at);

create table subscription (
  account_id uuid not null references account(id) on delete cascade,
  channel_id uuid not null references channel(id) on delete cascade,
  created_at timestamptz not null default now(),
  primary key (account_id, channel_id)
);

create table access_request (
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  account_id uuid not null references account(id) on delete cascade,
  reason     text,
  status     text not null default 'pending',      -- 'pending'|'approved'|'rejected'
  created_at timestamptz not null default now(),
  decided_at timestamptz,
  unique (channel_id, account_id)
);

create table archive_item (
  id         uuid primary key default gen_random_uuid(),
  account_id uuid not null references account(id) on delete cascade,
  target_type text not null,                       -- 'channel'|'post'|'product'|'board'
  target_id  uuid not null,
  category   text,                                 -- 사용자 정의 분류
  created_at timestamptz not null default now(),
  unique (account_id, target_type, target_id)
);

-- 예약: 페르소나 중립 (트레이너/플라워샵/극장/페스티벌 공통)
create table resource (            -- 트레이너, 픽업 창구, 상영관, 무대
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  name       text not null,
  kind       text not null default 'staff',
  post_id    uuid references post(id) on delete set null  -- 인물 소개 카드와 연결
);

create table reservation_slot (
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  component_id uuid references component(id) on delete set null,
  resource_id uuid references resource(id) on delete set null,
  starts_at  timestamptz not null,
  ends_at    timestamptz not null,
  capacity   int  not null default 1,
  booked_qty int  not null default 0,
  status     text not null default 'open',         -- 'open'|'closed'|'full'
  price_cents bigint,
  check (booked_qty <= capacity)
);
create index on reservation_slot(channel_id, starts_at);

create table reservation_option (  -- 페르소나별 추가 입력 정의
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  label      text not null,                        -- '요청 사항', '꽃 종류', '좌석'
  input_type text not null,                        -- 'text'|'number'|'select'|'date'|'time'
  choices    jsonb,
  required   boolean not null default false,
  position   int not null default 0
);

create table reservation (
  id         uuid primary key default gen_random_uuid(),
  slot_id    uuid not null references reservation_slot(id) on delete cascade,
  account_id uuid references account(id) on delete set null,
  name       text not null,                        -- 예약자명 (비회원 허용)
  qty        int  not null default 1,
  note       text,
  option_values jsonb not null default '{}'::jsonb,
  status     text not null default 'requested',    -- 'requested'|'confirmed'|'cancelled'|'no_show'
  created_at timestamptz not null default now(),
  updated_at timestamptz not null default now()
);
create index on reservation(slot_id);
create index on reservation(account_id, created_at desc);

create table notification (
  id         uuid primary key default gen_random_uuid(),
  account_id uuid not null references account(id) on delete cascade,
  channel_id uuid references channel(id) on delete set null,
  kind       text not null,                        -- 'reservation'|'comment'|'payment'|'access'|'channel'
  body       text not null,
  read_at    timestamptz,
  created_at timestamptz not null default now()
);
create index on notification(account_id, created_at desc);

create table payment (
  id         uuid primary key default gen_random_uuid(),
  channel_id uuid not null references channel(id) on delete cascade,
  account_id uuid references account(id) on delete set null,
  reservation_id uuid references reservation(id) on delete set null,
  post_id    uuid references post(id) on delete set null,
  amount_cents bigint not null,
  currency   text not null default 'KRW',
  status     text not null default 'paid',
  paid_at    timestamptz not null default now()
);

create table page_view (           -- 대시보드 방문자·조회수용
  id         bigserial primary key,
  channel_id uuid not null references channel(id) on delete cascade,
  page_id    uuid references page(id) on delete set null,
  post_id    uuid references post(id) on delete set null,
  visitor_key text,
  viewed_at  timestamptz not null default now()
);
create index on page_view(channel_id, viewed_at desc);
```

### API 계약 (제안)

**공개 / 방문자**
- `GET /api/channels/:slug` → 채널 + 페이지 목록 + role. 비공개면 접근 권한 확인.
- `GET /api/channels/:slug/pages/:pageSlug` → 컴포넌트(순서대로) + 각 컴포넌트가 참조하는 콘텐츠. 화면 1회 렌더에 필요한 것을 한 번에.
- `GET /api/posts/:id` → 상세 + 댓글. 조회수 기록.
- `POST /api/channels/:slug/subscribe` / `DELETE` 동일 경로.
- `POST /api/channels/:slug/access-requests` { reason }
- `POST /api/archive` { targetType, targetId, category } / `DELETE /api/archive/:id` / `GET /api/me/archive?category=`
- `GET /api/channels/:slug/slots?from=&to=` → 월 그리드 + 슬롯 가용성(잔여 = capacity − booked_qty).
- `POST /api/reservations` { slotId, name, qty, note, optionValues } → status 'requested'
- `PATCH /api/reservations/:id` (변경) / `DELETE` (취소)
- `GET /api/me/reservations`, `GET /api/me/notifications`, `POST /api/notifications/:id/read`
- `POST /api/channels/:slug/board` { title, body, images } → 구독자만. 채널 설정에 따라 'pending' 또는 'published'.
- `POST /api/posts/:id/comments` { body }

**오너**
- `POST /api/channels` { slug, name, category } — 생성. `GET /api/channels/check-slug?slug=` 로 중복 확인.
- `PATCH /api/channels/:id` { name, slug, primaryColor, menuStyle, iconUrl, loader, visibility }
- `POST /api/channels/:id/pages`, `PATCH /api/pages/:id` { label, role, bgImageUrl }, `PATCH /api/channels/:id/pages/reorder` { orderedIds }, `DELETE /api/pages/:id`
- `POST /api/pages/:id/components` { type }, `PATCH /api/components/:id` { title, config }, `PATCH /api/pages/:id/components/reorder` { orderedIds }, `DELETE /api/components/:id`
- `POST /api/channels/:id/posts`, `PATCH /api/posts/:id`, `DELETE /api/posts/:id` (아카이브는 status='archived')
- `GET /api/channels/:id/dashboard` → 오늘/이번주 예약, 방문자·조회수, 구독자, 게시판 신규 글·댓글, 매출, 대기 중 접근 요청.
- `PATCH /api/access-requests/:id` { status }
- `PATCH /api/reservations/:id` (확정/취소), `POST /api/channels/:id/slots` (슬롯 열기 — 오너 CTA "예약 열기")

**순서 변경(reorder)** 은 개별 PATCH를 여러 번 보내지 말고 `orderedIds` 배열을 한 번에 받아 트랜잭션으로 position을 다시 쓰는 방식을 권합니다. 인라인 ↑/↓ 클릭이 잦으므로 클라이언트는 낙관적 업데이트 후 디바운스 저장하십시오.

**서브도메인 라우팅** — `{slug}.domain.com` 은 Next.js middleware에서 Host 헤더를 읽어 `/_channels/{slug}/...` 로 rewrite 하고, 와일드카드 도메인 + 인증서를 준비하십시오.

**이미지 업로드** — 프로토타입의 드래그&드롭 슬롯은 실제 업로드로 대체됩니다. 오브젝트 스토리지(S3/R2)에 presigned PUT으로 올리고 DB에는 URL만 저장하십시오. Neon에 바이너리를 넣지 마십시오.

---

## Design Tokens

Nocturne 디자인 시스템에서 파생. 값은 `_ds/nocturne-.../styles.css`의 `:root` 변수를 정본으로 삼고, 아래는 이 디자인이 실제로 사용한 부분집합입니다.

**색**
| 역할 | 값 |
| --- | --- |
| 페이지 배경 | `#161826` |
| 패널 배경(좌·우) | `#1b1d2b` |
| 카드 표면 | `#232532` |
| 보더 | `#3f424d` |
| 미세 구분선 | `rgba(233,233,237,0.08)` |
| 본문 텍스트 | `#e9e9ed` |
| 보조 텍스트 | `#b2b6ca` |
| 뮤트 텍스트 | `#9397ab` |
| 더 뮤트 | `#75798c` |
| 가장 흐림 | `#595d6c` |
| 시스템 액센트(빌더 UI) | `#968ae0` |
| 액센트 밝은 텍스트 | `#b5abfc` / `#d2cefd` / `#e7e5fe` |
| 액센트 틴트 배경 | `#2b2741` |
| 액센트 보더(편집) | `#5d5294` |
| **채널 대표색(기본)** | `#c9ced6` (메탈릭 실버, 오너 편집 가능) |

액센트 두 종류를 구분하십시오. `#968ae0` 계열은 **빌더 도구 UI**(편집 크롬, 시트, 토스트, FAB)에 쓰고, **채널 대표색**은 채널 콘텐츠 자체(채널명, 링크, CTA 보더, 캘린더 선택, 가격)에 씁니다. 그래야 오너가 대표색을 바꿔도 도구 UI가 흔들리지 않습니다.

**간격** (Nocturne 0.7× 밀도 스케일) — 2.8 / 4 / 5.6 / 8.4 / 11.2 / 16.8 / 22.4 / 28 / 33.6 px

**타이포** — Inter (헤딩·본문 공통), 한글은 Noto Sans KR 폴백. 헤딩 weight는 **500을 넘기지 않음**.
| 쓰임 | 크기 / 특성 |
| --- | --- |
| Hero h1 | 34px / 500 / lh 1.15 / ls -.02em |
| 가격(요금제) | 25px / 500 / ls -.02em |
| 섹션 h2 | 21px / 500 / ls -.01em |
| 지표 값 | 19px / 500 |
| 상세 가격 | 18px |
| 시트 제목 | 16px / 500 |
| 채널명, 인스펙터 제목 | 15px / 500 |
| 카드 제목 | 14 – 14.5px / 500 |
| 본문 | 13.5px / lh 1.6 – 1.75 |
| UI 라벨·버튼 | 12 – 12.5px |
| 메타 | 11 – 11.5px |
| kicker / uppercase | 10.5 – 11px / ls .05 – .14em |

**radius** — 5px(미니 버튼) / 6px(input, 소버튼) / 8px(카드, 버튼) / 14px(컴포넌트 섹션, 시트 상단) / 999px(pill)

**그림자** — 부유 요소는 `0 16px 40px rgba(0,0,0,0.65)` 하나만. 겹쳐 쌓지 마십시오.

**Nocturne 규칙 준수 사항**
- 액센트는 선과 광으로만 — 큰 면적을 채우지 않습니다. 프라이머리 버튼은 **1px 보더 + 투명 배경**이며 절대 솔리드 필이 아닙니다.
- 순수 검정/흰색 금지. 그림자만 예외.
- 아이콘은 Phosphor Icons를 쓰십시오. 프로토타입에는 도형 자리만 있습니다.
- 사진은 `.lighten`(`mix-blend-mode: lighten`) 래퍼를 통과시키고, 어두운 배경에서 촬영된 이미지를 선호하십시오.
- 헤딩은 좌측 정렬, 비대칭 레이아웃. 우측에 여백을 남깁니다.

---

## Assets
- `image-slot.js` — 프로토타입의 드래그&드롭 이미지 플레이스홀더 웹 컴포넌트. **프로덕션에서는 실제 업로드 컴포넌트로 교체**하십시오. 디자인에서 이미지가 들어가는 자리를 표시하는 역할입니다.
- 사진 에셋 없음. 히어로, 콘텐츠 카드, 트레이너 사진, 갤러리, 채널 아이콘, 페이지 배경 자리는 모두 비어 있습니다. 각 슬롯의 placeholder 텍스트가 어떤 이미지가 들어갈 자리인지 설명합니다.
- 아이콘 없음 — Phosphor Icons로 채우십시오.
- 목업 텍스트(트레이너 이름, 프로그램 설명, 게시글, 요금제, FAQ, 주소)는 전부 가상입니다. 실제 채널 데이터로 교체하십시오.

## Files
| 파일 | 내용 |
| --- | --- |
| `Channel Builder.dc.html` | 전체 프로토타입. 상단에 마크업(레이아웃·컴포넌트·시트·오버레이), 하단 `<script>`에 상태 로직과 목업 데이터. 브라우저에서 바로 열립니다. |
| `image-slot.js` | 이미지 플레이스홀더 웹 컴포넌트 |
| `support.js` | 프로토타입 런타임. **참고용일 뿐, 이식 대상이 아닙니다.** |
| `nocturne/styles.css` | Nocturne 토큰 시트 — 색·간격·타이포·radius·shadow의 정본 |
| `nocturne/readme.md` | Nocturne 사용 지침 |
| `nocturne/theme.json` | 테마 파라미터(기계 판독용) |

## 구현 순서 제안
1. Neon 스키마 마이그레이션 + 계정/인증 (Managing / Profile 두 종류).
2. 채널 생성 플로우 (slug 검증 → 카테고리 = UI 템플릿 선택) + 서브도메인 미들웨어 rewrite.
3. 읽기 경로: 채널 → 페이지 → 컴포넌트 → 콘텐츠 렌더링. 여기서 3분할/모바일 반응형 셸을 확정합니다.
4. 상세 뷰(우측 열 / 전체화면 전환)와 구독·아카이브.
5. 예약: 슬롯 생성(오너) → 캘린더 가용성 조회 → 요청/변경/취소 → 알림.
6. 편집 모드: 컴포넌트 추가·순서·삭제 → 채널 설정 → 페이지 설정. 편집 방식은 **하나만** 채택.
7. 게시판·댓글·승인 워크플로.
8. 대시보드 집계 쿼리 + 알림 센터.
9. 접근성 보강(포커스 트랩, 키보드 캘린더, aria) — 미루지 마십시오.
