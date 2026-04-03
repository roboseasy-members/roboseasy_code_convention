# Roboseasy 코드 컨벤션

## C++ 코드 규칙서 바로가기

[Roboseasy C++ 코드 규칙서](roboseasy_cpp_convention.md)

## Python 코드 규칙서 바로가기

[Roboseasy Python 코드 규칙서](roboseasy_python_convention.md)

---

## 깃헙 브랜치 관리 규칙

### 브랜치 구조

| 브랜치 | 용도 |
|--------|------|
| `main` | 완료된 프로젝트의 최종 코드 |
| `develop` | 진행중인 프로젝트의 최신 코드 |
| `feature/{이름}` | 각 기능별 개발 브랜치 |

### 규칙

- **main** : 프로젝트가 완료되면 최종 코드를 `main`에 머지한다.
- **develop** : 현재 진행중인 프로젝트의 마지막 코드를 관리한다.
- **feature/{이름}** : 특정 기능을 개발할 때 `develop`에서 분기하여 작업한다.

### 예시

`so-arm` 레포지토리에서 IK 연구 개발과 VLA 연구를 각각 다른 사람이 진행하는 경우:

```
main
develop
feature/ik
feature/vla
```

각 개발자는 자신의 `feature` 브랜치에서 작업하고, 완성되면 `develop`으로 머지한다.

---

## 커밋 메시지 규칙

| 태그 | 사용 시점 | 형식 |
|------|----------|------|
| `[add]` | 코드 및 파일이 추가된 경우 | `[add] 요약 설명` |
| `[del]` | 코드 및 파일이 삭제된 경우 | `[del] 요약 설명` |
| `[fix]` | 코드 및 파일이 수정된 경우 | `[fix] 요약 설명` |
| `[improve]` | 코드 및 파일이 개선된 경우 | `[improve] 요약 설명` |
| `[merge]` | 코드 및 파일이 머지된 경우 | `[merge] 요약 설명` |

### 예시

```
[add] IK solver 초기 구현
[fix] VLA 모델 입력 차원 오류 수정
[improve] 관절 보간 알고리즘 성능 개선
[del] 사용하지 않는 테스트 파일 제거
[merge] feature/ik를 develop에 머지
```
