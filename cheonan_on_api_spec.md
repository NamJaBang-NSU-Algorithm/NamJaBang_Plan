# 천안 온 — API 명세서 (API Spec)

> Base URL: `/api/v1`
> 인증 방식: 카카오 OAuth 로그인 후 발급되는 자체 JWT(Access Token)를 `Authorization: Bearer {token}` 헤더로 전달
> 모든 Response는 `application/json`. 아래 예시는 성공(200) 기준이며, 실패 시 공통 포맷은 문서 하단 참고.
> 각 엔드포인트는 `cheonan_on_feature_spec.md`의 화면 ID를 기준으로 어떤 화면에서 쓰이는지 명시한다.

---

## 1. 인증 (Auth)

### 1.1 카카오 로그인
`POST /api/v1/auth/kakao/login`

카카오 인가 코드로 로그인/최초 가입 처리 후 서비스 자체 토큰 발급. `04_로그인`에서 사용.

**인증 필요**: 아니오

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| authorizationCode | string | Y | 카카오 OAuth 인가 코드 |
| redirectUri | string | Y | 프론트에서 사용한 redirect URI (카카오 토큰 교환용) |

**Response 예시**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs...",
  "user": {
    "id": 12,
    "nickname": "안민영",
    "profileImageUrl": "https://.../avatar.png",
    "isNewUser": false
  }
}
```

### 1.2 로그아웃
`POST /api/v1/auth/logout`

**인증 필요**: 예

**Request Body**: 없음 (Authorization 헤더의 토큰 기준으로 리프레시 토큰 무효화)

**Response 예시**
```json
{ "message": "로그아웃되었습니다." }
```

### 1.3 토큰 재발급
`POST /api/v1/auth/token/refresh`

**인증 필요**: 아니오 (refreshToken 자체가 인증 수단)

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| refreshToken | string | Y | 발급받은 리프레시 토큰 |

**Response 예시**
```json
{
  "accessToken": "eyJhbGciOiJIUzI1NiIs...",
  "refreshToken": "eyJhbGciOiJIUzI1NiIs..."
}
```

---

## 2. 행사 (Event)

### 2.1 행사 목록 조회
`GET /api/v1/events`

`01_홈`(인기 행사), `02_행사목록`, `02b_행사목록_캘린더뷰`, `02c_행사목록_지도뷰`, `02d_행사목록_검색결과없음`이 모두 이 엔드포인트 하나를 쿼리 파라미터 조합만 다르게 해서 사용한다.

**인증 필요**: 아니오 (단, `isBookmarked` 필드를 채우려면 선택적으로 토큰 사용)

**Query Parameter**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| q | string | N | 검색어 (행사명/장소/키워드) |
| category | string | N | `FESTIVAL`\|`EXHIBITION`\|`PERFORMANCE`\|`EXPERIENCE`\|`SEASONAL`. 미지정 시 전체 |
| region | string | N | `DONGNAM`\|`SEOBUK`. 미지정 시 전체 |
| priceType | string | N | `FREE`\|`PAID`. 미지정 시 전체 |
| startDate | date | N | 조회 시작일 (`02b` 캘린더 뷰에서 해당 월의 1일로 전달) |
| endDate | date | N | 조회 종료일 (`02b`에서 해당 월의 말일로 전달) |
| sort | string | N | `latest`(등록순, 기본값) \| `popular`(인기순, `01_홈` 인기 행사에서 사용) \| `dateAsc`(임박순) |
| page | integer | N | 페이지 번호 (기본 1) |
| size | integer | N | 페이지당 개수 (기본 20, `01_홈`은 8로 호출) |

**Response 예시**
```json
{
  "totalCount": 12,
  "events": [
    {
      "id": 1,
      "title": "2025 천안흥타령춤축제",
      "category": "FESTIVAL",
      "region": "DONGNAM",
      "startDate": "2025-10-11",
      "endDate": "2025-10-13",
      "location": "천안종합운동장 일원",
      "latitude": 36.8151,
      "longitude": 127.1139,
      "priceType": "FREE",
      "priceAmount": null,
      "imageUrl": "https://.../thumb.jpg",
      "isBookmarked": true
    }
  ]
}
```
> `02c_행사목록_지도뷰`는 이 응답의 `latitude`/`longitude`를 그대로 지도 핀 좌표로 사용한다. `02d`는 `totalCount: 0`, `events: []` 응답을 받았을 때 프론트에서 렌더링하는 상태다 (별도 엔드포인트 아님).

### 2.2 행사 상세 조회
`GET /api/v1/events/{eventId}`

`03_행사상세`(및 `03b`)에서 사용.

**인증 필요**: 아니오 (선택적 토큰으로 `isBookmarked` 채움)

**Response 예시**
```json
{
  "id": 1,
  "title": "2025 천안흥타령춤축제",
  "category": "FESTIVAL",
  "region": "DONGNAM",
  "startDate": "2025-10-11",
  "endDate": "2025-10-13",
  "startTime": "10:00",
  "endTime": "21:00",
  "location": "천안종합운동장 일원",
  "address": "충남 천안시 동남구 ...",
  "latitude": 36.8151,
  "longitude": 127.1139,
  "priceType": "FREE",
  "priceAmount": null,
  "organizer": "천안시, 천안흥타령춤축제 추진위원회",
  "contact": "041-000-0000",
  "homepageUrl": "https://cheonan-dance.kr",
  "description": "천안의 대표 축제인 흥타령춤축제가...",
  "imageUrl": "https://.../hero.jpg",
  "isBookmarked": true,
  "averageRating": 4.7,
  "reviewCount": 128
}
```
> `reviewCount: 0`이면 프론트는 `03b_행사상세_리뷰없음` 상태로 렌더링한다.

### 2.3 길찾기(경로) 조회
`GET /api/v1/events/{eventId}/directions`

`06_길찾기`에서 사용. 서버가 카카오모빌리티 등 외부 길찾기 API를 호출해 결과를 그대로 중계한다 (`cheonan_on_erd_spec.md` 4절 참고 — DB에 저장하지 않음).

**인증 필요**: 아니오

**Query Parameter**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| originLat | decimal | Y | 사용자 현재 위치 위도 |
| originLng | decimal | Y | 사용자 현재 위치 경도 |
| mode | string | N | `transit`(대중교통, 기본값) \| `car`(자동차) \| `walk`(도보) |

**Response 예시**
```json
{
  "eventId": 1,
  "mode": "transit",
  "durationMinutes": 42,
  "distanceMeters": 18400,
  "arrivalTime": "2025-10-11T14:42:00+09:00",
  "steps": [
    { "description": "남서울대학교 정문에서 도보 이동", "durationMinutes": 5, "distanceMeters": 350 },
    { "description": "두정역(1호선) 승차", "durationMinutes": 18, "distanceMeters": 12100 },
    { "description": "천안역에서 400번 버스 환승", "durationMinutes": 12, "distanceMeters": 5400 },
    { "description": "종합운동장 정류장 하차 후 도보 이동", "durationMinutes": 7, "distanceMeters": 550 }
  ]
}
```
> 위치 정보 권한이 없거나 경로를 찾을 수 없는 경우 `404 ROUTE_NOT_FOUND` 또는 `400 ORIGIN_REQUIRED`로 응답 (11절 공통 에러 포맷 참고).

---

## 3. 리뷰 (Review)

### 3.0 리뷰 사진 업로드
`POST /api/v1/uploads/images`

`03_행사상세`/`03b_행사상세_리뷰없음`의 리뷰 작성 폼에 있는 카메라 아이콘(사진 첨부)에서 사용. 이 엔드포인트는 화면에 이미 노출된 기능이므로 MVP 범위에 포함한다 — 3.2절 리뷰 작성 시 반환된 `imageUrl`을 그대로 전달한다.

**인증 필요**: 예

**Request Body**: `multipart/form-data`, 필드명 `file` (이미지 1개)

**Response 예시**
```json
{ "imageUrl": "https://.../uploads/reviews/3f9c.jpg" }
```
> 저장소는 로컬 디스크 또는 S3 호환 오브젝트 스토리지 중 배포 환경에 맞게 선택 (`cheonan_on_architecture_spec.md` 2절 참고). 파일 크기/확장자 제한은 구현 시 정의.

### 3.1 행사 리뷰 목록 조회
`GET /api/v1/events/{eventId}/reviews`

`03_행사상세`에서 사용.

**인증 필요**: 아니오

**Query Parameter**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| page | integer | N | 페이지 번호 (기본 1) |
| size | integer | N | 페이지당 개수 (기본 20) |

**Response 예시**
```json
{
  "eventId": 1,
  "averageRating": 4.7,
  "reviewCount": 128,
  "reviews": [
    {
      "id": 501,
      "userId": 12,
      "nickname": "김서연",
      "rating": 5,
      "content": "가족과 함께 다녀왔는데 아이들이 체험 프로그램을 정말 좋아했어요.",
      "imageUrl": null,
      "createdAt": "2025-09-14T10:00:00+09:00"
    }
  ]
}
```

### 3.2 리뷰 작성
`POST /api/v1/events/{eventId}/reviews`

`03_행사상세`의 리뷰 작성 폼에서 사용. 행사당 사용자 1건만 작성 가능.

**인증 필요**: 예

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| rating | integer | Y | 별점 (1~5) |
| content | string | Y | 리뷰 내용 |
| imageUrl | string | N | 첨부 사진 URL (3.0절 업로드 API로 먼저 업로드 후 URL 전달) |

**Response 예시**
```json
{ "id": 501, "eventId": 1, "rating": 5, "content": "가족과 함께 다녀왔는데...", "createdAt": "2025-09-14T10:00:00+09:00" }
```

### 3.3 내 리뷰 목록 조회
`GET /api/v1/users/me/reviews`

`05b_마이페이지_내가쓴리뷰`에서 사용.

**인증 필요**: 예

**Response 예시**
```json
{
  "reviews": [
    {
      "id": 501,
      "event": { "id": 1, "title": "2025 천안흥타령춤축제", "imageUrl": "https://.../thumb.jpg" },
      "rating": 5,
      "content": "가족과 함께 다녀왔는데...",
      "createdAt": "2025-09-14T10:00:00+09:00"
    }
  ]
}
```

### 3.4 리뷰 삭제
`DELETE /api/v1/reviews/{reviewId}`

`05b_마이페이지_내가쓴리뷰`의 삭제(X) 아이콘에서 사용. 본인이 작성한 리뷰만 삭제 가능 (아니면 403).

**인증 필요**: 예

**Response 예시**
```json
{ "message": "리뷰가 삭제되었습니다." }
```

---

## 4. 북마크 (Bookmark)

### 4.1 내 북마크 목록 조회
`GET /api/v1/bookmarks`

`05_마이페이지`(내 북마크 탭)에서 사용. 0건이면 프론트는 `05c_마이페이지_북마크없음`으로 렌더링한다.

**인증 필요**: 예

**Response 예시**
```json
{
  "totalCount": 6,
  "events": [
    { "id": 1, "title": "2025 천안흥타령춤축제", "imageUrl": "https://.../thumb.jpg", "startDate": "2025-10-11", "endDate": "2025-10-13", "priceType": "FREE" }
  ]
}
```

### 4.2 북마크 등록
`POST /api/v1/bookmarks`

`01_홈`/`02_행사목록`/`02c_행사목록_지도뷰`/`03_행사상세`의 하트 아이콘에서 사용.

**인증 필요**: 예

**Request Body**
| 필드 | 타입 | 필수 | 설명 |
|---|---|---|---|
| eventId | integer | Y | 북마크할 행사 ID |

**Response 예시**
```json
{ "eventId": 1, "isBookmarked": true }
```

### 4.3 북마크 삭제
`DELETE /api/v1/bookmarks/{eventId}`

**인증 필요**: 예

**Response 예시**
```json
{ "eventId": 1, "isBookmarked": false }
```

---

## 5. 마이페이지 (My Page)

### 5.1 내 정보 조회
`GET /api/v1/users/me`

`05_마이페이지` 상단 프로필 영역에서 사용.

**인증 필요**: 예

**Response 예시**
```json
{
  "id": 12,
  "nickname": "안민영",
  "email": "dkslasdud@gmail.com",
  "profileImageUrl": "https://.../avatar.png"
}
```

> 내가 쓴 리뷰 목록은 3.3절(`GET /users/me/reviews`) 참고.

---

## 6. 공통 에러 응답 포맷

```json
{
  "error": {
    "code": "EVENT_NOT_FOUND",
    "message": "존재하지 않는 행사입니다."
  }
}
```

| HTTP Status | 사용 예 |
|---|---|
| 400 | 잘못된 요청 파라미터 (예: `directions` 호출 시 `originLat`/`originLng` 누락) |
| 401 | 토큰 없음/만료 |
| 403 | 권한 없음 (본인이 작성하지 않은 리뷰 삭제 시도 등) |
| 404 | 리소스 없음 (행사/리뷰 등), 또는 경로를 찾을 수 없음(`ROUTE_NOT_FOUND`) |
| 409 | 중복 리뷰 작성 시도 (행사당 1건 제한) |
| 500 | 서버 내부 오류 |

---

## 7. 추후 고도화 시 고려
- 행사 데이터를 공공 API(TourAPI, 공공데이터포털)에서 수집하는 배치/동기화 엔드포인트 및 관리자용 행사 등록/수정/삭제 API
- 리뷰 좋아요/신고 API
- 커뮤니티 게시글/댓글 API
- 북마크 행사 D-day 알림 발송 API 및 알림 설정 API
- 페이지네이션 이후 단계(무한 스크롤 커서 방식 등)로의 전환
