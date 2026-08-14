# 2026-SAG

2026 Scientific Advisory Group 관리 대시보드

GVF 2026 대시보드(v3.17)의 템플릿을 그대로 유지하고, 콘텐츠를 초기화한 버전입니다.
Firebase 프로젝트는 GVF와 **완전히 분리**되어 있습니다.

## 배포
- 단일 파일 정적 사이트 (`index.html`)
- GitHub Push → Vercel 자동 배포

---

## ① Firebase 프로젝트 만들기 (최초 1회)

1. [Firebase 콘솔](https://console.firebase.google.com/)에서 **프로젝트 추가** → 이름 예: `sag-2026`
   (Google Analytics는 사용 안 함으로 두어도 무방합니다)
2. **빌드 → Firestore Database → 데이터베이스 만들기**
   - 모드: 프로덕션 모드
   - 위치: `asia-northeast3 (Seoul)` 권장 — **생성 후 변경 불가**
3. **규칙** 탭에서 아래로 교체 후 **게시**:

   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /{document=**} {
         allow read, write: if true;
       }
     }
   }
   ```

   > ⚠️ 이 규칙은 URL을 아는 누구나 읽기·쓰기가 가능합니다. GVF와 동일한 방식이며,
   > 비공개 정보를 다룰 경우 Firebase Authentication 적용을 검토하세요.

4. **⚙️ 프로젝트 설정 → 일반 → 내 앱 → 웹(`</>`)** 등록 → `firebaseConfig` 복사

## ② 앱에 연결하기

`index.html` 상단 `FIREBASE_CONFIG`에 붙여넣고 커밋하면 끝입니다.

```js
const FIREBASE_CONFIG = {
  apiKey: "AIzaSy...",
  authDomain: "sag-2026.firebaseapp.com",
  projectId: "sag-2026",
  storageBucket: "sag-2026.firebasestorage.app",
  messagingSenderId: "...",
  appId: "..."
};
```

비워둔 채 배포해도 앱은 **미연결(로컬 전용) 모드**로 정상 동작합니다.
이 경우 화면 하단 **내보내기 → 🔗 Firestore 연결하기** 패널에 config를 붙여넣어
브라우저 단위로 연결할 수 있지만, 브라우저에만 저장되므로 파일에 넣는 편을 권장합니다.

---

## ③ 행사 정보 입력

`index.html` 상단의 `const EVENT = { ... }` 블록만 수정하면 홈 화면, Agenda 헤더,
TXT 리포트, 엑셀 내보내기에 일괄 반영됩니다.

| 키 | 설명 |
|---|---|
| `name` | 정식 명칭 (기본값: 2026 Scientific Advisory Group) |
| `shortName` | 파일명·리포트용 약칭 |
| `subtitle` | 부제 |
| `partners` | 공동주최 기관 |
| `venue` / `venueShort` | 장소 (전체 / 요약) |
| `time` | 진행 시간 |
| `expected` / `sessions` / `duration` | 홈 화면 요약 카드 값 |
| `dateLabel` | 화면 표시용 날짜 |
| `y` / `m` / `d` | 첫째 날 — D-day 기준 (**m은 0-index**: 11월 = 10) |
| `endY` / `endM` / `endD` | 마지막 날 |
| `days` | 총 일수 |

현재 설정: **2026년 11월 9일(월)–10일(화), 2일** (`y:2026, m:10, d:9`)

D-day 표시는 기간에 따라 자동으로 바뀝니다.

| 시점 | 표시 |
|---|---|
| 11/8 이전 | `D-1`, `D-87` … / "일 남음" |
| 11/9 | `Day 1` / "진행 중" |
| 11/10 | `Day 2` / "진행 중" |
| 11/11 이후 | `D+2` / "일 경과" |

---

## 데이터 저장 구조

| 항목 | 값 |
|---|---|
| Firestore 컬렉션 | `sag2026` |
| 주 문서 | `sag2026/main` |
| 참석자 문서 | `sag2026/attendees` |
| localStorage 접두사 | `sag_*`, `sag2026_*` |

GVF 앱(`gvf2026` 컬렉션 / `gvf-407a5` 프로젝트)과 겹치는 키가 없습니다.
