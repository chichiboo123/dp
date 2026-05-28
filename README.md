# 디지털 페이퍼 (Digital Paper)

Google Drive의 PDF 학습지를 불러와 직접 필기하고, 완성된 결과물을 내보낼 수 있는 교육용 웹 앱입니다.

---

## 주요 기능

- **PDF 불러오기** — Google Drive에서 PDF 파일을 선택하거나 URL로 직접 로드
- **자유 필기** — 펜 도구로 색상·굵기 조절, 지우개 도구 지원
- **멀티 페이지** — 페이지별 독립 필기 레이어 관리
- **실행 취소/다시 실행** — 페이지당 최대 60단계 히스토리
- **확대/축소** — 50~400% 줌, 너비 맞춤, 모바일 핀치 제스처 지원
- **내보내기** — 현재 페이지 JPG 또는 전체 페이지 PDF로 저장
- **공유 링크** — URL 파라미터로 특정 파일·페이지·제출처 지정
- **제출 연동** — Padlet·Canva·일반 iframe 삽입 지원
- **다국어** — 한국어·영어·일본어 자동 감지 및 수동 전환
- **서버리스** — 모든 데이터는 브라우저에만 존재하며 서버에 저장되지 않음

---

## 기술 스택

| 분류 | 라이브러리·서비스 |
|------|-----------------|
| UI | React 18, Tailwind CSS |
| PDF 렌더링 | pdf.js v3.11.174 |
| PDF 생성 | jsPDF v2.5.1 |
| 인증·파일 선택 | Google Identity Services, Google Picker API |
| 배포 | GitHub Pages + GitHub Actions |

> 빌드 도구 없이 단일 `index.html`로 동작합니다. 모든 의존성은 CDN에서 로드됩니다.

---

## 구글 인증(Google OAuth 2.0) 안전성

### 인증 방식

이 앱은 Google Identity Services(GIS)의 **암묵적 토큰 방식(Implicit Token Flow)**을 사용합니다. 사용자가 로그인하면 Google이 단기 액세스 토큰을 발급하며, 이 토큰은 브라우저의 `localStorage`에만 저장됩니다.

### 권한 범위 최소화

요청 권한 범위는 **`drive.readonly`(읽기 전용)** 하나뿐입니다. 앱은 사용자의 Google Drive 파일을 생성·수정·삭제할 수 없으며, 오직 선택한 PDF를 읽어오는 것만 가능합니다.

### 토큰 관리

| 항목 | 내용 |
|------|------|
| 저장 위치 | 브라우저 `localStorage` (서버 전송 없음) |
| 유효 기간 | 발급 후 약 1시간 자동 만료 |
| 자동 삭제 | 토큰 만료 시 `localStorage`에서 즉시 제거 |
| 수동 삭제 | 로그아웃 버튼 클릭 시 `google.accounts.oauth2.revoke()` 호출로 즉시 무효화 |
| 서버 저장 | 없음 — 백엔드가 토큰을 수집하거나 보관하지 않습니다 |

### 보안 주의 사항

- 공용 PC에서는 반드시 **로그아웃**하거나 브라우저 탭을 닫기 전 세션을 종료하세요.
- 브라우저 개발자 도구에서 `localStorage`를 열면 토큰이 노출될 수 있습니다. 신뢰하지 않는 환경에서는 사용을 삼가세요.
- 앱은 Google Cloud Console에 등록된 **특정 도메인**에서만 동작하도록 제한되어 있습니다.

---

## 로컬 실행

### 사전 준비

[Google Cloud Console](https://console.cloud.google.com/)에서 다음을 설정해야 합니다.

1. **Google Drive API** 및 **Google Picker API** 활성화
2. **OAuth 2.0 클라이언트 ID** 생성 (유형: 웹 애플리케이션)
   - 승인된 JavaScript 원본에 `http://localhost:8000` 추가
3. **API 키** 생성 (HTTP 리퍼러 제한 권장)

### 실행

```bash
# 저장소 클론
git clone https://github.com/chichiboo123/dp.git
cd dp

# config.js에 인증 정보 입력
# CLIENT_ID와 API_KEY를 발급받은 값으로 교체
cat > config.js << 'EOF'
window.APP_CONFIG = {
  CLIENT_ID: "YOUR_CLIENT_ID.apps.googleusercontent.com",
  API_KEY: "YOUR_API_KEY",
};
EOF

# 정적 파일 서버 실행 (예시)
python -m http.server 8000
# 또는: npx http-server
```

브라우저에서 `http://localhost:8000`으로 접속합니다.

---

## GitHub Pages 배포

1. 저장소를 Fork합니다.
2. **Settings → Secrets and variables → Actions**에서 아래 시크릿을 추가합니다.

   | 시크릿 이름 | 값 |
   |------------|-----|
   | `GOOGLE_CLIENT_ID` | OAuth 2.0 클라이언트 ID |
   | `GOOGLE_API_KEY` | API 키 |

3. `main` 브랜치에 푸시하면 GitHub Actions가 자동으로 빌드·배포합니다.
4. 배포 후 Google Cloud Console에서 승인된 JavaScript 원본에 배포 URL을 추가합니다.

> OAuth 설정 트러블슈팅은 [`DEPLOYMENT_OAUTH_CHECKLIST.md`](./DEPLOYMENT_OAUTH_CHECKLIST.md)를 참고하세요.

---

## 사용 방법

### 교사 (학습지 공유)

1. 구글 로그인 후 Google Drive에서 PDF 파일 선택
2. 필요한 경우 펜으로 예시 필기 추가
3. **제출 링크 설정**에서 Padlet·Canva 등 제출처 URL 입력
4. 상단의 공유 링크를 복사해 학생에게 배포

### 학생 (필기 및 제출)

1. 교사가 공유한 링크를 열면 로그인 없이 PDF가 바로 표시됨
2. 펜 도구로 필기 후 JPG 또는 PDF로 내보내기
3. 내보낸 파일을 Padlet·Canva 등에 업로드하여 제출

---

## 브라우저 지원

Canvas API, Fetch API, localStorage를 지원하는 최신 브라우저 권장합니다.

- Chrome / Edge 90+
- Firefox 90+
- Safari 14+
- 모바일 Chrome / Safari (터치·핀치 줌 지원)

---

## 개발자 정보

| 항목 | 내용 |
|------|------|
| 개발자 | chichiboo123 |
| 이메일 | chichiboo@kakao.com |
| GitHub | [github.com/chichiboo123](https://github.com/chichiboo123) |
| 저장소 | [github.com/chichiboo123/dp](https://github.com/chichiboo123/dp) |

버그 리포트나 기능 제안은 [Issues](https://github.com/chichiboo123/dp/issues)에 남겨주세요.

---

## 라이선스

이 프로젝트는 별도의 라이선스가 명시되지 않았습니다. 문의는 위 이메일로 연락주세요.
