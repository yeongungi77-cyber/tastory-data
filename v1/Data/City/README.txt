TASTORY 도시 데이터 (Data/City) — v1.313
=========================================

도시 하나 = 파일 하나(영문 소문자 이름.json, UTF-8). 프로그램을 빌드할 때 함께 들어간다(파일을 고친 뒤 다시 빌드).
서버가 시작할 때 모든 파일을 검사해 통과한 도시만 화면에 넘긴다(/api/cities.js).
잘못된 파일·도시는 그 도시만 건너뛰고 앱은 그대로 동작한다. 작은 문제(검색 범위 밖 지도 점, 없는 분류로 가는 별칭,
미식 프로필에 없는 값 등)는 그 부분만 빼고 쓴다.
무엇이 빠졌는지 보기: 개발판 화면 콘솔의 '도시 데이터 검사' 경고, 또는 go test ./...
(city_test.go TestCityFilesClean 이 경고 0개를 요구한다).

[칸]  ★ = 필수(ready 도시)
  schema★        1
  id★            영문 소문자 2~32자(osaka). 파일 이름과 같게
  name★          화면 이름(오사카). 계획표에 region 으로 저장되는 값 — 바꾸면 예전 계획표가 이 도시를 못 찾으니 바꾸지 않는다
  ja             일본어 이름
  en★            영문 이름. Google 검색어 끝에 붙는다(たこ焼き Osaka)
  status★        ready = 여행 가능 / soon = 지역 선택·홈에 '업데이트 예정'으로만(name·desc·foods 만 씀)
  order          지역 선택·홈 순서(작을수록 앞). 첫 ready 도시가 기본 도시(지금 후쿠오카)
  updated        고친 날짜(메모)
  desc★          도시 설명(400자 이내)
  famous         숙소 설정 '○○는 무엇으로 유명할까요?' 아래 한 줄
  searchBounds★  [남, 서, 북, 동] Google 검색·숙소 찾기 범위(일본 안, 가로·세로 3도 이내)
  mapBounds      [남, 서, 북, 동] 도시 지도가 처음 보여 줄 도심 범위(searchBounds 안). 없으면 searchBounds
  center         [위도, 경도] 큰 지도 가운데(mapBounds 안). 없으면 mapBounds 가운데
  boundary       {"query": "大阪市", "label": "오사카시", "source": "...", "rings": [[[위도,경도], ...], ...]} 행정 경계선.
                 rings·source 는 손으로 쓰지 않고 python tools/city_boundary.py --write 로 넣는다(OpenStreetMap, ODbL 1.0 —
                 이 경계선 자료만 ODbL, 지도 화면에 '경계선 © OpenStreetMap' 출처). 2026-10-13부터 앱은 실행 중에 경계를 묻지 않으므로
                 rings 가 없으면 경계선을 그리지 않음
  lodgingExample 숙소 입력칸 예시
  foods          대표 음식(숙소 설정 칩·처음 찾아볼 음식). 이름은 기본 분류 세부분류와 같게 하고, subs 에 info(설명)를 넣는다
  towns          주변 소도시(로드맵 38 '지역 미식 탐색'용 — 아직 화면에 쓰지 않음)
  points         지도 점 [{"name", "lat", "lng", "kind", "desc"}]
                 kind: 교통·쇼핑·명소·먹거리·거리·공원·전망(아이콘). searchBounds 밖의 점은 뺀다
  categories★    기본 분류 [{"major": 대분류, "subs": [세부분류…]}] — 화면 순서 그대로
                 · 대분류는 다른 도시와 개념이 같으면 같은 이름으로(안전 표시·취향 연결이 이름으로 이어진다)
                 · 세부분류 이름은 도시 안에서 한 번만. '기타'·'미분류'·'기타 음식점'은 쓰지 않는다(앱이 따로 씀)
                 · 이름을 바꾸거나 빼도 이미 저장된 음식점은 그대로이고, 왼쪽 분류에 '이전 분류'로 남는다
                 · 후쿠오카 순서는 자동 분류 동점 처리에도 쓰인다(classify.go basicSubOrder 와 같아야 함 — 시험이 확인)
  alias          자동 분류 이름 바꾸기 {"다른 도시 세부분류": "이 도시 세부분류"}
                 예: 오사카 "하카타 라멘": "돈코츠 라멘", 후쿠오카 "키츠네 우동": "우동"
                 바뀐 이름은 이 도시 기본 분류에 있어야 한다. 표에 없는 분류는 저장할 때 '내 분류'로 자동 생성된다
  majors         대분류 단위 연결 {"코나몬·철판": {"allergy": ["wheat", "egg"]}} (값은 아래 subs 와 같음)
  subs           세부분류·음식별 정보 {"타코야키": {…}}
                   ja      일본어 이름(음식 설명에 표시)
                   search  Google 검색어(일본어 권장, 앞 6개만). 검색어 1개 = 요청 수가 가장 적다.
                           없으면 프로그램 안 검색어(search.go) → 분류 이름 그대로
                   match   가게 이름·Google 유형·소개에 이 말이 있으면 이 세부분류로 자동 분류(2~40자)
                           한국어는 쓰지 않는다 — 검색할 때 고른 분류 이름이 함께 들어가서 모든 결과가 그 분류가 되기 때문
                   allergy·diet·dislike  안전 표시·'피하고 싶은 음식' 참고 표시
                           값은 Data/Taste/questions.json 의 SAFETY_ALLERGY·SAFETY_DIET·SAFETY_DISLIKE 선택지
                           (wheat·egg·fish·shellfish… / pork·beef·seafood·raw_food… / seafood·offal…)
                   taste   미식 프로필 음식 id(Data/Taste/foods.json — takoyaki·udon·regional 등). 취향 추천에 연결
                   info    {"desc": 설명(필수, 300자 이내), "taste": 맛 포인트, "tip": 알아두면 좋은 점}

  (curated — 2026-10-12 의 옛 「TASTORY 선정 가게」 칸. Google Place ID 기준이라 v1.333 Google 제거 뒤 쓸 수 없어 v0.33 에서 삭제(사장님 결정).
   선정 가게는 Data/Restaurants 의 영업 확인 목록(확인표 → restaurant_build.py)으로 대신한다. 옛 도시 파일에 칸이 남아 있어도 무시된다.)

[자동 분류 순서]
  1. 프로그램 안의 명시 규칙(classify.go — 하카타 라멘·모츠나베 등)
  2. 도시 파일의 match (order 순 도시, 기본 분류 순서) — 도시와 상관없이 같은 이름으로 나온다
  3. 프로그램 안의 점수 규칙(classify.go·classify2.go)
  화면이 결과를 지금 여행 도시의 분류로 옮긴다:
  이미 있는 분류(기본·내 분류)는 그대로 → 이 도시 기본 분류의 같은 세부분류 → alias → 그 밖에는 '내 분류'로 자동 생성.
  후쿠오카 기본 음식의 검색어·분류 낱말은 프로그램 안(search.go·classify.go)에 있다(동작을 바꾸지 않으려고 그대로 둠).

[새 도시 추가]
  1. 비슷한 도시 파일을 복사해 id·name·en·범위·지도 점·기본 분류·음식 정보를 바꾼다.
     처음엔 status "soon" 으로 두고 내용을 채운 뒤 "ready" 로 바꾼다.
  2. 새 음식은 subs 에 search·match·info, 안전·취향 연결을 적는다. 다른 도시 이름은 alias 로 이 도시 분류에 연결한다.
  3. go test ./...  (경고 0개, 모든 기본 세부분류에 검색어, 대표 음식에 설명이 있는지)
  4. 빌드 → tests/city/browser_city.py (지금 다섯 도시 기준 시험 — 도시를 늘리면 기대값을 함께 고친다)
  5. 다른 도시 파일의 alias 에도 새 음식 이름을 이어 둔다(그 도시 여행에서 가까운 분류로 옮겨지게)
