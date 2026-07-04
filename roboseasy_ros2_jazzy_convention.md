# RobosEasy ROS2 Jazzy 프로젝트 / 워크스페이스 규칙서 (Ubuntu 24.04)

> **기반 프레임워크:** ROS2 Jazzy Jalisco (LTS, ~2029.05) &nbsp;|&nbsp; **빌드 시스템:** colcon (ament) &nbsp;|&nbsp; **OS:** Ubuntu 24.04 &nbsp;|&nbsp; **Python 3.12**
>
> 패키지 · 워크스페이스 · 네이밍 · 인터페이스 규칙은 [ROS2 Humble 규칙서](roboseasy_ros2_humble_convention.md)와 **완전히 동일**하다.
> 이 문서는 Ubuntu 24.04 + Jazzy 환경에서 **달라지는 것만** 다룬다.

---

## 📋 Humble ↔ Jazzy 차이 요약표

| 항목 | Humble (Ubuntu 22.04) | Jazzy (Ubuntu 24.04) |
|------|----------------------|----------------------|
| 소스 경로 | `/opt/ros/humble/setup.bash` | `/opt/ros/jazzy/setup.bash` |
| Python | 3.10 | **3.12** |
| 로컬 통신 제한 | `ROS_LOCALHOST_ONLY=1` | **`ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST`** (`ROS_LOCALHOST_ONLY`는 deprecated) |
| 다중 PC 연동 | `ROS_LOCALHOST_ONLY=0` | `ROS_AUTOMATIC_DISCOVERY_RANGE=SUBNET` (+ 필요시 `ROS_STATIC_PEERS`) |
| pip 설치 | 시스템에 직접 가능 | **차단됨 (PEP 668)** → apt/rosdep 우선, 예외 시 venv |
| 기본 RMW | `rmw_fastrtps_cpp` | `rmw_fastrtps_cpp` (동일) |
| `setup.py` 빌드 경고 | 없음 | `SetuptoolsDeprecationWarning` 출력 (무시 가능) |
| 네이밍 · 패키지 구조 | — | **Humble 규칙서와 동일** |

---

## 1. ROS2 환경 소스

**규칙:** `humble` 대신 `jazzy`를 source 한다 — 나머지 순서(배포판 → 워크스페이스)는 동일

```bash
# ── ROS2 배포판 소스 (항상 먼저) ──
source /opt/ros/jazzy/setup.bash

# ── 워크스페이스 소스 (빌드 후 존재할 때만) ──
if [ -f "$HOME/roboseasy_ws/install/setup.bash" ]; then
	source "$HOME/roboseasy_ws/install/setup.bash"
fi
```

> ⚠️ 한 셸에서 **Humble과 Jazzy를 동시에 source 하지 않는다.** 서로 다른 배포판을 섞으면 빌드·통신이 모두 깨진다.
> 두 배포판을 오가는 경우 alias(`sr_humble`, `sr_jazzy`)로 분리하고 새 터미널에서 하나만 source 한다.

---

## 2. 통신 범위 환경 변수 (Humble과 가장 다른 부분) ⭐

**규칙:** `ROS_LOCALHOST_ONLY` 대신 `ROS_AUTOMATIC_DISCOVERY_RANGE`를 사용한다

Jazzy에서 `ROS_LOCALHOST_ONLY`는 **deprecated**이며 설정 시 경고가 출력된다.

```bash
# ✅ GOOD — Jazzy 방식

# 단일 PC 개발 (외부 노출 차단)
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST

# 다중 PC 연동 (같은 서브넷 자동 발견)
export ROS_AUTOMATIC_DISCOVERY_RANGE=SUBNET

# 특정 PC와만 통신 (자동 발견 끄고 지정 피어만)
export ROS_AUTOMATIC_DISCOVERY_RANGE=LOCALHOST
export ROS_STATIC_PEERS='192.168.0.10;192.168.0.11'

# ❌ BAD — Humble 방식 (Jazzy에서 deprecated 경고)
export ROS_LOCALHOST_ONLY=1
```

| 값 | 의미 |
|----|------|
| `LOCALHOST` | 같은 PC 안에서만 자동 발견 (단독 개발 권장) |
| `SUBNET` | 같은 서브넷 전체에서 자동 발견 (기본값) |
| `OFF` | 자동 발견 안 함 — `ROS_STATIC_PEERS`로 지정한 피어만 |
| `SYSTEM_DEFAULT` | DDS 설정 파일을 그대로 따름 |

> ⚠️ **팀 규칙:** `ROS_DOMAIN_ID` 배정 규칙은 Humble과 동일하게 유지한다. 도메인 격리 + 발견 범위 제한을 함께 쓴다.

---

## 3. Python 패키지 설치 (PEP 668) ⭐

**규칙:** apt / rosdep 우선 — 시스템 Python에 `pip install` 금지

Ubuntu 24.04의 Python 3.12는 **externally-managed** 환경이라 시스템에 직접 `pip install` 하면 오류가 난다.

```bash
# ✅ GOOD — 1순위: apt (ROS 패키지 형태로 설치)
sudo apt install python3-numpy python3-scipy

# ✅ GOOD — 2순위: rosdep (package.xml 의존성 일괄 설치)
cd ~/roboseasy_ws
rosdep install --from-paths src --ignore-src -r -y

# ✅ GOOD — 3순위: apt에 없는 패키지만 venv 사용
python3 -m venv ~/roboseasy_venv --system-site-packages
source ~/roboseasy_venv/bin/activate
pip install some_package

# ❌ BAD — 시스템 Python에 직접 설치 (error: externally-managed-environment)
pip install numpy

# ❌ BAD — 강제 우회 (시스템 파이썬 오염, apt와 충돌)
pip install numpy --break-system-packages
```

> ⚠️ venv를 워크스페이스 안에 만들 경우 colcon이 빌드 대상으로 오인하지 않도록
> venv 폴더에 빈 `COLCON_IGNORE` 파일을 넣는다: `touch ~/roboseasy_ws/venv/COLCON_IGNORE`
>
> 💡 `--system-site-packages` 옵션을 줘야 venv 안에서도 `rclpy` 등 apt로 설치된 ROS 패키지를 임포트할 수 있다.

---

## 4. colcon 빌드

**규칙:** Humble과 동일 — `setup.py` 관련 경고는 무시한다

```bash
cd ~/roboseasy_ws
colcon build --symlink-install
source install/setup.bash
```

Jazzy에서 `ament_python` 패키지를 빌드하면 아래와 같은 경고가 출력되는데, **정상 동작이며 무시한다.**

```
SetuptoolsDeprecationWarning: setup.py install is deprecated.
```

> ⚠️ 경고를 없애려고 `setup.py`를 `pyproject.toml`로 바꾸지 않는다. Jazzy의 `ament_python`은 여전히 `setup.py` 기반이다.
> 패키지 구조·`setup.py` 작성 규칙은 [Humble 규칙서 §5](roboseasy_ros2_humble_convention.md#5-setuppy-python-패키지)를 그대로 따른다.

---

## 5. `~/.bashrc` 세팅

**규칙:** [.bashrc / alias 가이드](roboseasy_bashrc_alias_guide.md)를 그대로 따른다

`.bashrc` / alias 가이드는 Ubuntu 24.04 + Jazzy 기준으로 작성되어 있다.
source 경로 · 환경 변수 · alias · 복사용 전체 블록 모두 해당 문서를 사용하면 된다.

---

## 6. Humble → Jazzy 마이그레이션 체크리스트

기존 Humble 워크스페이스를 Ubuntu 24.04 PC로 옮길 때 확인한다.

- [ ] `~/.bashrc`의 source 경로를 `humble` → `jazzy`로 변경
- [ ] `ROS_LOCALHOST_ONLY` → `ROS_AUTOMATIC_DISCOVERY_RANGE`로 교체
- [ ] `build/` `install/` `log/` 삭제 후 **전체 재빌드** (배포판 간 산출물 호환 안 됨)
- [ ] `rosdep update && rosdep install --from-paths src --ignore-src -r -y`로 의존성 재설치
- [ ] pip로 설치하던 Python 패키지 → apt(`python3-*`) 또는 venv로 전환
- [ ] Python 3.12 문법 호환 확인 (3.10 코드는 대부분 그대로 동작)

---

*RobosEasy ROS2 Jazzy Convention v1.0 — Ubuntu 24.04 + ROS2 Jazzy Jalisco 기반. 공통 규칙은 [Humble 규칙서](roboseasy_ros2_humble_convention.md) 참조*
