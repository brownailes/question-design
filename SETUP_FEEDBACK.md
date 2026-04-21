# 개선 요청 → Google Sheet 연동 설정

## 1. Apps Script 코드 붙여넣기

1. [Google Sheet 열기](https://docs.google.com/spreadsheets/d/17MSca2ywty8JLxYxOp0z6w3Tflx2BDamPokKJrIGXGg/)
2. 상단 메뉴 **확장 프로그램 → Apps Script** 클릭
3. 기존 코드 전체 삭제 후 아래 코드 붙여넣기:

```javascript
function doPost(e) {
  try {
    var params = e.postData.contents;
    var parsed = {};
    if (params) {
      params.split('&').forEach(function(pair) {
        var parts = pair.split('=');
        parsed[decodeURIComponent(parts[0])] = decodeURIComponent((parts[1] || '').replace(/\+/g, ' '));
      });
    }

    var ss = SpreadsheetApp.openById('17MSca2ywty8JLxYxOp0z6w3Tflx2BDamPokKJrIGXGg');
    var now = new Date();

    if (parsed['type'] === 'log') {
      // 1. 수업 디자인 생성 로그 처리 (Logs 탭이 없으면 생성)
      var logSheet = ss.getSheetByName('Logs') || ss.insertSheet('Logs');
      logSheet.appendRow([
        now,
        parsed['grade'] || '',
        parsed['subject'] || '',
        parsed['topic'] || ''
      ]);
    } else {
      // 2. 기존 사용자 피드백 의견 처리 (기본 시트 또는 Feedback 탭)
      var feedbackSheet = ss.getSheetByName('Feedback') || ss.getSheets()[0];
      feedbackSheet.appendRow([
        now,
        parsed['message'] || '',
        parsed['timestamp'] || ''
      ]);
    }

    return ContentService
      .createTextOutput(JSON.stringify({ success: true }))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(err) {
    return ContentService
      .createTextOutput(JSON.stringify({ success: false, error: err.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function doGet(e) {
  return ContentService
    .createTextOutput(JSON.stringify({ status: 'ok' }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

4. **저장** (Ctrl+S)

## 2. 웹 앱으로 배포

1. Apps Script 우측 상단 **배포 → 새 배포** 클릭
2. 유형: **웹 앱** 선택
3. 설정:
   - 설명: `피드백 수신`
   - 다음 사용자로 실행: **나 (your email)**
   - 액세스 권한: **모든 사용자**
4. **배포** 클릭 → 권한 허용
5. **웹 앱 URL** 복사 (형식: `https://script.google.com/macros/s/…/exec`)

## 3. index.html에 URL 입력

`index.html` 파일에서 아래 줄을 찾아:

```javascript
const APPS_SCRIPT_URL = 'YOUR_APPS_SCRIPT_URL';
```

복사한 URL로 교체:

```javascript
const APPS_SCRIPT_URL = 'https://script.google.com/macros/s/여기에붙여넣기/exec';
```

저장 후 `git push` 하면 완료.
