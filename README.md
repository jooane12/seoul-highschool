# 서울 고등학교 길잡이

서울 고등학교를 검색하고, 학교 정보와 집에서 학교까지의 네이버지도·카카오맵 길찾기 링크를 보여주는 정적 웹페이지입니다.
서버와 인증키가 필요 없습니다.

## 구성
| 파일 | 설명 |
|---|---|
| `index.html` | 화면 전체 (HTML+CSS+JS 한 파일) |
| `schools.json` | 서울 고등학교 목록·좌표 (아래 1번 스크립트로 생성) |
| `tools/build_schools.py` | NEIS에서 학교 목록을 받아 `schools.json` 생성 |
| `optional/` | 나중에 자동차 소요 시간 등을 정확히 계산할 때 쓸 Cloudflare Worker (지금은 사용 안 함) |

## 1. 학교 목록 만들기 (처음 한 번, 학교 정보를 갱신하고 싶을 때 다시)
```
python tools/build_schools.py
```
- NEIS 인증키 없이 동작합니다. 학교 위치 좌표 계산 때문에 7~10분쯤 걸립니다.
- 중간에 멈춰도 다시 실행하면 `tools/geocode_cache.json` 덕분에 이어서 진행합니다.

### 1-1. 학급·학생 수 합치기 (학교알리미 인증키 필요)
```
$env:SCHOOLINFO_KEY = '발급받은키'
python tools/build_stats.py
```
이미 받은 원본(`tools/raw`)으로 키 없이 다시 합치려면 `python tools/build_stats.py --offline`

### 1-2. 학교 유형 구분 (일반고/자사고/특목고/특성화고)
```
python tools/classify.py
```
- NEIS의 고교 구분은 자사고와 자율형 공립고가 섞여 있고 일반고로 전환한 학교도 갱신이 늦어, `tools/classify.py`의 명단으로 보정합니다.
- **해마다 모집요강이 나오면** `classify.py` 위쪽의 `JASAGO`(자사고 명단)와 `CONVERTED`(전환 학교)를 갱신하세요. 근거 링크는 파일 맨 위에 있습니다.
- 실행 순서: `build_schools.py` → `build_stats.py` → `classify.py`

## 2. 내 컴퓨터에서 확인
```
python -m http.server 8000
```
브라우저에서 http://localhost:8000 접속. (`index.html`을 더블클릭하면 `schools.json`을 못 읽습니다.)

## 3. GitHub Pages로 공유
git 프로그램 없이 웹사이트에서 파일을 끌어다 놓는 방법입니다.

1. github.com 로그인 → 오른쪽 위 `+` → **New repository**
   - Repository name: `seoul-highschool` (주소에 들어갑니다)
   - **Public** 선택 → Create repository
2. 만들어진 화면에서 **uploading an existing file** 링크 클릭
3. 상위 폴더의 `github-upload` 폴더 **안의 파일·폴더 전부**(`index.html`, `schools.json`, `README.md`, `tools`, `optional`)를 브라우저 창에 끌어다 놓고 **Commit changes**
   - 폴더째 끌어다 놓아도 됩니다. `tools/raw`, `geocode_cache.json` 같은 작업용 파일은 올리지 마세요.
4. 저장소 **Settings → Pages** → Branch를 `main` / `(root)`로 선택 → Save
5. 1~2분 뒤 `https://내계정.github.io/seoul-highschool/` 로 접속됩니다. 이 주소를 공유하세요.

주의
- 저장소는 **공개**입니다. 비밀번호·API 키·개인정보를 올리지 마세요. (현재 버전은 키가 필요 없고, 올릴 파일에 키가 없는 것을 확인했습니다.)
- 인증키를 `$env:...` 로 입력한 터미널 기록은 파일에 남지 않지만, 채팅·문서에 붙여넣지 마세요.
- 학교 정보를 갱신하려면 이 폴더에서 1~1-2 단계를 다시 실행한 뒤, 바뀐 `schools.json`을 같은 방법으로 다시 올립니다(같은 이름으로 덮어쓰기).
