# 3-1팀 퇴근보고 — Firebase 연결 순서

## 1. 콘솔 설정 (10분)
1. Firebase 콘솔 → 기존 프로젝트(cash-eabc1) 선택
2. **Firestore Database** → 데이터베이스 만들기
   - 데이터베이스 ID: **`(default)`** 권장 (다른 이름을 쓰면 index.html 의 `DATABASE_ID` 에 그 이름을 적어야 한다)
   - 위치: **`asia-northeast3 (서울)`** — 생성 후 변경 불가
   - **프로덕션 모드에서 시작** (테스트 모드는 30일 뒤 잠기고 그 사이 누구나 접근 가능)
   - 생성 직후 기본 규칙은 `allow read, write: if false` = 전부 거부다.
     **아래 3번 규칙 게시를 먼저 끝내지 않으면 앱이 "연결 오류"로만 표시된다.**
3. **Authentication** → 시작하기 → 로그인 방법에서 **이메일/비밀번호** 사용 설정
4. Authentication → Users → 멘토 5명 + 팀장 계정 추가 (이메일 / 임시 비밀번호)
5. **프로젝트 설정 → 내 앱 → 웹 앱** 추가 → SDK 구성 값 복사

## 2. 설정값 붙여넣기
`index.html` 최상단 `FIREBASE_CONFIG` 에 복사한 6개 값을 채운다.
apiKey 와 projectId 두 개만 있어도 클라우드 모드로 켜진다.

## 3. 보안 규칙 적용
Firestore → 규칙 탭에 `firestore.rules` 내용을 붙여넣고 게시.
`leader@example.com` 을 팀장 실제 계정으로 바꾼다.
(멘토별 쓰기 제한이 불필요하면 reports 블록의 write 조건을 `request.auth != null` 로 단순화)

## 4. 배포
GitHub 저장소에 index.html 을 올리고 Pages 활성화.
Firebase 콘솔 → Authentication → 설정 → **승인된 도메인**에
`<사용자명>.github.io` 추가 (없으면 로그인이 거부된다).

## 데이터 구조
```
team31/main/reports/{YYYY-MM-DD_멘토명}   보고 1건 = 문서 1개
team31/main/events/{자동ID}                상담·면담 일정
team31/main/config/team                    멘토 목록 · 직함 · 월별 영업일
```
서체/글자크기 등 화면 설정은 개인 취향이므로 각 기기 localStorage 에만 저장된다.

## 동작 방식
- 좌측 하단 배지: 동기화됨 / 동기화 중 / 연결 오류 / 로그인 필요 / 로컬 저장
- 저장 시 **변경된 문서만** 전송 → 멘토 5명이 동시에 저장해도 서로 덮어쓰지 않음
- onSnapshot 실시간 수신 → 멘토가 저장하면 팀장 화면이 새로고침 없이 갱신
- 오프라인 캐시(IndexedDB) → 인터넷이 끊겨도 입력 가능, 복구 시 자동 전송
- 입력 중일 때는 화면 갱신을 미뤄 타이핑이 날아가지 않게 처리
