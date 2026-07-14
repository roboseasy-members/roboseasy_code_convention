# RobosEasy Git 규칙서

> **커밋 태그:** `Tag: 한줄 설명` 형식 &nbsp;|&nbsp; **브랜치:** main / develop / feature &nbsp;|&nbsp; **버전:** Semantic Versioning
>
> **적용 대상:** RobosEasy 팀의 모든 GitHub 레포지토리

---

## 📋 규칙 요약표

| 대상 | 규칙 | 예시 |
|------|------|------|
| 레포지토리 이름 | 소문자 snake_case | `soarm101`, `roboseasy_code_convention` |
| 브랜치 구조 | main / develop / feature/{이름} | `feature/ik`, `feature/vla` |
| **커밋 메시지** | **`Tag: 한줄 설명`** | `Add: IK solver 초기 구현` |
| 커밋 태그 | 대문자 시작 + 콜론(`:`) + 공백 | `Add:`, `Fix:`, `Improve:` |
| 커밋 단위 | 1 커밋 = 1 목적 | 기능 추가와 버그 수정을 한 커밋에 X |
| 버전 태그 | `v` + Semantic Versioning | `v1.0.0`, `v2.1.3` |
| 금지 사항 | main 직접 push / force push / 비밀정보 커밋 | — |

---

## 1. 레포지토리 네이밍

**규칙:** 모두 소문자 + snake_case — 숫자 사용 가능, 하이픈(-) 대신 언더바(_)

```
# ✅ GOOD
soarm101
roboseasy_code_convention
alice4_walking_module

# ❌ BAD
SoArm101                  # 대문자 X
roboseasy-code-convention # 하이픈 X
RoboseasyCodeConvention   # PascalCase X
```

---

## 2. 브랜치 전략

**규칙:** `main` / `develop` / `feature/{이름}` 3단 구조

| 브랜치 | 용도 | 규칙 |
|--------|------|------|
| `main` | 완료된 프로젝트의 최종 코드 | 프로젝트 완료 시 `develop`에서 머지 |
| `develop` | 진행중인 프로젝트의 최신 코드 | `feature` 브랜치의 머지 대상 |
| `feature/{이름}` | 각 기능별 개발 브랜치 | `develop`에서 분기, 완성 후 `develop`으로 머지 |

```
# 예: so-arm 레포에서 IK 연구와 VLA 연구를 각각 진행하는 경우
main
develop
feature/ik
feature/vla
```

```bash
# feature 브랜치 생성 — develop에서 분기
git switch develop
git switch -c feature/ik

# 작업 완료 후 develop으로 머지
git switch develop
git merge feature/ik
git push origin develop
```

> ⚠️ 각 개발자는 자신의 `feature` 브랜치에서만 작업한다. `main`과 `develop`에서 직접 커밋하지 않는다.

---

## 3. 커밋 메시지 형식 ⭐

**규칙:** `Tag: 한줄 설명` — 태그는 대문자로 시작, 콜론(`:`) 뒤 공백 1개

- 제목은 **한 줄**로, 무엇을 왜 했는지 요약한다 (50자 이내 권장).
- 한국어 작성을 허용한다. 끝에 마침표를 붙이지 않는다.
- "무엇을 했는지"가 커밋 메시지만 보고도 파악되어야 한다.

```
# ✅ GOOD
Add: IK solver 초기 구현
Fix: VLA 모델 입력 차원 오류 수정
Improve: 관절 보간 알고리즘 성능 개선
Delete: 사용하지 않는 테스트 파일 제거
Merge: feature/ik를 develop에 머지

# ❌ BAD
add: ik solver          # 소문자 태그 X
Add : IK solver         # 콜론 앞 공백 X
Add:IK solver           # 콜론 뒤 공백 없음 X
수정함                   # 태그 없음 X
Fix: 버그 수정           # 무엇을 고쳤는지 알 수 없음 X
Add: IK solver 추가하고 URDF 오타도 수정  # 두 가지 목적 X → 커밋 분리
```

---

## 4. 커밋 태그 종류 ⭐

**규칙:** 기본 태그 5종을 우선 사용하고, 상황에 맞으면 보조 태그를 사용한다

### 기본 태그 (필수 숙지)

| 태그 | 사용 시점 | 예시 |
|------|----------|------|
| `Add:` | 새 파일 · 코드 · 기능 추가 | `Add: FSR 센서 드라이버 노드 추가` |
| `Fix:` | 버그 · 오류 · 오타 수정 | `Fix: ZMP 계산 부호 오류 수정` |
| `Improve:` | 기존 기능의 성능 · 품질 개선 | `Improve: 상태 추정 필터 지연 감소` |
| `Delete:` | 파일 · 코드 삭제 | `Delete: 미사용 legacy 컨트롤러 제거` |
| `Merge:` | 브랜치 병합 커밋 | `Merge: feature/vla를 develop에 머지` |

### 보조 태그

| 태그 | 사용 시점 | 예시 |
|------|----------|------|
| `Docs:` | 문서 작성 · 수정 (README, 규칙서, 주석) | `Docs: 설치 가이드에 Jazzy 절차 추가` |
| `Refactor:` | 동작 변경 없는 구조 정리 | `Refactor: 콜백 함수를 별도 클래스로 분리` |
| `Style:` | 포매팅 · 네이밍 등 로직과 무관한 수정 | `Style: 규칙서에 맞게 변수명 snake_case로 통일` |
| `Test:` | 테스트 코드 추가 · 수정 | `Test: IK solver 단위 테스트 추가` |
| `Rename:` | 파일 · 폴더 이름 변경, 위치 이동 | `Rename: estimator.py를 state_estimator.py로 변경` |
| `Setup:` | 빌드 설정 · 의존성 · 환경 구성 | `Setup: package.xml에 pinocchio 의존성 추가` |
| `Revert:` | 이전 커밋 되돌리기 | `Revert: Add: IK solver 초기 구현 롤백` |
| `Hotfix:` | 배포된 main의 긴급 수정 | `Hotfix: 부팅 시 노드 크래시 긴급 수정` |
| `Release:` | 버전 릴리즈 · 태깅 | `Release: v1.2.0` |

### 태그 선택 기준 (헷갈릴 때)

| 상황 | 태그 |
|------|------|
| 안 되던 것을 되게 함 | `Fix:` |
| 되던 것을 더 좋게 함 (성능·기능) | `Improve:` |
| 되던 것을 그대로 두고 코드만 정리 | `Refactor:` |
| 로직은 안 건드리고 포맷·이름만 정리 | `Style:` |
| 코드가 아닌 문서만 수정 | `Docs:` |

---

## 5. 커밋 단위

**규칙:** 1 커밋 = 1 목적 (atomic commit)

하나의 커밋에는 하나의 논리적 변경만 담는다. 태그를 하나로 정할 수 없다면 커밋을 분리하라는 신호다.

```
# ✅ GOOD — 목적별로 분리
Add: FSR 센서 드라이버 노드 추가
Fix: IMU 콜백 타임스탬프 오류 수정
Docs: README에 실행 방법 추가

# ❌ BAD — 한 커밋에 여러 목적
Add: FSR 드라이버 추가, IMU 버그 수정, README 정리
```

> 💡 커밋을 자주, 작게 남기면 문제 발생 시 `git revert` / `git bisect`로 원인 커밋을 쉽게 찾을 수 있다.

---

## 6. 커밋 본문 (선택)

**규칙:** 상세 설명이 필요하면 제목 아래 빈 줄 하나 두고 본문 작성

```
Fix: 한 발 지지 시 CoP 계산 오차 수정

- FSR 4채널 중 비접촉 발 채널이 노이즈로 0이 아닌 값을 출력
- 접촉 판정 임계값(10N) 미만 채널은 CoP 계산에서 제외하도록 변경
- 관련 이슈: #23
```

> 💡 제목만으로 충분하면 본문은 생략한다. 이슈가 있으면 `#번호`로 참조한다.

---

## 7. 머지 규칙

**규칙:** `feature → develop → main` 방향으로만 머지

- `feature` 완성 → `develop`으로 머지 (`Merge:` 태그)
- 프로젝트 완료 → `develop`을 `main`으로 머지
- 머지 전 충돌(conflict)은 **feature 브랜치 쪽에서** 해결한 후 머지한다.
- 협업 레포에서는 GitHub **Pull Request**를 통한 머지를 권장한다 (리뷰 1인 이상).

```bash
# 머지 전 develop 최신화 후 feature에서 충돌 해결
git switch feature/ik
git pull origin develop     # 충돌 발생 시 여기서 해결
git switch develop
git merge feature/ik
```

---

## 8. 버전 태그 (Release)

**규칙:** `v` 접두사 + Semantic Versioning (`vMAJOR.MINOR.PATCH`)

| 자리 | 올리는 시점 | 예시 |
|------|------------|------|
| MAJOR | 호환성이 깨지는 변경 | `v1.x.x → v2.0.0` |
| MINOR | 기능 추가 (호환 유지) | `v1.0.x → v1.1.0` |
| PATCH | 버그 수정 | `v1.0.0 → v1.0.1` |

```bash
git tag -a v1.2.0 -m "Release: v1.2.0"
git push origin v1.2.0
```

---

## 9. 금지 사항 🚫

| 금지 항목 | 이유 |
|-----------|------|
| `main` / `develop`에 직접 push | 리뷰 없는 코드가 공유 브랜치를 오염 |
| 공유 브랜치에 `git push --force` | 팀원의 커밋 이력이 사라짐 |
| 비밀정보 커밋 (토큰, 키, 비밀번호) | 한 번 push되면 이력에 영구 잔존 |
| 빌드 산출물 커밋 (`build/`, `install/`, `log/`) | 레포 용량 증가, 충돌 유발 |
| 대용량 바이너리 커밋 (rosbag, 학습 weight 등) | 100MB 초과 시 push 거부, clone 속도 저하 |
| 개인 설정 파일 커밋 (`.vscode/`, IDE 설정) | 팀원 환경을 덮어씀 |

> ⚠️ 비밀정보를 실수로 커밋했다면 즉시 팀에 공유하고 해당 키를 **폐기·재발급**한다. 커밋 삭제만으로는 안전하지 않다.

---

## 10. .gitignore 표준 (ROS2 워크스페이스)

**규칙:** 레포 생성 직후 `.gitignore`부터 커밋한다

```gitignore
# ROS2 빌드 산출물
build/
install/
log/

# Python
__pycache__/
*.py[cod]
*.egg-info/

# C++
*.o
*.so
*.a

# 데이터 / 로그
*.bag
rosbag2_*/
*.csv

# IDE / 개인 설정
.vscode/
.idea/
*.swp

# 비밀정보
.env
*.pem
*_token*
```

---

*RobosEasy Git Convention v1.0 — 커밋 태그 · 브랜치 전략 · 레포 관리 규칙*
