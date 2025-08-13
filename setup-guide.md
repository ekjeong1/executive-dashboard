# 상세 설정 가이드

## 📋 목차
1. [Google Workspace 설정](#google-workspace-설정)
2. [Slack Bot 설정](#slack-bot-설정)
3. [대시보드 배포](#대시보드-배포)
4. [테스트 및 검증](#테스트-및-검증)

---

## Google Workspace 설정

### 1. Google Sheets 생성

#### 스프레드시트 구조
```
[대시보드 데이터]
├── meetings (시트1)
├── requests (시트2)
└── files (시트3)
```

#### meetings 시트 설정
1행 (헤더):
- A1: `date`
- B1: `start_time`
- C1: `location`
- D1: `title`
- E1: `attendees`
- F1: `link`
- G1: `status`

#### requests 시트 설정
1행 (헤더):
- A1: `timestamp`
- B1: `type`
- C1: `from`
- D1: `message`
- E1: `priority`
- F1: `status`
- G1: `assigned_to`
- H1: `notes`

#### files 시트 설정
1행 (헤더):
- A1: `uploaded_at`
- B1: `filename`
- C1: `file_id`
- D1: `drive_share_url`
- E1: `uploaded_by`
- F1: `related_request_id`

### 2. Google Drive 폴더 생성

1. Google Drive에서 새 폴더 생성 (예: "Dashboard Files")
2. 폴더 ID 복사:
   - 폴더 열기
   - URL에서 ID 추출: `https://drive.google.com/drive/folders/[FOLDER_ID]`
3. 폴더 공유 설정:
   - 링크가 있는 모든 사용자가 보기 가능
   - 또는 특정 사용자와 공유

### 3. Google Apps Script 설정

#### 스크립트 생성
1. Google Sheets에서 `확장 프로그램` → `Apps Script`
2. 기본 코드 삭제
3. `gas-backend.js` 내용 붙여넣기

#### 스크립트 속성 설정
`프로젝트 설정` → `스크립트 속성` → `속성 추가`

필수 속성:
```
SHEET_ID = [Google Sheets ID]
DRIVE_FOLDER_ID = [Drive 폴더 ID]
TOKEN = [보안 토큰 생성]
SLACK_BOT_TOKEN = [Slack Bot Token]
CEO_SLACK_ID = [CEO Slack User ID]
TZ = Asia/Seoul
ALLOW_ORIGIN = *
```

선택 속성:
```
MEETING_CHANNEL_ID = [미팅 채널 ID]
CEO_DM_CHANNEL = [DM 채널 ID - 자동 생성됨]
```

#### 웹 앱 배포
1. `배포` → `새 배포`
2. 설정:
   - 유형: `웹 앱`
   - 설명: `CEO Dashboard Backend`
   - 실행: `나`
   - 액세스: `모든 사용자`
3. `배포` 클릭
4. 권한 승인
5. 웹 앱 URL 복사

---

## Slack Bot 설정

### 1. Slack App 생성

1. [api.slack.com/apps](https://api.slack.com/apps) 접속
2. `Create New App` → `From scratch`
3. 앱 이름: `CEO Dashboard Bot`
4. Workspace 선택

### 2. OAuth & Permissions 설정

#### Bot Token Scopes 추가:
- `chat:write` - 메시지 전송
- `conversations:history` - 대화 내역 읽기
- `conversations:read` - 채널 정보 읽기
- `im:history` - DM 내역 읽기
- `im:read` - DM 정보 읽기
- `im:write` - DM 메시지 전송
- `users:read` - 사용자 정보 읽기
- `channels:history` - 채널 내역 읽기 (미팅 채널용)
- `channels:read` - 채널 정보 읽기 (미팅 채널용)

#### Bot Token 설치
1. `Install App to Workspace` 클릭
2. 권한 승인
3. `Bot User OAuth Token` 복사 (xoxb-로 시작)

### 3. CEO Slack ID 찾기

방법 1: Slack 웹에서
1. CEO 프로필 클릭
2. `More` → `Copy member ID`

방법 2: API로
```javascript
// Apps Script 테스트 함수
function findUserId() {
  const token = 'xoxb-YOUR-TOKEN';
  const email = 'ceo@company.com';
  
  const url = `https://slack.com/api/users.lookupByEmail?email=${email}`;
  const res = UrlFetchApp.fetch(url, {
    headers: { 'Authorization': `Bearer ${token}` }
  });
  
  const data = JSON.parse(res.getContentText());
  console.log('User ID:', data.user.id);
}
```

### 4. 미팅 채널 설정 (선택사항)

1. Slack에서 #미팅-현황 채널 생성
2. Bot을 채널에 초대: `/invite @CEO Dashboard Bot`
3. 채널 ID 찾기:
   - 채널 정보 → About → Channel ID 복사
   - 또는 웹에서 URL 확인

---

## 대시보드 배포

### 옵션 1: GitHub Pages

1. GitHub 저장소에 파일 업로드
2. Settings → Pages
3. Source: `Deploy from a branch`
4. Branch: `main` / `root`
5. Save
6. URL: `https://[username].github.io/[repository]/`

### 옵션 2: 로컬 테스트

1. Python 서버:
```bash
python -m http.server 8000
```

2. Node.js 서버:
```bash
npx http-server
```

3. 브라우저에서 `http://localhost:8000` 접속

### 옵션 3: 웹 호스팅

- Netlify: 드래그 앤 드롭으로 배포
- Vercel: GitHub 연동 자동 배포
- Firebase Hosting: Google 클라우드 기반

### 설정 파일 수정

`index.html`의 CONFIG 수정:
```javascript
const CONFIG = {
    GAS_URL: 'https://script.google.com/macros/s/[SCRIPT_ID]/exec',
    TOKEN: '[YOUR_SECRET_TOKEN]',
    AUTO_REFRESH: 30000,
    MEETING_REFRESH: 60000
};
```

---

## 테스트 및 검증

### 1. Apps Script 테스트

Apps Script 편집기에서 실행:

```javascript
// DM 테스트
function TEST_sendDM() {
  sendSlackDM_(CONF.CEO_SLACK_ID, '테스트 메시지');
}

// 미팅 파싱 테스트
function TEST_parseMeetingMessage() {
  // 함수 실행 후 로그 확인
}
```

### 2. API 엔드포인트 테스트

브라우저나 Postman에서:

```
GET 요청:
[GAS_URL]?path=meetings/list&token=[TOKEN]

POST 요청:
[GAS_URL]?path=requests/create
Body: {
  "token": "[TOKEN]",
  "type": "urgent",
  "from": "secretary",
  "message": "테스트 메시지"
}
```

### 3. 대시보드 기능 테스트

#### 체크리스트:
- [ ] 로그인/인증 확인
- [ ] 실시간 시간 업데이트
- [ ] 사용자 모드 전환 (대표님/비서)
- [ ] 긴급 요청 전송
- [ ] 슬랙 DM 수신 확인
- [ ] 채팅 내역 로드
- [ ] 미팅 정보 표시
- [ ] 파일 업로드
- [ ] 요청 상태 업데이트

### 4. 트러블슈팅

#### CORS 오류
```javascript
// Apps Script에서 확인
function _headers() {
  return {
    'Access-Control-Allow-Origin': '*', // 또는 특정 도메인
    'Access-Control-Allow-Methods': 'GET,POST,OPTIONS',
    'Access-Control-Allow-Headers': 'content-type,authorization'
  };
}
```

#### 인증 실패
- TOKEN이 일치하는지 확인
- 쿼리 파라미터로 전달되는지 확인

#### Slack 연동 실패
- Bot Token이 유효한지 확인
- 필요한 권한이 모두 있는지 확인
- Bot이 채널/DM에 접근 가능한지 확인

---

## 유지보수

### 로그 모니터링
Apps Script 편집기 → 실행 → 로그 확인

### 데이터 백업
정기적으로 Google Sheets 데이터 백업:
- 파일 → 다운로드 → Excel/CSV

### 토큰 갱신
3-6개월마다 보안 토큰 변경:
1. 새 토큰 생성
2. Apps Script 속성 업데이트
3. index.html CONFIG 업데이트

---

## 추가 리소스

- [Google Apps Script 문서](https://developers.google.com/apps-script)
- [Slack API 문서](https://api.slack.com/docs)
- [Google Sheets API](https://developers.google.com/sheets/api)
- [Google Drive API](https://developers.google.com/drive/api)

---

**문제 발생 시 Issues 탭에서 문의해주세요!**