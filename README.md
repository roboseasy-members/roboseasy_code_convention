# Roboseasy 코드 컨벤션

RobosEasy 팀의 코드 · 프로젝트 · 개발 환경 규약 문서 모음입니다.
각 문서는 **마크다운(문서)** 과 **웹(HTML)** 두 가지 버전을 제공합니다.

## 📚 문서 목록

| 문서 | 내용 | 마크다운 | 웹(HTML) |
|------|------|:-------:|:-------:|
| **C++ 코드 규칙서** | 클래스 · 함수 · 변수 네이밍, 스타일 (C++17) | [md](roboseasy_cpp_convention.md) | [web](html/roboseasy_cpp_convention.html) |
| **Python 코드 규칙서** | 네이밍 · 타입 힌트 · Docstring + ROS2 Python 패턴 | [md](roboseasy_python_convention.md) | [web](html/roboseasy_python_convention.html) |
| **ROS2 Humble 규칙서** | 패키지 · 워크스페이스 · 빌드 · 인터페이스 (Ubuntu 22.04) | [md](roboseasy_ros2_humble_convention.md) | [web](html/roboseasy_ros2_humble_convention.html) |
| **ROS2 Jazzy 규칙서** | Ubuntu 24.04 + Jazzy 전용 차이점 · 마이그레이션 | [md](roboseasy_ros2_jazzy_convention.md) | [web](html/roboseasy_ros2_jazzy_convention.html) |
| **`.bashrc` / alias 가이드** | ROS2 개발 환경 변수 · 팀 표준 alias 세팅 | [md](roboseasy_bashrc_alias_guide.md) | [web](html/roboseasy_bashrc_alias_guide.html) |

> 💡 코드 **내부** 규칙은 언어별 규칙서(C++ / Python), 코드 **외부**(패키지 · 빌드 · 워크스페이스) 규칙은 ROS2 규칙서를 참고합니다.
> Ubuntu 버전에 따라 ROS2 Humble(22.04) 또는 Jazzy(24.04) 규칙서를 선택하세요.

---

## 깃헙 레파지토리 생성 규칙

### 레파지토리 이름

- 모두 **소문자**로 작성한다.
- **snake_case** (언더바 `_`) 형식을 사용한다.
- 숫자 사용이 가능하다.

### 예시

```
soarm101
roboseasy_code_convention
```

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
