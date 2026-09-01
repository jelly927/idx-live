# IDX Live 를 우리 사이트로 올리기 (서버 없이 · 무료 · 15분)

두 조각이면 끝난다.
- **GitHub Pages + Actions** — index.html 호스팅 + 5분마다 fetch_data.py 를 돌려 data.json 갱신 (IDX·BI·뉴스·캘린더·외국인)
- **Cloudflare Worker** — 지수 실시간 틱용 전용 프록시 (브라우저 → Worker → Yahoo). 공용 프록시 의존 제거

## 1) GitHub (data + 호스팅)
1. github.com → New repository → 이름 `idx-live` (Private 가능. Private 는 GitHub Pro/Team 에서 Pages 지원)
2. 이 폴더 전체 업로드 (웹 UI "Add file → Upload files" 로 드래그해도 됨)
3. Settings → Pages → Source: **GitHub Actions**
4. Actions 탭 → `update-data` → **Run workflow** (첫 실행 2~3분)
5. 주소: `https://<계정>.github.io/idx-live/` — 이후 평일 08~18시 WIB 5분마다 자동 갱신

## 2) Cloudflare Worker (실시간 틱)
1. dash.cloudflare.com 가입 → Workers & Pages → Create Worker → 이름 `idx-live`
2. Edit code → `worker/worker.js` 내용 붙여넣기 → Deploy
3. 주소 복사 (`https://idx-live.<계정>.workers.dev`) → `index.html` 상단 `const LIVE_PROXY=''` 에 넣기 → GitHub 에 다시 올리기

## 3) 회사 도메인
- GitHub Pages: Settings → Pages → Custom domain 에 `idxlive.kisi.co.id` 입력 → DNS 에 CNAME `<계정>.github.io`
- Cloudflare 에도 같은 도메인 걸면 Worker 를 `idxlive.kisi.co.id/api/*` 로 붙일 수 있음 (선택)

## 확인
- 화면 우상단 상태가 `실시간 · 15분 지연 · Yahoo` 면 틱 정상, `Worker 미설정` 이면 2) 단계 미완
- Actions 로그에 `data.json | JCI …` 줄이 찍히면 수집 정상. `rss empty` 는 config.json 의 RSS 주소 수정
- IDX 403 → Actions 러너 IP 가 차단된 경우. `cloudscraper` 가 자동 우회하고, 그래도 막히면 Worker 를 IDX 프록시로 써서 우회 (worker.js 는 idx.co.id 허용돼 있음)

## 사내 서버가 있으면 (더 단순)
```
git clone … && pip install -r requirements.txt && playwright install chromium
python fetch_data.py --loop 300 &        # 수집
python -m http.server 80                 # 호스팅 (또는 nginx 로 폴더 서빙)
```
사내망이면 Worker 없이도 됨 — 서버가 같은 오리진에서 `/live` 엔드포인트로 Yahoo 를 중계하도록 아래 한 줄짜리 프록시를 추가하면 된다 (`worker/worker.js` 와 동일 로직, Flask 8줄).
