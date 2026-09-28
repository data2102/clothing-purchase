# 옷 구매 비교 — 작업 규칙

> 이 저장소는 콩순(사용자)의 **옷 구매 고민을 줄이는 도구**를 만드는 곳이다.
> 작업을 시작하기 전에 `PROGRESS.md`를 먼저 읽는다. 지금까지의 결정, 현재 상태, 다음 할 일이 거기 있다.

## 풀려는 문제 (사용자가 직접 말한 불편 3가지)

1. 옷마다 사이즈가 달라 **내가 입는 옷과 비교**해야 하는데, 매번 노션과 쇼핑몰을 오가며 본다.
2. 예산이 한정돼 한 번에 사지 않으므로 **여러 번에 걸쳐 다시 비교**하게 된다.
3. 여러 쇼핑몰을 보다 보니 **어디서 어떤 옷을 봤는지 기억이 안 난다.** 앱을 다 깔기도 애매하다.

모든 기능은 이 셋 중 무엇을 푸는지 말할 수 있어야 한다. 셋 중 어디에도 안 걸리면 만들지 않는다.

## 구성 요소 (전부 저장소 밖에 있다)

| 무엇 | 위치 | 비고 |
|---|---|---|
| 옷 쇼핑 리스트 (노션 DB) | https://app.notion.com/p/7bffa6934d1f445389d6dfa3f24917f1 | 데이터소스 `collection://4b8fb3bc-08ca-421e-be5e-927e164db79f` |
| 기본 보기(view) | https://app.notion.com/p/7bffa6934d1f445389d6dfa3f24917f1?v=d0728fbfc25e4319a0443244965c159e | 비교판이 이 view를 읽는다 |
| 옷 사이즈 (노션 페이지) | https://app.notion.com/p/1d920f17509d80819012cbfe8dfebcfe | 원래 실측 표(손대지 않음). 아래 DB·휴지통 페이지의 부모 |
| 내 옷 사이즈 (노션 DB) | https://app.notion.com/p/54b0c8f7f41b48669d57c273d1c9d817?v=8b70f481f6cd444f86e3626359664852 | 데이터소스 `collection://51969fd0-e3b5-475a-8a8e-5f7bbf736580`. **비교 기준의 원본**. 비교판에서 추가·수정 |
| 🗑 삭제한 옷 (노션 페이지) | https://app.notion.com/p/3e920f17509d8192a8b3cc1a5d54ff8c | 휴지통에서 「완전 삭제」한 쇼핑 리스트 행이 옮겨지는 곳 |
| 콩순 옷장 비교판 (아티팩트) | https://claude.ai/artifact/K6azdzCdcRxpN8sUQv51NJ | 원본: `artifact/closet-compare.html` |

### 노션 DB 칸
- 원래 칸: `옷 이름`(title) `종류`(셔츠/바지/신발/니트/맨투맨) `가격` `구매사이트` `구매링크` `사이즈`(글) `S` `M` `L`(글, "어깨 51 / 가슴 65.5 / 소매 61 / 총장 73" 형식) `캡쳐사진`(파일, 비어 있음)
- 2026-09-28 추가: `결정`(살래/고민/패스) `비교사이즈`(글) `어깨` `가슴` `총장` `허리`(숫자) `사진ID`(글, 아티팩트 asset id)
- 2026-09-28 추가(2): `비교옷ID`(글, 내 옷 사이즈 DB 페이지 id — 비어 있으면 자동) `휴지통`(체크박스)
- **원래 칸은 지우거나 덮어쓰지 않는다.** 바꿀 게 있으면 새 칸을 추가한다. (사용자가 비교판 「수정」에서 직접 고친 값은 예외)

### 내 옷 사이즈 DB 칸
`이름`(title) `구분`(상의/하의/신발) `구매처` `사이즈` `어깨` `가슴` `총장` `허리` `밑단`(숫자) `비고`

### 비교 기준
- 노션 「내 옷 사이즈」 DB를 비교판이 실시간으로 읽는다(코드에 고정값 없음). 처음 값은 「옷 사이즈」 페이지 표와 신발 265를 그대로 옮긴 것(2026-09-28).
- 옷별 `비교옷ID`가 있으면 그 옷과, 없으면 치수 차이 합이 가장 작은 같은 구분의 옷과 비교한다.

## 비교판(아티팩트) 고치는 법

1. `artifact/closet-compare.html`을 고친다.
2. Artifact 도구로 publish 하되 **`url: https://claude.ai/artifact/K6azdzCdcRxpN8sUQv51NJ`를 꼭 넘긴다.** 안 넘기면 새 아티팩트가 생겨 링크가 바뀐다. 다른 세션에서 처음 고칠 때는 먼저 `action: "read"`로 현재 판을 읽고 그 위에 고친다.
3. `capabilities`는 생략한다(생략하면 기존 선언 유지). 현재 선언: `mcp`(Notion: `notion-query-data-sources`, `notion-update-page`, `notion-create-pages`, `notion-move-pages`), `sample`, `assets`.
4. 고친 파일을 커밋한다. 저장소 원본과 게시본이 어긋나지 않게 한다.

### 노션 호출 모양 (실제 확인한 것)
- 읽기: `notion-query-data-sources` `{data:{mode:"view", view_url}}` → payload `{results:[{…칸 이름: 값, url}], has_more}`. 페이지 id는 `url` 끝의 32자리.
- 결정 저장: `notion-update-page` `{page_id, command:"update_properties", properties:{"결정":"살래"}}` (취소는 `null`)
- 추가: `notion-create-pages` `{parent:{type:"data_source_id", data_source_id:"4b8fb3bc-08ca-421e-be5e-927e164db79f"}, pages:[{properties:{…}}]}`
- 휴지통: `notion-update-page` `properties:{"휴지통":"__YES__"}` (되돌리기 `"__NO__"`). 읽을 때도 `"__YES__"/"__NO__"` 문자열.
- 완전 삭제: `notion-move-pages` `{page_or_database_ids:[id], new_parent:{type:"page_id", page_id:"3e920f17509d8192a8b3cc1a5d54ff8c"}}` (2026-09-28 시험 행으로 확인)
- 읽기 응답에서 빈 숫자 칸은 키 자체가 없다. 결과가 100개 넘으면 `has_more`/`next_cursor` → `start_cursor`로 이어 읽는다.

## 작업 규칙

- **결과물 먼저, 한국어로.** 설명은 결과물 아래 짧게.
- **숫자는 원본에서만.** 가격·실측은 쇼핑몰 화면이나 노션에 있는 값만 쓴다. 없으면 비워 두고 "실측 없음"으로 표시한다. 지어내지 않는다.
- **판정 기준을 숨기지 않는다.** 핏 판정은 가슴단면(바지는 허리단면) 차이로 ±2 비슷 / 2~5 조금 / 5 초과 크게. 기준을 바꾸면 페이지 하단 설명과 `PROGRESS.md`를 같이 고친다.
- 노션에 쓰기 전에, 사용자가 요청하지 않은 대량 수정은 먼저 묻는다.
- 중요한 결정은 `PROGRESS.md` "결정 이력" 맨 위에 **이유와 함께** 남긴다.
