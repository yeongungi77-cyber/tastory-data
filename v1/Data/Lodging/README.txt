TASTORY 숙소 목록 (Data/Lodging) — 2026-10-12
=============================================

지자체가 공개한 숙박업 허가 명단(민박 제외)을 TASTORY 가 추출·가공한 목록. 도시 하나 = 파일 하나(<도시 id>.json).
프로그램을 빌드할 때 함께 들어간다. 손으로 고치지 말고 tools/lodging_build.py 로 다시 만든다(docs/data/LODGING_DATA.md).
서버(lodgingdb.go)가 처음 쓸 때 검사해서 통과한 숙소만 쓴다. go test ./... 의 TestLodgingDataClean 이 경고 0개를 요구한다.

[파일 칸]
  city, builtOn, source{publisher,url,asOf,license,note}★ 출처 표시 의무 — 화면 숙소 검색 결과와 설정 › 출처에 나온다
  coordSources   좌표 출처 설명
  lodgings[]     id★(lg-<도시>-<10자리>, 우리 숙소 ID — 계획표 숙소의 lodgingId), name★, nameKo, aliasesKo(TASTORY 번역),
                 tags(한국어 검색 태그 — tools/lodging_tags.json 규칙),
                 address★(번지까지, 건물·호실·층 뺌), type(허가 종류), rows(원래 명단 행 번호),
                 lat·lng·coordSource(abr|gsi|city)·coordPrecision(building|block|town, city 는 unverified)·coordOn

[넣지 않는 것]  영업자 이름·전화번호, 호실 단위 주소, 민박(特区民泊·住宅宿泊事業), Google·OSM 좌표, 추정 좌표

[출처]
  福岡市 旅館業営業許可施設一覧 https://data.bodik.jp/dataset/401307_ryokangyou_eigyoukyoka (2026-09-30 기준, CC BY)
  大阪市 宿泊施設一覧 https://www.city.osaka.lg.jp/kenko/page/0000382418.html (2026-06-30 기준, CC BY 4.0)
  TASTORY 가 추출·가공(이름·주소 정리, 같은 숙소 합침, 좌표 추가)
