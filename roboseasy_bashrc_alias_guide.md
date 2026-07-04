# RobosEasy `~/.bashrc` / alias 세팅 가이드

> **대상 셸:** bash (Ubuntu 24.04) &nbsp;|&nbsp; **ROS2 Jazzy** &nbsp;|&nbsp; **워크스페이스:** `~/roboseasy_ws`
>
> 팀원 모두가 동일한 환경 변수 · alias를 쓰도록 통일하기 위한 가이드.
> 워크스페이스 · 패키지 규약은 [ROS2 Jazzy 규칙서](roboseasy_ros2_jazzy_convention.md) 참고.
> Ubuntu 22.04 (Humble) 환경은 문서 맨 아래 [Humble 사용자 참고](#humble-사용자-참고)를 확인한다.

---

## 📋 alias 요약표

| alias | 실제 명령 | 설명 |
|-------|-----------|------|
| `cw` | `cd ~/roboseasy_ws` | 워크스페이스로 이동 |
| `cs` | `cd ~/roboseasy_ws/src` | 소스 폴더로 이동 |
| `cb` | `cd ~/roboseasy_ws && colcon build --symlink-install` | 전체 빌드 |
| `cbs` | `colcon build --symlink-install --packages-select` | 특정 패키지만 빌드 |
| `cbu` | `colcon build --symlink-install --packages-up-to` | 의존성까지 빌드 |
| `sb` | `source ~/roboseasy_ws/install/setup.bash` | 워크스페이스 소스 |
| `sr` | `source /opt/ros/jazzy/setup.bash` | ROS2 소스 |
| `cc` | `cd ~/roboseasy_ws && rm -rf build install log` | 빌드 산출물 정리 |
| `rt` | `ros2 topic list` | 토픽 목록 |
| `rn` | `ros2 node list` | 노드 목록 |
| `re` | `ros2 topic echo` | 토픽 내용 확인 |
| `rs` | `ros2 service list` | 서비스 목록 |

---

## 1. `~/.bashrc` 편집 방법

**규칙:** 팀 공통 설정은 파일 하단에 `# ===== RobosEasy =====` 블록으로 모아 관리한다

```bash
# 편집
nano ~/.bashrc     # 또는 code ~/.bashrc

# 저장 후 현재 셸에 즉시 반영
source ~/.bashrc
```

> ⚠️ 개인 설정을 파일 곳곳에 흩뿌리지 말고, **한 블록에 모아** 두면 팀원 간 복사·비교·수정이 쉽다.

---

## 2. ROS2 환경 소스 (source)

**규칙:** `/opt/ros/jazzy` → 워크스페이스 순서로 source

```bash
# ── ROS2 배포판 소스 (항상 먼저) ──
source /opt/ros/jazzy/setup.bash

# ── 워크스페이스 소스 (빌드 후 존재할 때만) ──
if [ -f "$HOME/roboseasy_ws/install/setup.bash" ]; then
	source "$HOME/roboseasy_ws/install/setup.bash"
fi
```

> ⚠️ 순서가 중요하다. 배포판을 먼저 source 한 뒤 워크스페이스를 source 해야 로컬 패키지가 배포판을 덮어쓴다(overlay).
> 워크스페이스 setup 존재 여부를 `if [ -f ... ]`로 확인하면, 아직 빌드 전이라 파일이 없을 때 나는 오류를 막을 수 있다.
>
> ⚠️ 한 셸에서 **Humble과 Jazzy를 동시에 source 하지 않는다.** 배포판이 섞이면 빌드·통신이 모두 깨진다.

---

## 3. 환경 변수

**규칙:** `ROS_DOMAIN_ID`는 **팀에서 배정한 고정값** / 통신 범위는 `ROS_AUTOMATIC_DISCOVERY_RANGE`로 제한

같은 네트워크에서 여러 사람이 ROS2를 켜면 서로의 토픽이 섞인다. 도메인 격리 + 발견 범위 제한을 함께 쓴다.

```bash
# ── DDS 도메인 격리 (팀 배정값 사용, 0~101) ──
export ROS_DOMAIN_ID=17

# ── 자동 발견 범위 제한 (단일 PC 개발 시 권장) ──
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST

# ── RMW(DDS) 구현 통일 ──
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp

# ── 로그 색상 출력 ──
export RCUTILS_COLORIZED_OUTPUT=1
```

| 변수 | 역할 | 권장값 |
|------|------|--------|
| `ROS_DOMAIN_ID` | DDS 통신 도메인 격리 | 팀 배정 고정값 (예: 17) |
| `ROS_AUTOMATIC_DISCOVERY_RANGE` | 노드 자동 발견 범위 | 단독 개발 `LOCALHOST`, 다중 PC `SUBNET` |
| `ROS_STATIC_PEERS` | 자동 발견 없이 지정 피어와만 통신 | 필요 시 `'192.168.0.10;192.168.0.11'` |
| `RMW_IMPLEMENTATION` | 사용할 DDS 미들웨어 | `rmw_fastrtps_cpp` (기본) |
| `RCUTILS_COLORIZED_OUTPUT` | 로그 색상 | `1` |

```bash
# ❌ BAD — Humble 방식 (Jazzy에서 deprecated 경고 출력)
export ROS_LOCALHOST_ONLY=1
```

> ⚠️ **팀 규칙:** `ROS_DOMAIN_ID`는 각자 다른 값을 쓰되, 협업(다중 PC 연동) 시에는 **같은 값**으로 맞춘다.
> 여러 PC를 연결할 때는 `ROS_AUTOMATIC_DISCOVERY_RANGE=SUBNET`으로 바꿔야 서로 통신된다.

---

## 4. colcon 편의 기능

**규칙:** 자동완성과 `colcon_cd`를 활성화한다

```bash
# ── colcon 명령/패키지 자동완성 ──
source /usr/share/colcon_argcomplete/hook/register-python-argcomplete.bash

# ── colcon_cd: 패키지 이름으로 바로 이동 ──
source /usr/share/colcon_cd/function/colcon_cd.sh
export _colcon_cd_root=/opt/ros/jazzy/
```

```bash
# 사용 예 — 패키지 폴더로 즉시 이동
colcon_cd roboseasy_state_estimator
```

---

## 5. 팀 표준 alias

**규칙:** 이동 · 빌드 · 소스 · 디버깅 명령을 alias로 통일

```bash
# ── 이동 ──
alias cw='cd ~/roboseasy_ws'
alias cs='cd ~/roboseasy_ws/src'

# ── 빌드 (항상 워크스페이스 루트에서) ──
alias cb='cd ~/roboseasy_ws && colcon build --symlink-install'
alias cbs='colcon build --symlink-install --packages-select'
alias cbu='colcon build --symlink-install --packages-up-to'

# ── 소스 ──
alias sr='source /opt/ros/jazzy/setup.bash'
alias sb='source ~/roboseasy_ws/install/setup.bash'

# ── 정리 ──
alias cc='cd ~/roboseasy_ws && rm -rf build install log'

# ── ROS2 디버깅 단축 ──
alias rt='ros2 topic list'
alias rn='ros2 node list'
alias re='ros2 topic echo'
alias rs='ros2 service list'
```

```bash
# 사용 예 — 특정 패키지 빌드 후 소스
cbs roboseasy_state_estimator && sb
```

> ⚠️ alias 이름은 짧되 **팀 전체가 동일하게** 쓴다. 개인마다 다르면 문서·화면 공유 시 혼란이 생긴다.

---

## 6. 편의 함수 (function)

**규칙:** 인자가 필요한 복합 명령은 alias 대신 function으로 작성

```bash
# ── 빌드 + 소스를 한 번에 ──
cbsource() {
	cd ~/roboseasy_ws && \
	colcon build --symlink-install --packages-select "$@" && \
	source install/setup.bash
}

# ── 워크스페이스 완전 초기화 후 재빌드 ──
rebuild() {
	cd ~/roboseasy_ws && \
	rm -rf build install log && \
	colcon build --symlink-install && \
	source install/setup.bash
}
```

```bash
# 사용 예
cbsource roboseasy_state_estimator roboseasy_msgs
```

> 💡 **alias vs function:** 인자를 받지 않는 고정 명령은 `alias`, 인자(`$@`)를 넘겨야 하면 `function`을 쓴다.

---

## 7. 전체 예시 블록 (복사용)

아래 블록을 `~/.bashrc` **맨 아래**에 붙여넣고 `ROS_DOMAIN_ID`만 본인 값으로 바꾼다.

```bash
# ============================================================
# ===== RobosEasy ROS2 개발 환경 (Ubuntu 24.04 / Jazzy) =====
# ============================================================

# ── ROS2 소스 ──
source /opt/ros/jazzy/setup.bash
if [ -f "$HOME/roboseasy_ws/install/setup.bash" ]; then
	source "$HOME/roboseasy_ws/install/setup.bash"
fi

# ── 환경 변수 ──
export ROS_DOMAIN_ID=17                            # ← 본인/팀 배정값으로 변경
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST     # 다중 PC 연동 시 SUBNET으로
export RMW_IMPLEMENTATION=rmw_fastrtps_cpp
export RCUTILS_COLORIZED_OUTPUT=1

# ── colcon 편의 기능 ──
source /usr/share/colcon_argcomplete/hook/register-python-argcomplete.bash
source /usr/share/colcon_cd/function/colcon_cd.sh
export _colcon_cd_root=/opt/ros/jazzy/

# ── 이동 alias ──
alias cw='cd ~/roboseasy_ws'
alias cs='cd ~/roboseasy_ws/src'

# ── 빌드 alias ──
alias cb='cd ~/roboseasy_ws && colcon build --symlink-install'
alias cbs='colcon build --symlink-install --packages-select'
alias cbu='colcon build --symlink-install --packages-up-to'

# ── 소스 alias ──
alias sr='source /opt/ros/jazzy/setup.bash'
alias sb='source ~/roboseasy_ws/install/setup.bash'

# ── 정리 alias ──
alias cc='cd ~/roboseasy_ws && rm -rf build install log'

# ── ROS2 디버깅 alias ──
alias rt='ros2 topic list'
alias rn='ros2 node list'
alias re='ros2 topic echo'
alias rs='ros2 service list'

# ── 편의 함수 ──
cbsource() {
	cd ~/roboseasy_ws && \
	colcon build --symlink-install --packages-select "$@" && \
	source install/setup.bash
}
# ============================================================
```

> 붙여넣은 뒤 `source ~/.bashrc`로 반영한다.

---

## Humble 사용자 참고

Ubuntu 22.04 (ROS2 Humble) 환경에서는 위 블록에서 **딱 두 가지만** 바꾼다.

| 항목 | Jazzy (이 문서) | Humble |
|------|----------------|--------|
| source 경로 / `_colcon_cd_root` / `sr` | `/opt/ros/jazzy/` | `/opt/ros/humble/` |
| 통신 범위 제한 | `ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST` | `ROS_LOCALHOST_ONLY=1` (다중 PC 시 `0`) |

프로젝트 규약은 [ROS2 Humble 규칙서](roboseasy_ros2_humble_convention.md) 참고.

---

*RobosEasy `.bashrc` / alias Setup Guide v1.1 — Ubuntu 24.04 + ROS2 Jazzy 기반*
