# QLD · SCHD 리밸런싱 대시보드

GitHub Actions가 1시간마다 시세·환율·배당·나스닥 PER·공포탐욕지수를 모아 `data.json`을 갱신하고, GitHub Pages가 이 페이지를 웹 주소로 띄웁니다. 보유 수량·평균단가 같은 입력값은 각자 브라우저에만 저장되므로, 저장소에는 개인 정보가 올라가지 않습니다.

## 파일 구성

| 파일 | 역할 |
|---|---|
| `index.html` | 대시보드 화면. 열 때마다 `data.json`을 읽어 그립니다 |
| `data.json` | 자동 갱신되는 데이터 (직접 고칠 필요 없음) |
| `config.json` | 자동으로 못 받는 정보(배당 지급일, 확정 일정) 수동 입력 |
| `scripts/update_data.py` | 데이터를 모으는 파이썬 스크립트 |
| `.github/workflows/update.yml` | 1시간마다 스크립트 실행 + 페이지 배포 |
| `requirements.txt` | 스크립트에 필요한 패키지 목록 |

## 설치 (처음 한 번, 약 10분)

1. **GitHub 가입**: https://github.com 에서 무료 계정을 만듭니다.
2. **저장소 만들기**: 오른쪽 위 `+` → `New repository`
   - Repository name: 예) `qld-schd-dashboard`
   - **Public** 선택 (무료 계정에서 Pages를 쓰려면 공개 저장소여야 합니다)
   - `Create repository`
3. **파일 올리기**: 새 저장소 화면의 `uploading an existing file` 링크 → 압축을 푼 폴더 안의 파일과 폴더를 **전부** 끌어다 놓기 → `Commit changes`
   - `.github` 폴더처럼 점으로 시작하는 폴더가 탐색기에서 안 보이면 숨김 파일 표시를 켜세요 (Windows: 탐색기 `보기` → `숨긴 항목`, Mac: Finder에서 `Cmd + Shift + .`).
   - 업로드 후 저장소에 `.github/workflows/update.yml`이 보이는지 확인하세요. 없으면 `Add file` → `Create new file`에서 파일 이름 칸에 `.github/workflows/update.yml`을 입력하고 내용을 붙여 넣으면 됩니다.
4. **Actions 쓰기 권한 주기**: 저장소 `Settings` → 왼쪽 `Actions` → `General` → 맨 아래 `Workflow permissions`에서 **Read and write permissions** 선택 → `Save`
5. **Pages 켜기**: `Settings` → 왼쪽 `Pages` → `Build and deployment`의 Source를 **GitHub Actions**로 선택
6. **첫 실행**: 위쪽 `Actions` 탭 → 왼쪽 `Update data & deploy` → 오른쪽 `Run workflow` → `Run workflow`
   - 2~3분 뒤 초록색 체크가 뜨면 성공입니다.
7. **주소 확인**: `Settings` → `Pages` 위쪽에 `Your site is live at https://아이디.github.io/qld-schd-dashboard/` 가 표시됩니다. 이 주소를 즐겨찾기 하세요.

이후에는 매시 7분(UTC)에 자동으로 갱신됩니다. 화면 오른쪽 위에 마지막 갱신 시각이 표시됩니다.

## 갱신되는 항목과 출처

| 항목 | 출처 | 비고 |
|---|---|---|
| QLD·SCHD 현재가, 나스닥100·SCHD 1년 일봉 | Yahoo Finance (yfinance) | 장중에는 진행 중인 일봉 포함 |
| 원/달러 환율 | Yahoo Finance `KRW=X` | 페이지를 열어 둔 상태에서도 브라우저가 1시간마다 별도로 갱신 |
| 배당 이력(배당락일·금액) | Yahoo Finance | 지급일은 `config.json` 기준, 없으면 배당락일 + 5~6일로 추정 |
| 향후 4분기 배당 일정 | 최근 4번의 배당을 52주씩 밀어 추정 | `config.json`의 확정 일정이 있으면 그것으로 교체 |
| SCHD 연도별 배당률·밴드 | Yahoo Finance 가격·배당으로 계산 | 해가 바뀌면 자동으로 한 해 추가 |
| 나스닥100 TTM·선행 PER, PER 밴드 | History of Market (CC BY 4.0) | |
| 공포탐욕지수 | CNN 데이터 | CNN이 서버 접속을 막으면 직전 값 유지 |

어떤 항목이 실패해도 나머지는 정상 갱신되고, 실패한 항목은 직전 값을 그대로 씁니다. 실패가 있으면 화면 갱신 시각 옆에 "일부 항목 이전 값 유지"가 표시되며, 원인은 `Actions` 탭의 실행 기록에서 `[FAIL]` 줄로 확인할 수 있습니다.

## 손으로 고칠 것 (config.json)

배당 금액이 **공시되었지만 아직 배당락 전**인 경우나 확정 일정이 발표되었을 때만 수정합니다. 저장소에서 `config.json` → 연필 아이콘 → 수정 → `Commit changes`.

```json
"confirmed": {
  "SCHD": [ { "ex": "2026-12-10", "pay": "2026-12-15", "amount": 0.2850 } ],
  "QLD":  [ { "ex": "2026-12-23", "pay": "2026-12-30", "amount": null } ]
}
```

- `amount`를 `null`로 두면 전년 같은 분기 금액으로 추정합니다.
- 화면의 배당 일정 표에서 금액을 직접 입력해도 되지만, 그 값은 본인 브라우저에만 저장됩니다. `config.json`에 넣으면 모든 사람 화면에 반영됩니다.
- 지급일(`pay_dates`)은 배당락 후 실제 지급일이 다르게 잡히는 경우에만 추가하면 됩니다.

## 다른 사람과 공유하기

- **화면만 쓰게 하기**: Pages 주소를 알려주면 됩니다. 각자 입력한 수량·단가는 각자 브라우저에 따로 저장되어 서로 보이지 않습니다.
- **양식 자체를 복사해 쓰게 하기**: `Settings` → `General` → **Template repository** 체크. 그러면 저장소 화면에 `Use this template` 버튼이 생기고, 다른 사람이 누르면 자기 계정에 똑같은 저장소가 만들어집니다. 그 사람은 위 설치 4~7단계만 하면 자기 주소로 자동 갱신 대시보드를 갖게 됩니다.

## 알아둘 점

- GitHub의 정기 실행은 서버가 붐비면 수 분~수십 분 늦어질 수 있습니다.
- 공개 저장소에서 60일 동안 활동이 없으면 GitHub가 정기 실행을 멈출 수 있습니다. 메일이 오면 `Actions` 탭에서 다시 켜면 됩니다.
- 매시간 `data.json` 커밋이 쌓입니다. 정상 동작이며 저장소 용량에는 큰 영향이 없습니다.
- `index.html`을 더블클릭해 열면 브라우저 보안 때문에 `data.json`을 못 읽습니다. 반드시 Pages 주소로 여세요.
- 개인 점검용 자료이며 투자 권유가 아닙니다.
