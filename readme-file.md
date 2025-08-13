# 📊 CEO Dashboard - 대표님 통합 대시보드

실시간으로 대표님과 비서 간의 소통을 지원하는 통합 대시보드 시스템입니다.  
슬랙과 연동되어 긴급 요청, 서명 요청, 일정 관리 등을 효율적으로 처리할 수 있습니다.

## ✨ 주요 기능

### 1. 실시간 소통
- **양방향 메시징**: 대표님 ↔ 비서 간 실시간 DM
- **긴급 요청**: 우선순위가 높은 긴급 사항 즉시 전달
- **서명 요청**: 결재가 필요한 문서 알림
- **일정 조율**: 미팅 예약 및 일정 변경 요청

### 2. 미팅 관리
- 슬랙 #미팅-현황 채널과 자동 동기화
- 오늘의 미팅 일정 실시간 표시
- 미팅 상태 추적 (예정/진행중/완료)

### 3. 파일 공유
- 드래그 앤 드롭으로 간편한 파일 업로드
- Google Drive 자동 저장
- 파일 링크 자동 생성 및 공유

### 4. 요청 추적
- 모든 요청의 상태 추적 (대기/처리중/완료)
- 요청 히스토리 관리
- 우선순위별 분류

## 🚀 시작하기

### 사전 요구사항

1. **Google Workspace 계정**
   - Google Sheets
   - Google Drive
   - Google Apps Script

2. **Slack Workspace**
   - Bot Token (xoxb-로 시작)
   - CEO의 Slack User ID
   - 미팅 채널 ID (선택사항)

### 설치 과정

#### 1단계: Google Sheets 설정

1. 새 Google Sheets 문서 생성
2. 다음 시트들을 추가:
   - `meetings` - 미팅 정보
   - `requests` - 요청 사항
   - `files` - 파일 목록
   
3. 각 시트의 헤더 설정:

**meetings 시트:**
```
date | start_time | location | title | attendees | link | status
```

**requests 시트:**
```
timestamp | type | from | message | priority | status | assigned_to | notes
```

**files 시트:**
```
uploaded_at | filename | file_id | drive_share_url | uploaded_by | related_request_id
```

#### 2단계: Google Apps Script 설정

1. Google Sheets에서 `확장 프로그램` → `Apps Script` 클릭
2. `backend/gas-backend.js` 내용을 복사하여 붙여넣기
3. 스크립트 속성 설정 (`프로젝트 설정` → `스크립트 속성`):

| 속성 이름 | 설명 | 예시 |
|----------|------|------|
| `SHEET_ID` | Google Sheets ID | `1abc...xyz` |
| `DRIVE_FOLDER_ID` | 파일 저장용 Drive 폴더 ID | `1def...uvw` |
| `TOKEN` | 보안 토큰 (임의 생성) | `my-secret-token-2024` |
| `SLACK_BOT_TOKEN` | Slack Bot Token | `xoxb-1234...` |
| `CEO_SLACK_ID` | CEO의 Slack User ID | `U1234567890` |
| `MEETING_CHANNEL_ID` | 미팅 채널 ID (선택) | `C1234567890` |
| `TZ` | 타임존 | `Asia/Seoul` |
| `ALLOW_ORIGIN` | CORS 허용 도메인 | `*` 또는 특정 도메인 |

4. 웹 앱으로 배포:
   - `배포` → `새 배포`
   - 유형: `웹 앱`
   - 실행: `나`
   - 액세스: `모든 사용자`
   - 배포 후 URL 복사

#### 3단계: 대시보드 설정

1. `index.html` 파일 수정:
```javascript
const CONFIG = {
    GAS_URL: 'https://script.google.com/macros/s/YOUR_SCRIPT_ID/exec', // 여기에 배포 URL 입력
    TOKEN: 'my-secret-token-2024', // Apps Script와 동일한 토큰
    // ...
};
```

2. 웹 서버에 호스팅 또는 GitHub Pages 사용

### Slack Bot 설정

1. [api.slack.com](https://api.slack.com) 접속
2. 새 앱 생성 또는 기존 앱 사용
3. OAuth & Permissions에서 필요한 권한 추가:
   - `chat:write`
   - `conversations:history`
   - `conversations:read`
   - `im:history`
   - `im:read`
   - `im:write`
   - `users:read`

4. Bot Token 복사 (xoxb-로 시작)

## 📖 사용 방법

### 대표님 모드
1. 우측 상단에서 "대표님 모드" 선택
2. 비서에게 전달할 메시지 입력
3. 요청 타입 선택 (긴급/예약)
4. 전송 버튼 클릭

### 비서 모드
1. 우측 상단에서 "비서 모드" 선택
2. 대표님께 전달할 메시지 입력
3. 요청 타입 선택 (긴급/서명요청/예약)
4. 전송 버튼 클릭

### 미팅 정보 동기화
- 자동: 1분마다 슬랙 채널에서 자동 동기화
- 수동: 브라우저 새로고침

## 🔧 문제 해결

### 일반적인 문제

**Q: 슬랙 메시지가 전송되지 않아요**
- Slack Bot Token이 올바른지 확인
- CEO_SLACK_ID가 정확한지 확인
- Bot이 해당 사용자에게 DM을 보낼 권한이 있는지 확인

**Q: 미팅 정보가 표시되지 않아요**
- MEETING_CHANNEL_ID 설정 확인
- Bot이 해당 채널에 접근 권한이 있는지 확인
- 채널에 Bot을 초대했는지 확인

**Q: 파일 업로드가 실패해요**
- DRIVE_FOLDER_ID가 올바른지 확인
- 해당 폴더에 대한 쓰기 권한이 있는지 확인
- 파일 크기가 제한을 초과하지 않는지 확인 (10MB 이하 권장)

## 📁 프로젝트 구조

```
executive-dashboard/
├── README.md           # 프로젝트 설명서
├── index.html          # 대시보드 UI
├── backend/
│   └── gas-backend.js  # Google Apps Script 백엔드
├── docs/
│   ├── setup-guide.md  # 상세 설정 가이드
│   └── api-docs.md     # API 문서
└── .gitignore          # Git 제외 파일
```

## 🔒 보안 고려사항

1. **토큰 관리**
   - 절대 토큰을 코드에 하드코딩하지 마세요
   - 환경 변수나 보안 저장소 사용
   - 정기적으로 토큰 교체

2. **접근 제어**
   - ALLOW_ORIGIN을 특정 도메인으로 제한
   - Google Apps Script 실행 권한 최소화
   - 민감한 데이터는 암호화

3. **데이터 보호**
   - HTTPS 사용 필수
   - 개인정보는 최소한만 수집
   - 정기적인 데이터 백업

## 📝 라이선스

MIT License

## 🤝 기여하기

1. Fork the repository
2. Create your feature branch (`git checkout -b feature/AmazingFeature`)
3. Commit your changes (`git commit -m 'Add some AmazingFeature'`)
4. Push to the branch (`git push origin feature/AmazingFeature`)
5. Open a Pull Request

## 📞 지원

문제가 있거나 도움이 필요하시면 Issues 탭에서 문의해주세요.

## 🙏 감사의 말

이 프로젝트는 효율적인 업무 소통을 위해 만들어졌습니다.  
사용해주시는 모든 분들께 감사드립니다.

---

**Version:** 1.0.0  
**Last Updated:** 2024  
**Status:** 🟢 Active Development