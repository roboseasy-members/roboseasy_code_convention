# RobosEasy Python 코드 규칙서

> **베이스:** Google Python Style Guide &nbsp;|&nbsp; **타입 힌트:** PEP 484 / PEP 526 &nbsp;|&nbsp; **Docstring:** Google Style (PEP 257) &nbsp;|&nbsp; **포매터:** Black / YAPF 권장
>
> **Python 3.10+** &nbsp;|&nbsp; **ROS2 Humble**

---

## 📋 규칙 요약표

| 대상 | 규칙 | 예시 |
|------|------|------|
| 패키지 / 모듈 | snake_case | `roboseasy_utils`, `state_estimator.py` |
| 클래스 | PascalCase | `StateEstimator`, `JointController` |
| 예외(Exception) | PascalCase + Error | `JointLimitError`, `SensorTimeoutError` |
| 함수 / 메서드 | snake_case | `calculate_zmp`, `get_joint_state` |
| 변수 (일반) | snake_case | `joint_position`, `control_time` |
| **Boolean 변수** | **`is_` / `has_` 접두사** | `is_active`, `has_velocity_interface` |
| 상수 | UPPER_SNAKE_CASE | `GRAVITY_ACCEL`, `MAX_JOINT_VEL` |
| protected 멤버 | `_single_leading` | `_joint_names`, `_timer` |
| private 멤버 | `__double_leading` | `__callback_group` |
| ROS2 노드 클래스 | PascalCase + Node | `StateEstimatorNode`, `JointControllerNode` |
| ROS2 토픽/서비스 | snake_case 슬래시 경로 | `/roboseasy/joint_state` |
| ROS2 콜백 함수 | snake_case + `_callback` | `imu_callback`, `timer_callback` |
| 파일명 | snake_case + `.py` | `joint_controller.py` |
| **들여쓰기** | **탭(Tab) 1개** | `insertSpaces: false, tabSize: 4` |
| **경로(이식성)** | **절대 경로 하드코딩 금지 / `BASE_DIR` 기준** | `BASE_DIR = os.path.dirname(os.path.abspath(__file__))` |

---

## 1. 모듈 / 패키지

**규칙:** `snake_case` — 짧고 소문자, 하이픈(-) 사용 금지

파일명과 패키지명은 소문자 snake_case. 하이픈(`-`)은 임포트가 불가능하므로 절대 사용하지 않는다.

```
# ✅ GOOD
roboseasy_utils.py
state_estimator.py
joint_controller.py
ros2_sensor_bridge.py

# ❌ BAD
RoboseasyUtils.py    # PascalCase X
state-estimator.py   # 하이픈 X
se.py                # 의미 없는 약어 X
```

---

## 2. 클래스 / 예외(Exception)

**규칙:** 클래스: `PascalCase` / 예외: `PascalCase + Error` 접미사

```python
# ✅ GOOD
class StateEstimator:
    pass

class JointController:
    pass

# 예외 클래스
class JointLimitError(Exception):
    pass

class SensorTimeoutError(RuntimeError):
    pass

# ❌ BAD
class state_estimator: pass      # snake_case X
class jointcontroller: pass      # 소문자 X
class JointLimit(Exception): pass    # Error 없음 X
class SensorException: pass      # Exception 접미사 X
```

---

## 3. 함수 / 메서드

**규칙:** `snake_case` — 동사로 시작, 파라미터 + 반환 타입 힌트 필수

```python
# ✅ GOOD
def calculate_zmp(
    force_z: float,
    cop_x: float,
    cop_y: float,
) -> tuple[float, float]:
    ...

def get_joint_position(joint_name: str) -> float:
    ...

def update_state(timestamp: float) -> None:
    ...

def is_foot_on_ground(threshold: float = 10.0) -> bool:
    ...

# ❌ BAD
def CalculateZMP(): ...      # PascalCase X
def zmp(): ...               # 의미 없는 약어 X
def doUpdate(): ...          # camelCase X
def get_positions(names):    # 타입 힌트 누락 X
    ...
```

---

## 4. 변수 (일반)

**규칙:** `snake_case` — 의미 있는 이름, 한 글자 변수 금지 (루프 인덱스 / 수학 표기 제외)

```python
# ✅ GOOD
joint_position = 0.0
control_time = 0.01
gravity_accel = 9.80665
imu_data = ImuData()

# 루프 인덱스 예외
for i in range(num_joints):
    ...

# 수학 표기 예외
x, y, z = position

# ❌ BAD
jp = 0.0          # 약어 X
CT = 0.01         # 대문자 X (상수 아님)
JointPos = 0.0    # PascalCase X
jointPos = 0.0    # camelCase X
a = get_imu_data() # 의미 없는 단문자 X
```

---

## 5. Boolean 변수 ⭐

**규칙:** 반드시 `is_` 또는 `has_` 접두사로 시작

`bool` 타입 변수는 변수명만 보고도 `True/False` 값임을 즉시 인식할 수 있어야 한다.

| 접두사 | 사용 상황 | 질문 형태 |
|--------|-----------|-----------|
| `is_` | 상태 / 조건 여부 | "~인가? / ~한 상태인가?" |
| `has_` | 기능 / 데이터 보유 여부 | "~를 가지고 있는가?" |

```python
# ✅ GOOD
class StateEstimatorNode(Node):
    def __init__(self) -> None:
        # 인스턴스 변수 — is_ / has_ 접두사 사용
        self.is_active: bool = False
        self.is_initialized: bool = False
        self.is_foot_contact: bool = False
        self.has_velocity_data: bool = False
        self.has_imu_subscriber: bool = False


# 로컬 변수도 동일 규칙
def process(self) -> None:
    is_valid = not self._queue.empty()
    has_data = len(self._buffer) > 0

    if is_valid and has_data:
        self._update()


# ❌ BAD
class BadNode(Node):
    def __init__(self) -> None:
        self.active = False           # is_ 없음 X
        self.velocity_data = False    # has_ 없음 X
        self.bActive = True           # 헝가리안 b X
        self.foot_contact_flag = True # flag 접미사 X
        self.check = True             # 의미 불명확 X
        self.status = False           # 타입 불분명 X
```

> 💡 **한 눈에 확인:**
> ```python
> if is_foot_contact and has_velocity_data:
> ```
> 타입 선언 없이도 모두 `bool`임을 즉시 인식.

---

## 6. 상수

**규칙:** `UPPER_SNAKE_CASE` — 모듈 최상단 선언, `Final` 타입 힌트 권장

```python
from typing import Final

GRAVITY_ACCEL: Final[float] = 9.80665
MAX_JOINT_VEL: Final[float] = 3.14
DEFAULT_CONTROL_HZ: Final[int] = 100
ROBOT_NAME: Final[str] = 'roboseasy_abo'
```

---

## 7. private / protected 멤버

**규칙:** protected: `_단일 밑줄` / private: `__이중 밑줄`

```python
class StateEstimatorNode(Node):
    def __init__(self) -> None:
        # protected — 서브클래스에서 접근 가능
        self._joint_names: list[str] = []
        self._control_time: float = 0.01
        self._is_initialized: bool = False

        # private — name mangling 적용
        self.__callback_group = ReentrantCallbackGroup()
        self.__timer = None
```

> **Google Style 권고:** Python에는 진정한 private이 없으므로 `_single_underscore`를 기본으로 사용하고, name mangling이 꼭 필요한 경우에만 `__double`을 사용한다.

---

## 8. 타입 힌트 — 기본 (PEP 484)

**규칙:** 모든 함수 파라미터와 반환값에 타입 힌트 필수

Python 3.10+에서는 `list[str]`, `dict[str, float]`처럼 내장 타입을 직접 사용한다.
`typing.List`, `typing.Dict` 등 구버전 방식은 사용하지 않는다.

```python
# ✅ GOOD — Python 3.10+ 내장 타입
def get_joint_positions(
    joint_names: list[str],
    timestamp: float,
) -> dict[str, float]:
    ...


class JointController:
    def __init__(self) -> None:
        self.joint_names: list[str] = []
        self.positions: dict[str, float] = {}
        self.control_hz: int = 100
        self.is_active: bool = False

    def publish_state(self) -> None:  # 반환 없는 메서드는 -> None
        ...

    def get_all_positions(self) -> dict[str, float]:  # 반환 타입 명시
        ...


# ❌ BAD — 구버전 typing 모듈 (Python 3.9 이하 방식)
from typing import List, Dict

def get_positions(names: List[str]) -> Dict[str, float]:  # X — list/dict 직접 사용
    ...


# ❌ BAD — 타입 힌트 누락
def get_positions(names):    # 파라미터 타입 없음 X
    ...                      # 반환 타입 없음 X
```

---

## 9. 타입 힌트 — 복합 타입

```python
import numpy as np
import numpy.typing as npt
from collections.abc import Callable

# tuple — 고정 길이: 원소 타입 모두 명시
def get_cop() -> tuple[float, float]:
    return 0.0, 0.0

# 가변 길이 tuple
def get_all_positions() -> tuple[float, ...]:
    ...

# numpy ndarray
def calculate_jacobian(
    q: npt.NDArray[np.float64],
) -> npt.NDArray[np.float64]:
    ...

# Callable
def register_callback(
    cb: Callable[[float], None],
) -> None:
    ...
```

---

## 10. 타입 힌트 — Optional / Union

**규칙:** Python 3.10+에서는 `X | None`, `X | Y` 문법 사용

```python
# ✅ GOOD — Python 3.10+

# 함수 반환값
def find_joint(name: str) -> float | None:
    ...

def set_target(value: int | float) -> None:
    ...

# 클래스 인스턴스 변수
class StateEstimatorNode(Node):
    def __init__(self) -> None:
        self._timer: rclpy.timer.Timer | None = None
        self._last_imu: Imu | None = None


# ❌ BAD — 구버전 Optional / Union
from typing import Optional, Union

def find_joint(name: str) -> Optional[float]:           # X → float | None
    ...

def set_target(value: Union[int, float]) -> None:       # X → int | float
    ...
```

---

## 11. Docstring — Google Style

**규칙:** `Args` / `Returns` / `Raises` / `Example` 섹션 구조

모든 public 모듈, 클래스, 함수에 Docstring을 작성한다. 한국어 작성을 허용한다.

```python
def calculate_dcm(
    com_pos: tuple[float, float],
    com_vel: tuple[float, float],
    omega: float,
) -> tuple[float, float]:
    """DCM (Divergent Component of Motion) 을 계산한다.

    LIPM 기반으로 발산 운동 성분을 계산하여 균형 제어에 사용한다.

    Args:
        com_pos: CoM 위치 (x, y) [m]
        com_vel: CoM 속도 (vx, vy) [m/s]
        omega: LIPM 고유 주파수 sqrt(g/h) [rad/s]

    Returns:
        DCM 위치 (xi_x, xi_y) [m] tuple 형태로 반환.

    Raises:
        ValueError: omega가 0 이하인 경우.

    Example:
        >>> dcm = calculate_dcm((0.0, 0.0), (0.1, 0.0), 3.13)
        >>> print(dcm)
        (0.031, 0.0)
    """
    if omega <= 0:
        raise ValueError(f'omega must be positive, got {omega}')
    xi_x = com_pos[0] + com_vel[0] / omega
    xi_y = com_pos[1] + com_vel[1] / omega
    return xi_x, xi_y
```

> **클래스 Docstring:** 클래스 수준에서는 `Attributes:` 섹션으로 주요 멤버를 기술한다.

---

## 12. 인라인 주석

**규칙:** `#` 뒤 공백 1개 / 코드와 같은 줄은 공백 2개 후 `#`

코드 로직을 설명하되, 코드 자체가 자명한 경우 생략한다. 한국어 주석을 허용한다.

```python
# ✅ GOOD
# CoM 속도를 저역통과 필터로 노이즈 제거
com_vel_x = self._lpf_x.filter(raw_vel_x)

control_time = 0.01  # 100 Hz 제어 주기 [s]

# ZMP가 지지 다각형 밖에 있을 때 균형 복구 트리거
if abs(zmp_x) > support_polygon_x:
    self._trigger_recovery()

# TODO: 발목 토크 피드포워드 항 추가 예정
# FIXME: 한 발 지지 시 CoP 계산 오차 확인 필요

# ❌ BAD — 코드를 그대로 번역하는 주석 금지
x = x + 1  # x에 1을 더함  (이런 주석은 오히려 가독성을 해침)
```

---

## 13. 들여쓰기

**규칙:** 탭(Tab) 1개 — 공백(Space) 혼용 금지

탭 1개로 들여쓰기한다. 공백과 탭을 **절대 혼용하지 않는다** (Python은 혼용 시 `TabError` 발생). 에디터에서 탭 너비는 4로 통일하는 것을 권장한다.

```python
# ✅ GOOD — 탭(Tab) 들여쓰기
class StateEstimatorNode(Node):

	def initialize_filters(
		self,
		control_hz: int,
		cutoff_freq: float,
		filter_type: str = 'lpf',
	) -> None:
		self._lpf = LowPassFilter(
			clock=self.get_clock(),
			cutoff=cutoff_freq,
		)

	def calculate_zmp(
		self,
		force_z: float,
		cop_x: float,
	) -> tuple[float, float]:
		zmp_x = cop_x - (force_z * self._height)
		return zmp_x, 0.0


# ❌ BAD — 공백과 탭 혼용 (TabError 발생)
class BadNode(Node):
    def __init__(self) -> None:  # 공백 4개
	    self.name = 'bad'         # 탭 + 공백 혼용 X

# ❌ BAD — 2칸 공백
class BadNode2(Node):
  def __init__(self) -> None:  # 2칸 공백 X
    self.name = 'bad'
```

> ⚠️ **팀 규칙:** 에디터(VSCode, PyCharm 등)의 탭 설정을 **Tab Size: 4, Insert Spaces: OFF**로 맞춰 통일한다.

---

## 14. 줄 길이

**규칙:** 한 줄 최대 **80자** (Google Style) — 괄호로 줄 바꿈, 백슬래시(`\`) 지양

```python
# ✅ GOOD — 괄호로 줄 바꿈
result = calculate_dcm(
    com_pos=self._com_position,
    com_vel=self._com_velocity,
    omega=self._omega,
)

if (self.is_foot_contact
        and self.has_velocity_data
        and self._is_initialized):
    self._update_state()

# ❌ BAD — 80자 초과
result = calculate_dcm(com_pos=self._com_position, com_vel=self._com_velocity, omega=self._omega)
```

---

## 15. 따옴표

**규칙:** 작은따옴표(`'`) 기본 — 팀 내 통일이 가장 중요

```python
# ✅ GOOD
node_name = 'state_estimator'
topic = '/roboseasy/joint_state'

# f-string 내부 표현식에 따옴표 필요 시 큰따옴표 허용
msg = f'joint {joint["name"]} error'

# 멀티라인
description = (
    'RobosEasy state estimator node. '
    'Subscribes to IMU and joint states.'
)

# ❌ BAD — 혼용 금지
name1 = 'abo'
name2 = "alice"   # 같은 파일에서 혼용 X
```

---

## 16. 임포트 순서

**규칙:** 표준 라이브러리 → 서드파티 → ROS2 → 로컬, 그룹 사이 빈 줄

와일드카드 임포트(`from X import *`) 금지.

```python
# ─── 1) Python 표준 라이브러리 ───────────────────────────
from __future__ import annotations
import math
from collections.abc import Callable
from typing import Final

# ─── 2) 서드파티 라이브러리 ──────────────────────────────
import numpy as np
import numpy.typing as npt

# ─── 3) ROS2 관련 ────────────────────────────────────────
import rclpy
from rclpy.node import Node
from rclpy.qos import QoSProfile
from sensor_msgs.msg import Imu, JointState
from std_msgs.msg import Float64MultiArray

# ─── 4) 로컬 (자체 패키지) ───────────────────────────────
from roboseasy_utils.filters import LowPassFilter
from roboseasy_msgs.msg import RobotState
```

> ⚠️ 각 그룹 사이에 **반드시 빈 줄 1개**를 둔다. `from X import *` 와일드카드 임포트 금지.

---

## 17. 공백 / 빈 줄

**규칙:** 최상위 클래스/함수 전후 2줄 / 메서드 사이 1줄 / 연산자 주변 공백 1개

```python
# ✅ GOOD


class StateEstimatorNode(Node):  # ← 위에 빈 줄 2개
    """상태 추정 노드."""

    def __init__(self) -> None:  # ← 클래스 내 첫 메서드 위 빈 줄 1개
        self.is_active: bool = False

    def timer_callback(self) -> None:  # ← 메서드 사이 빈 줄 1개
        self._update()

    def imu_callback(self, msg: Imu) -> None:  # ← 메서드 사이 빈 줄 1개
        self._imu_queue.append(msg)


def main() -> None:  # ← 최상위 함수 위 빈 줄 2개
    ...


# ✅ 연산자 주변 공백 1개
x = y + z
result = a * b + c * d
if x == 0:
    ...


# ❌ BAD — 빈 줄 없음
class BadNode(Node):
    def __init__(self) -> None:
        pass
    def timer_callback(self) -> None:  # 메서드 사이 빈 줄 없음 X
        pass
    def imu_callback(self, msg: Imu) -> None:  # X
        pass
def main() -> None:  # 클래스와 함수 사이 빈 줄 없음 X
    pass
```

---

## 18. 경로 처리 (이식성) ⭐

**규칙:** 절대 경로 하드코딩 금지 — 파일 기준 경로(`BASE_DIR`)나 환경 변수로 작성해, 개발자 개인 PC 계정에 의존하지 않고 누구나 그대로 실행할 수 있게 한다.

`/home/철수/...` 처럼 개인 계정·특정 PC에 종속된 절대 경로를 코드에 박아두면, 다른 사람이 클론했을 때 곧바로 깨진다. 파일·리소스 위치는 **항상** 현재 파일 위치를 기준으로 계산하거나, 환경 변수 / ROS2 파라미터로 주입한다.

```python
import os

# ✅ GOOD — 현재 파일 위치 기준으로 경로 계산 (누구나 그대로 실행 가능)
BASE_DIR = os.path.dirname(os.path.abspath(__file__))
CONFIG_PATH = os.path.join(BASE_DIR, 'config', 'robot.yaml')
URDF_PATH = os.path.join(BASE_DIR, '..', 'urdf', 'roboseasy.urdf')

# ✅ GOOD — 환경 변수 / 홈 디렉터리 기준 (계정명에 비종속)
DATA_DIR = os.path.join(os.path.expanduser('~'), 'roboseasy_data')
LOG_DIR = os.environ.get('ROBOSEASY_LOG_DIR', os.path.join(BASE_DIR, 'logs'))

# ✅ GOOD — ROS2 패키지 리소스는 ament index로 조회
from ament_index_python.packages import get_package_share_directory

pkg_share = get_package_share_directory('roboseasy_bringup')
urdf_path = os.path.join(pkg_share, 'urdf', 'roboseasy.urdf')


# ❌ BAD — 개인 계정 / 특정 PC 절대 경로 하드코딩
CONFIG_PATH = '/home/chulsoo/roboseasy_ws/config/robot.yaml'   # 계정명 종속 X
URDF_PATH = 'C:/Users/roboseasy/urdf/roboseasy.urdf'           # 특정 PC 종속 X
DATA_DIR = '/home/chulsoo/data'                                # 남이 클론하면 깨짐 X
```

> ⚠️ **핵심:** 경로는 "내 PC에서만 되는 코드"의 대표적 원인이다. `BASE_DIR = os.path.dirname(os.path.abspath(__file__))` 패턴을 기본으로 삼고, 문자열로 시작하는 절대 경로가 코드에 보이면 리뷰에서 반드시 지적한다. 최신 코드에서는 `pathlib.Path(__file__).resolve().parent` 사용도 권장한다.

---

## 🤖 ROS2 Python 패턴

---

## R1. 노드 클래스 구조

**규칙:** 클래스명 = `PascalCase + Node` / `Node` 상속 / `__init__` 순서: 파라미터 → 상태변수 → Pub/Sub → Timer

```python
from __future__ import annotations

import rclpy
from rclpy.node import Node
from sensor_msgs.msg import Imu, JointState
from roboseasy_msgs.msg import RobotState


class StateEstimatorNode(Node):
    """로봇 상태 추정 노드.

    IMU 및 관절 상태를 구독하여 로봇 상태를 추정하고 퍼블리시한다.
    """

    def __init__(self) -> None:
        super().__init__('state_estimator_node')

        # ── 1) 파라미터 선언 ──────────────────────────────
        self.declare_parameter('control_hz', 100)
        self._control_hz: int = (
            self.get_parameter('control_hz')
            .get_parameter_value()
            .integer_value
        )

        # ── 2) 내부 상태 변수 ─────────────────────────────
        self._is_initialized: bool = False
        self._has_imu_data: bool = False
        self._joint_names: list[str] = []

        # ── 3) Subscriber / Publisher ─────────────────────
        self._imu_sub = self.create_subscription(
            Imu,
            '/roboseasy/imu/data',
            self.imu_callback,
            10,
        )
        self._state_pub = self.create_publisher(
            RobotState,
            '/roboseasy/robot_state',
            10,
        )

        # ── 4) Timer ──────────────────────────────────────
        self._timer = self.create_timer(
            1.0 / self._control_hz,
            self.timer_callback,
        )

    def timer_callback(self) -> None:
        """주기 타이머 콜백."""
        ...

    def imu_callback(self, msg: Imu) -> None:
        """IMU 수신 콜백."""
        self._has_imu_data = True
```

---

## R2. 토픽 / 서비스 / 액션 이름

**규칙:** `/namespace/topic_name` 형식, snake_case / 파일 상단에 상수로 선언

```python
from typing import Final

# 토픽 이름을 상수로 선언 (파일 상단)
TOPIC_IMU: Final[str]         = '/roboseasy/imu/data'
TOPIC_JOINT_STATE: Final[str] = '/roboseasy/joint_states'
TOPIC_ROBOT_STATE: Final[str] = '/roboseasy/robot_state'
TOPIC_FSR: Final[str]         = '/roboseasy/fsr/data'

SRV_QUERY_STATE: Final[str]   = '/roboseasy/query_state'
ACTION_WALK: Final[str]        = '/roboseasy/walk'

NODE_NAME: Final[str] = 'state_estimator_node'
```

> ⚠️ 토픽명을 코드 전체에 문자열 리터럴로 흩뿌리지 않는다. 상수로 선언하면 오타를 방지하고 일괄 수정이 쉬워진다.

---

## R3. 콜백 함수

**규칙:** `snake_case + _callback` 접미사 / 메시지 파라미터명 `msg` 통일

```python
# ✅ GOOD
def timer_callback(self) -> None:
    """주기 타이머 콜백: 상태 추정 및 퍼블리시."""
    self._update_pinocchio()
    self._publish_state()

def imu_callback(self, msg: Imu) -> None:
    """IMU 데이터 수신 콜백."""
    self._imu_queue.append(msg)
    self._has_imu_data = True

def joint_state_callback(self, msg: JointState) -> None:
    """관절 상태 수신 콜백."""
    self._joint_state_queue.append(msg)

def fsr_callback(self, msg: FsrState) -> None:
    """FSR 센서 데이터 수신 콜백."""
    self._fsr_queue.append(msg)

# ❌ BAD
def on_imu(self, msg): ...          # _callback 없음 X
def handle_joint(self, msg): ...    # _callback 없음 X
def ImuCallback(self, msg): ...     # PascalCase X
def imu_callback(self, data): ...   # data X → msg 통일
```

---

## R4. 파라미터 선언

**규칙:** `declare_parameter` → `get_parameter` → 인스턴스 변수 저장

```python
def __init__(self) -> None:
    super().__init__('state_estimator_node')

    # 파라미터 선언 (기본값 포함)
    self.declare_parameter('control_hz',   100)
    self.declare_parameter('lipm_height',  0.75)
    self.declare_parameter('urdf_path',    '')
    self.declare_parameter('use_sim_time', False)

    # 파라미터 읽기 → 인스턴스 변수 저장
    self._control_hz: int = (
        self.get_parameter('control_hz')
        .get_parameter_value().integer_value
    )
    self._lipm_height: float = (
        self.get_parameter('lipm_height')
        .get_parameter_value().double_value
    )
    self._urdf_path: str = (
        self.get_parameter('urdf_path')
        .get_parameter_value().string_value
    )
```

---

## R5. main() 함수

**규칙:** 타입 힌트 포함 / `try-finally` 패턴 / `if __name__ == '__main__'`

```python
def main(args: list[str] | None = None) -> None:
    """노드 진입점."""
    rclpy.init(args=args)
    node = StateEstimatorNode()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        node.get_logger().info('Keyboard interrupt, shutting down.')
    finally:
        node.destroy_node()
        rclpy.shutdown()


if __name__ == '__main__':
    main()
```

---

*RobosEasy Python Coding Convention v1.0 — Google Python Style Guide + ROS2 Humble + PEP 484/526 기반*
