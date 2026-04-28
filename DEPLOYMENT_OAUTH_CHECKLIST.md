# Google OAuth / GitHub Pages 점검 체크리스트

## 1) 403 access_denied 원인
Google 로그인 팝업의 `access_denied`는 대부분 아래 중 하나입니다.

1. **OAuth 동의화면이 Testing 상태**이고, 로그인한 계정이 테스트 사용자에 없음
2. OAuth 클라이언트의 **Authorized JavaScript origins**에 `https://dp.chichiboo.link` 미등록
3. OAuth 동의화면의 **Authorized domains**에 `chichiboo.link` 미등록
4. Google Drive API 또는 Google Picker API 미활성화
5. API Key HTTP referrer 제한값에 `dp.chichiboo.link` 누락

## 2) Google Cloud Console 권장 설정
- OAuth Client type: **Web application**
- Authorized JavaScript origins:
  - `https://dp.chichiboo.link`
  - `https://<github-username>.github.io` (백업/테스트용)
- OAuth 동의화면:
  - Publishing status: 가능하면 **In production**
  - Testing 상태라면 Test users에 실제 로그인 계정 추가
  - Authorized domain에 `chichiboo.link` 추가
- API & Services:
  - Google Drive API 활성화
  - Google Picker API 활성화

## 3) GitHub Secrets
Repository → Settings → Secrets and variables → Actions
- `GOOGLE_CLIENT_ID`
- `GOOGLE_API_KEY`

워크플로우는 시크릿이 비어있으면 실패하도록 구성되어 있습니다.
