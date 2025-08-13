# API 문서

## 기본 정보

### Base URL
```
https://script.google.com/macros/s/[SCRIPT_ID]/exec
```

### 인증
모든 요청에 `token` 파라미터 필요:
- GET: `?token=[TOKEN]`
- POST: Body에 `{"token": "[TOKEN]", ...}`

### 응답 형식
- 성공: JSON 또는 Text
- 실패: Text 에러 메시지

---

## 엔드포인트

### 미팅 관리

#### GET /meetings/list
오늘의 미팅 목록 조회

**요청:**
```
GET [BASE_URL]?path=meetings/list&token=[TOKEN]
```

**응답:**
```json
{
  "items": [
    {
      "date": "2024-08-13",
      "start_time": "10:00",
      "location": "2층 회의실",
      "title": "주간 회의",
      "attendees": "전체",
      "link": "https://meet.google.com/...",
      "status": "scheduled"
    }
  ]
}
```

#### POST /meetings/ingest
새 미팅 추가

**요청:**
```json
POST [BASE_URL]?path=meetings/ingest
{
  "token": "[TOKEN]",
  "date": "2024-08-13",
  "start_time": "14:00",
  "location": "온라인",
  "title": "투자자 미팅",
  "attendees": "CEO, CFO",
  "link": "https://zoom.us/...",
  "status": "scheduled"
}
```

**응답:**
```
OK
```

#### GET /meetings/pull
슬랙 채널에서 미팅 정보 동기화

**요청:**
```
GET [BASE_URL]?path=meetings/pull&token=[TOKEN]
```

**응답:**
```json
{
  "ok": true,
  "added": 3,
  "since": "1234567890.123456",
  "savedTs": "1234567891.123456"
}
```

---

### 요청 관리

#### POST /requests/create
새 요청 생성 및 슬랙 DM 전송

**요청:**
```json
POST [BASE_URL]?path=requests/create
{
  "token": "[TOKEN]",
  "type": "urgent",        // urgent, sign, reserve
  "from": "secretary",     // secretary, ceo
  "message": "긴급히 서명이 필요합니다",
  "priority": "high"       // high, normal, low
}
```

**응답:**
```
OK
```

#### POST /requests/update
요청 상태 업데이트

**요청:**
```json
POST [BASE_URL]?path=requests/update
{
  "token": "[TOKEN]",
  "row": 5,               // 스프레드시트 행 번호
  "status": "doing"       // open, doing, done
}
```

**응답:**
```
OK
```

#### GET /requests/count
열린 요청 수 조회

**요청:**
```
GET [BASE_URL]?path=requests/count&token=[TOKEN]
```

**응답:**
```json
{
  "open": 5
}
```

---

### 채팅 (Slack DM)

#### POST /chat/send
슬랙 DM 메시지 전송

**요청:**
```json
POST [BASE_URL]?path=chat/send
{
  "token": "[TOKEN]",
  "text": "안녕하세요, 대표님"
}
```

**응답:**
```json
{
  "ok": true,
  "ts": "1234567890.123456"
}
```

#### GET /chat/history
DM 대화 내역 조회 (최근 30개)

**요청:**
```
GET [BASE_URL]?path=chat/history&token=[TOKEN]
```

**응답:**
```json
{
  "items": [
    {
      "ts": "1234567890.123456",
      "user": "U1234567890",  // 또는 "bot"
      "text": "메시지 내용"
    }
  ]
}
```

---

### 파일 관리

#### POST /files/upload
Base64 인코딩된 파일 업로드

**요청:**
```json
POST [BASE_URL]?path=files/upload
{
  "token": "[TOKEN]",
  "filename": "contract.pdf",
  "mimeType": "application/pdf",
  "base64": "JVBERi0xLjQKJcfs...",
  "uploaded_by": "secretary",
  "related_request_id": "REQ-001"
}
```

**응답:**
```json
{
  "fileId": "1abc...xyz",
  "url": "https://drive.google.com/file/d/1abc...xyz/view"
}
```

#### GET /files/list
최근 업로드된 파일 목록 (최근 20개)

**요청:**
```
GET [BASE_URL]?path=files/list&token=[TOKEN]
```

**응답:**
```json
{
  "items": [
    {
      "uploaded_at": "2024-08-13T10:00:00Z",
      "filename": "report.pdf",
      "url": "https://drive.google.com/..."
    }
  ]
}
```

---

## 에러 처리

### 에러 응답
```
Unauthorized         // 인증 실패
Bad Request         // 잘못된 요청
Unknown GET path    // 존재하지 않는 경로
GET error: [메시지]  // 처리 중 오류
POST error: [메시지] // 처리 중 오류
```

### 공통 에러 코드

| 에러 | 원인 | 해결 방법 |
|------|------|-----------|
| `Unauthorized` | 토큰 불일치 | TOKEN 확인 |
| `MEETING_CHANNEL_ID missing` | 채널 ID 미설정 | 스크립트 속성 설정 |
| `No file` | 파일 데이터 없음 | base64 데이터 확인 |
| `메시지 없음` | 빈 메시지 | message 필드 확인 |
| `파라미터 오류` | 필수 파라미터 누락 | 요청 형식 확인 |
| `DM 전송 실패` | Slack API 오류 | Bot 권한 확인 |

---

## 사용 예제

### JavaScript (Fetch API)
```javascript
// GET 요청
async function getMeetings() {
  const response = await fetch(
    `${CONFIG.GAS_URL}?path=meetings/list&token=${CONFIG.TOKEN}`
  );
  const data = await response.json();
  console.log(data.items);
}

// POST 요청
async function createRequest() {
  const response = await fetch(`${CONFIG.GAS_URL}?path=requests/create`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      token: CONFIG.TOKEN,
      type: 'urgent',
      from: 'secretary',
      message: '긴급 승인 요청',
      priority: 'high'
    })
  });
  const result = await response.text();
  console.log(result); // "OK"
}
```

### cURL
```bash
# GET 요청
curl "https://script.google.com/macros/s/[SCRIPT_ID]/exec?path=meetings/list&token=[TOKEN]"

# POST 요청
curl -X POST "https://script.google.com/macros/s/[SCRIPT_ID]/exec?path=requests/create" \
  -H "Content-Type: application/json" \
  -d '{
    "token": "[TOKEN]",
    "type": "urgent",
    "from": "secretary",
    "message": "테스트 메시지"
  }'
```

---

## 제한 사항

### Google Apps Script 제한
- 실행 시간: 6분
- URL Fetch 크기: 50MB
- 일일 실행 시간: 6시간
- 동시 실행: 30개

### Slack API 제한
- Rate Limit: Tier 2 (1분당 20회)
- 메시지 길이: 40,000자
- 파일 크기: 1GB (Free plan: 5GB 총 용량)

### 권장 사항
- 파일 업로드: 10MB 이하
- 요청 빈도: 10초당 1회 이하
- 대화 내역: 30개씩 페이징

---

## 디버깅

### Apps Script 로그 확인
1. Apps Script 편집기 열기
2. `실행` → `실행 기록`
3. 실행 선택 → 로그 보기

### 테스트 함수
```javascript
// Apps Script에서 직접 실행
function TEST_sendDM() {
  console.log('테스트 시작');
  sendSlackDM_(CONF.CEO_SLACK_ID, '테스트 메시지');
  console.log('테스트 완료');
}

function TEST_parseMeetingMessage() {
  const sample = "2024-08-13 10:00 회의실 | 주간회의 | 참석자: 전체";
  const result = parseMeetingLine_(sample, 'Asia/Seoul');
  console.log(result);
}
```

---

## 보안 고려사항

1. **토큰 관리**
   - 환경별 다른 토큰 사용
   - 정기적 토큰 교체
   - 토큰 노출 시 즉시 변경

2. **CORS 설정**
   - 프로덕션: 특정 도메인만 허용
   - `ALLOW_ORIGIN: 'https://yourdomain.com'`

3. **데이터 검증**
   - 입력값 검증
   - SQL Injection 방지 (해당 없음)
   - XSS 방지 (클라이언트 측)

4. **접근 제어**
   - Google Apps Script 실행 권한 제한
   - Slack Bot 권한 최소화
   - Drive 폴더 공유 설정 확인

---

**API 관련 문의는 Issues에서 받습니다.**