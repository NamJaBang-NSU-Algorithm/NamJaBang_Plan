# 천안 온 — 팀 역할 분담

> 팀 인원: 4명. 안민영(Spring Boot 백엔드 경험 有, Flask는 처음)을 제외한 3명은 백엔드 코딩 경험 없음.
> 프론트엔드는 AI 코드 생성 도구로 Figma 화면("천안 온 - UI/UX 디자인" 파일)을 그대로 구현할 예정이라 코딩 난이도가 상대적으로 낮다 — 그래서 4명 전원을 **백엔드(Flask + MySQL)** 중심으로 배정한다.

## 1. 진행 순서

**1단계 — 안민영이 기준 틀을 먼저 만든다**
- Flask 앱 뼈대(app factory, config, DB 연결) 구축
- `auth` 블루프린트(카카오 OAuth + JWT) 구현
- `events` 블루프린트를 "정석 패턴" 예시로 끝까지 완성 (route → schema → service → model 흐름을 다른 팀원이 그대로 따라할 수 있게)
- `event` 모델(`models/event.py`)을 가장 먼저 확정 — 공공 API 데이터 수집 담당자들이 이 모델이 있어야 작업을 시작할 수 있음

**2단계 — 나머지 3명이 1단계의 패턴을 복제해 각자 영역을 맡는다**

## 2. 역할 분담표

| 담당 | 역할 | 관련 문서 |
|---|---|---|
| **안민영** | 백엔드 뼈대 + `auth`(카카오 로그인/JWT) + `events`(목록·상세, 기준 패턴) + `directions`(길찾기, 외부 지도 API 연동) | `cheonan_on_architecture_spec.md` 2절, `cheonan_on_api_spec.md` 1·2절 |
| **팀원 A** | `reviews`(리뷰 CRUD + 사진 업로드) + `bookmarks`(북마크 CRUD) — 둘 다 패턴이 단순한 CRUD라 하나로 묶음 | `cheonan_on_api_spec.md` 3·4절 |
| **팀원 B** | 공공 API 데이터 수집 #1 — 한국관광공사 TourAPI 연동, 천안 지역 행사 데이터를 가져와 `event` 테이블에 적재하는 스크립트 | `cheonan_on_architecture_spec.md`의 데이터 수집 관련 서비스, 천안시 기획서 8절 |
| **팀원 C** | 공공 API 데이터 수집 #2 — 공공데이터포털 연동 (+ `users` 마이페이지 API처럼 가벼운 나머지 엔드포인트) | `cheonan_on_api_spec.md` 5절 |

## 3. 진행 시 참고

- 팀원 B/C의 데이터 수집 스크립트는 REST 블루프린트가 아니라 **한 번(또는 주기적으로) 실행하는 배치 스크립트**다. Flask 지식보다 "외부 API 호출 → JSON 파싱 → DB insert"만 알면 되므로 처음 백엔드를 접하는 사람에게 적합한 입문 과제이지만, `event` 모델이 먼저 있어야 시작할 수 있다 — 안민영의 1단계가 끝난 뒤 착수.
- TourAPI/공공데이터포털 중 무엇을 우선할지, 공식 API로 커버되지 않는 최신 행사를 수동 보완 입력할지는 천안시 기획서 8절 참고 (수동 보완 입력은 관리자 페이지가 필요한데 이번 스코프에는 없으므로, 필요하면 DB에 직접 INSERT하는 방식으로 임시 대응).
- 안민영은 Spring Boot 경험이 있어 Blueprint≈Controller, SQLAlchemy≈JPA, Flask-JWT-Extended≈Spring Security JWT 정도로 대응시키면 적응이 빠르다. 나머지 팀원에게도 이 대응 관계를 공유하면 온보딩이 쉬워진다.
- `cheonan_on_feature_spec.md`에 정리된 "디자인 갭"(로그아웃 버튼 없음, 헤더 검색/하트 아이콘 동작 미정의, 페이지네이션 UI 없음)은 프론트/백엔드 모두에 영향이 있으니 착수 전에 팀 전체가 한 번 훑어보는 것을 권장.
