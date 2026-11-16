# Firebase 설정 방법 (운영자만 수정 + 실시간 반영)

`index.html` 맨 아래쪽의 `FIREBASE_CONFIG`가 비어 있으면 사이트는 예전처럼 **로컬 모드**(각자 기기에만 저장, 누구나 수정)로 동작합니다.
아래 순서대로 설정하면 **온라인 모드**로 바뀝니다. 모두가 같은 대진표를 실시간으로 보고, 운영자로 등록된 Gmail 계정만 수정할 수 있습니다.

## 1. 프로젝트 만들기
1. https://console.firebase.google.com 에 Google 계정으로 로그인 → **프로젝트 추가** (Google 애널리틱스는 꺼도 됩니다)
2. 프로젝트 홈에서 **웹 앱 추가(</>)** → 앱 이름 입력 → 나오는 `firebaseConfig` 값을 복사해 둡니다

## 2. Google 로그인 켜기
1. **빌드 → Authentication → 시작하기 → Sign-in method → Google → 사용 설정 → 저장**
2. **Authentication → 설정 → 승인된 도메인 → 도메인 추가**: `hg0328.github.io`

## 3. Realtime Database 만들기
1. **빌드 → Realtime Database → 데이터베이스 만들기** → 위치 선택 → **잠금 모드로 시작**
2. **규칙** 탭에 아래 내용을 붙여 넣고 `대표운영자@gmail.com`을 대표 운영자 Gmail로 바꾼 뒤 **게시**합니다 (3군데)

```json
{
  "rules": {
    "bracket2026": {
      ".read": true,
      ".write": "auth != null && auth.token.email_verified == true && (auth.token.email == '대표운영자@gmail.com' || root.child('admins').child(auth.token.email.replace('.', ',')).val() == true)"
    },
    "admins": {
      ".read": "auth != null && auth.token.email == '대표운영자@gmail.com'",
      ".write": "auth != null && auth.token.email_verified == true && auth.token.email == '대표운영자@gmail.com'",
      "$key": {
        ".read": "auth != null && auth.token.email.replace('.', ',') == $key"
      }
    }
  }
}
```

> 실제 보안은 이 규칙이 지킵니다. 학생이 사이트 코드를 고쳐도 규칙에 없는 계정은 저장할 수 없습니다.

## 4. index.html에 값 넣기
`index.html`에서 아래 부분을 찾아 1번에서 복사한 값과 대표 운영자 Gmail을 넣습니다 (`databaseURL`은 Realtime Database 화면 위쪽 주소).

```js
var FIREBASE_CONFIG = {
  apiKey: "...",
  authDomain: "...firebaseapp.com",
  databaseURL: "https://...firebasedatabase.app",
  projectId: "...",
  appId: "..."
};
var OWNER_EMAIL = "대표운영자@gmail.com";
```

`apiKey` 같은 값은 원래 웹페이지에 공개되는 값이라 저장소에 올려도 됩니다.

## 5. 사용하기
- 사이트 맨 아래 **운영자 로그인** → Google 계정 선택
- 대표 운영자는 **운영자 관리**에서 다른 운영자의 Gmail을 추가하거나 삭제할 수 있습니다
- 등록되지 않은 계정으로 로그인하면 자동으로 로그아웃되고, 화면은 보기 전용으로 남습니다
- 카카오톡 안에서 열린 브라우저에서는 Google 로그인이 막힐 수 있습니다. 운영자는 크롬이나 사파리로 열어 주세요

## 참고
- 온라인 모드에서는 **공유** 버튼이 사라집니다 (주소만 보내면 모두 같은 화면을 봅니다)
- 이전에 각자 기기에 저장한 입력은 온라인 모드로 옮겨지지 않습니다. 운영자가 처음 한 번 다시 입력해야 합니다
