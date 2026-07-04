# RobosEasy ROS2 Humble 프로젝트 / 워크스페이스 규칙서

> **기반 프레임워크:** ROS2 Humble &nbsp;|&nbsp; **빌드 시스템:** colcon (ament) &nbsp;|&nbsp; **OS:** Ubuntu 22.04
>
> Ubuntu 24.04 환경은 [ROS2 Jazzy 규칙서](roboseasy_ros2_jazzy_convention.md) 참고.
>
> 코드 내부 규칙이 아닌 **패키지 · 워크스페이스 · 빌드 · 인터페이스** 등 프로젝트 수준 규약을 다룬다.
> 노드 내부 코드 규칙은 [Python 규칙서](roboseasy_python_convention.md) / [C++ 규칙서](roboseasy_cpp_convention.md) 참고.

---

## 📋 규칙 요약표

| 대상 | 규칙 | 예시 |
|------|------|------|
| 워크스페이스 | `{목적}_ws` snake_case | `roboseasy_ws`, `abo_ws` |
| 패키지 | `roboseasy_` 접두 + snake_case | `roboseasy_bringup`, `roboseasy_state_estimator` |
| 메타 패키지 | `{로봇}_` 접두 | `abo_description`, `abo_bringup` |
| 인터페이스 패키지 | `_msgs` / `_interfaces` 접미 | `roboseasy_msgs`, `roboseasy_interfaces` |
| 노드 실행파일 | snake_case + `_node` | `state_estimator_node` |
| 런치 파일 | snake_case + `.launch.py` | `bringup.launch.py` |
| 파라미터 파일 | snake_case + `.yaml` | `state_estimator.yaml` |
| msg 파일 | PascalCase + `.msg` | `RobotState.msg` |
| srv 파일 | PascalCase + `.srv` | `QueryState.srv` |
| action 파일 | PascalCase + `.action` | `Walk.action` |
| 토픽 / 서비스 | `/roboseasy/…` snake_case | `/roboseasy/joint_states` |
| 프레임(TF) ID | snake_case | `base_link`, `l_foot_link` |
| URDF / Xacro | snake_case + `.urdf.xacro` | `abo.urdf.xacro` |

---

## 1. 워크스페이스 구조

**규칙:** `~/{목적}_ws` 형식 — 소스는 반드시 `src/` 아래에만 둔다

colcon 워크스페이스는 `build/`, `install/`, `log/`, `src/`로 구성된다.
**`src/`를 제외한 나머지 3개 디렉토리는 절대 커밋하지 않는다** (`.gitignore`).

```
roboseasy_ws/
├── build/          # colcon 빌드 산출물 (git 제외)
├── install/        # 설치 산출물 · setup.bash (git 제외)
├── log/            # 빌드/실행 로그 (git 제외)
└── src/            # ← 소스 패키지만 여기에
    ├── roboseasy_bringup/
    ├── roboseasy_msgs/
    ├── roboseasy_state_estimator/
    └── roboseasy_description/
```

> ⚠️ 항상 **워크스페이스 최상단**(`roboseasy_ws/`)에서 `colcon build`를 실행한다.
> `src/` 안에서 빌드하면 `build/`·`install/`이 엉뚱한 위치에 생성된다.

---

## 2. 패키지 이름

**규칙:** `roboseasy_` 접두 + snake_case — 하이픈(`-`) 금지, 소문자만

패키지명은 곧 Python 임포트 이름 / ament 리소스 이름이 되므로 하이픈을 쓸 수 없다.

```
# ✅ GOOD
roboseasy_bringup            # 런치 · 설정 통합 패키지
roboseasy_state_estimator    # 상태 추정 노드 패키지
roboseasy_msgs               # 인터페이스(msg/srv/action) 패키지
roboseasy_description        # URDF · 메시 · RViz 설정
abo_bringup                 # 특정 로봇(abo) 전용은 로봇명 접두 허용

# ❌ BAD
roboseasy-bringup     # 하이픈 X (임포트 불가)
RoboseasyBringup      # PascalCase X
se_pkg                # 의미 없는 약어 X
bringup               # 접두 없음 → 다른 패키지와 충돌 위험
```

> 💡 **패키지 역할별 접미 관례**
> `_bringup`(통합 실행) · `_description`(URDF/메시) · `_msgs`(인터페이스) · `_control`(제어) · `_gazebo`(시뮬)

---

## 3. 패키지 내부 구조

**규칙:** Python / C++ 각각의 ament 표준 레이아웃을 따른다

### Python 패키지 (`ament_python`)

```
roboseasy_state_estimator/
├── package.xml
├── setup.py
├── setup.cfg
├── resource/
│   └── roboseasy_state_estimator      # ament 리소스 마커(빈 파일)
├── roboseasy_state_estimator/         # ← 실제 파이썬 모듈(패키지명과 동일)
│   ├── __init__.py
│   └── state_estimator_node.py
├── launch/
│   └── state_estimator.launch.py
├── config/
│   └── state_estimator.yaml
└── test/
    └── test_state_estimator.py
```

### C++ 패키지 (`ament_cmake`)

```
roboseasy_joint_controller/
├── package.xml
├── CMakeLists.txt
├── include/
│   └── roboseasy_joint_controller/    # 공개 헤더(패키지명 하위)
│       └── joint_controller.hpp
├── src/
│   └── joint_controller.cpp
├── launch/
│   └── joint_controller.launch.py
└── config/
    └── joint_controller.yaml
```

> ⚠️ Python 패키지에서 `resource/{패키지명}` 마커 파일과 `roboseasy_state_estimator/` 모듈 폴더 이름은
> **package.xml의 `<name>`과 정확히 일치**해야 한다. 하나라도 다르면 `ros2 run`이 노드를 찾지 못한다.

---

## 4. package.xml

**규칙:** `format="3"` / `<name>`은 패키지명과 일치 / 의존성은 사용 목적별 태그로 구분

```xml
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>roboseasy_state_estimator</name>
  <version>1.0.0</version>
  <description>RobosEasy 상태 추정 노드 패키지</description>
  <maintainer email="khw11044@gmail.com">RobosEasy</maintainer>
  <license>Apache-2.0</license>

  <!-- 빌드 도구 -->
  <buildtool_depend>ament_python</buildtool_depend>

  <!-- 런타임 의존 (실행 시 필요) -->
  <exec_depend>rclpy</exec_depend>
  <exec_depend>sensor_msgs</exec_depend>
  <exec_depend>roboseasy_msgs</exec_depend>

  <!-- 테스트 전용 -->
  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```

| 태그 | 용도 |
|------|------|
| `<depend>` | 빌드 + 런타임 모두 필요 (C++에서 흔함) |
| `<build_depend>` | 빌드 시에만 필요 |
| `<exec_depend>` | 실행 시에만 필요 (Python에서 흔함) |
| `<test_depend>` | 테스트 시에만 필요 |

---

## 5. setup.py (Python 패키지)

**규칙:** `data_files`에 launch/config 등록 / `entry_points`에 노드 실행파일 등록

```python
import os
from glob import glob
from setuptools import find_packages, setup

package_name = 'roboseasy_state_estimator'

setup(
	name=package_name,
	version='1.0.0',
	packages=find_packages(exclude=['test']),
	data_files=[
		('share/ament_index/resource_index/packages',
			['resource/' + package_name]),
		('share/' + package_name, ['package.xml']),
		# launch / config 파일을 install 공간으로 복사 (필수)
		(os.path.join('share', package_name, 'launch'),
			glob('launch/*.launch.py')),
		(os.path.join('share', package_name, 'config'),
			glob('config/*.yaml')),
	],
	install_requires=['setuptools'],
	zip_safe=True,
	maintainer='RobosEasy',
	maintainer_email='khw11044@gmail.com',
	description='RobosEasy 상태 추정 노드 패키지',
	license='Apache-2.0',
	entry_points={
		'console_scripts': [
			# {실행이름} = {모듈경로}:{함수명}
			'state_estimator_node = '
			'roboseasy_state_estimator.state_estimator_node:main',
		],
	},
)
```

> ⚠️ `launch/`·`config/` 파일을 `data_files`에 등록하지 않으면 `install/`로 복사되지 않아
> `ros2 launch`가 파일을 찾지 못한다. 새 런치/설정 파일을 추가할 때마다 확인한다.
>
> ⚠️ 실행 이름(`state_estimator_node`)은 `ros2 run {패키지} {실행이름}`에서 사용되므로 snake_case로 통일한다.

---

## 6. CMakeLists.txt (C++ 패키지)

**규칙:** C++17 / `ament_target_dependencies` 사용 / launch·config 설치 명시

```cmake
cmake_minimum_required(VERSION 3.8)
project(roboseasy_joint_controller)

if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif()
if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(roboseasy_msgs REQUIRED)

add_executable(joint_controller_node src/joint_controller.cpp)
target_include_directories(joint_controller_node PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)
ament_target_dependencies(joint_controller_node
  rclcpp sensor_msgs roboseasy_msgs)

install(TARGETS joint_controller_node
  DESTINATION lib/${PROJECT_NAME})

# launch / config 설치 (필수)
install(DIRECTORY launch config
  DESTINATION share/${PROJECT_NAME})

ament_package()
```

---

## 7. 인터페이스 (msg / srv / action)

**규칙:** 인터페이스는 **별도 패키지**(`roboseasy_msgs`)로 분리 / 파일명 PascalCase

노드 패키지 안에 msg를 두면 순환 의존이 생기기 쉽다. 인터페이스는 전용 패키지에 모은다.

```
roboseasy_msgs/
├── package.xml            # rosidl_default_generators 의존
├── CMakeLists.txt
├── msg/
│   ├── RobotState.msg
│   └── FsrState.msg
├── srv/
│   └── QueryState.srv
└── action/
    └── Walk.action
```

```
# ✅ GOOD — RobotState.msg (필드는 snake_case)
std_msgs/Header header
float64[] joint_positions
float64[] joint_velocities
bool is_stable

# ✅ GOOD — QueryState.srv (--- 로 요청/응답 구분)
string joint_name
---
float64 position
float64 velocity

# ✅ GOOD — Walk.action (goal / result / feedback)
float64 target_distance
---
bool is_success
---
float64 progress
```

| 항목 | 규칙 | 예시 |
|------|------|------|
| 파일명 | PascalCase | `RobotState.msg` |
| 필드명 | snake_case | `joint_positions`, `is_stable` |
| 상수 | UPPER_SNAKE_CASE | `uint8 MODE_STAND = 0` |

---

## 8. 토픽 / 서비스 / 액션 / 프레임 이름

**규칙:** `/roboseasy/{그룹}/{이름}` 계층 구조, 모두 snake_case

```
# ✅ GOOD — 토픽
/roboseasy/imu/data
/roboseasy/joint_states
/roboseasy/robot_state
/roboseasy/fsr/data

# ✅ GOOD — 서비스 / 액션
/roboseasy/query_state
/roboseasy/walk

# ✅ GOOD — TF 프레임 ID
base_link
l_foot_link
imu_link

# ❌ BAD
/RobosEasy/JointStates    # PascalCase X
/joint-states             # 하이픈 X
jointStates               # 네임스페이스 없음 + camelCase X
```

> 💡 **네임스페이스로 다중 로봇 대응:** 같은 노드를 여러 로봇에 띄울 때는
> 런치에서 `namespace='abo'`를 주어 `/abo/roboseasy/...`처럼 분리한다. 토픽을 하드코딩하지 않는다.

---

## 9. 런치 파일

**규칙:** `snake_case.launch.py` (Python 런치) / 파라미터는 YAML 파일로 분리

```python
# state_estimator.launch.py
import os

from ament_index_python.packages import get_package_share_directory
from launch import LaunchDescription
from launch_ros.actions import Node


def generate_launch_description() -> LaunchDescription:
	config = os.path.join(
		get_package_share_directory('roboseasy_state_estimator'),
		'config',
		'state_estimator.yaml',
	)

	return LaunchDescription([
		Node(
			package='roboseasy_state_estimator',
			executable='state_estimator_node',
			name='state_estimator_node',
			output='screen',
			parameters=[config],
		),
	])
```

> ⚠️ 파라미터 기본값을 런치 파일 안에 하드코딩하지 않는다. `config/*.yaml`로 분리해 재사용·수정을 쉽게 한다.
> 시스템 전체 실행은 `roboseasy_bringup` 패키지의 통합 런치(`bringup.launch.py`)에서 개별 런치를 `IncludeLaunchDescription`으로 조합한다.

---

## 10. 파라미터 YAML

**규칙:** `{노드이름}: → ros__parameters:` 계층 / 노드 이름과 일치

```yaml
# config/state_estimator.yaml
state_estimator_node:
  ros__parameters:
    control_hz: 100
    lipm_height: 0.75
    use_sim_time: false
    urdf_path: ''
```

> ⚠️ 최상위 키(`state_estimator_node`)는 런치의 `name` 및 노드의 `super().__init__('...')`과 **정확히 일치**해야 파라미터가 적용된다.

---

## 11. colcon 빌드 규칙

**규칙:** 항상 워크스페이스 루트에서 빌드 / `--symlink-install` 권장

```bash
# 전체 빌드 (워크스페이스 루트에서)
colcon build --symlink-install

# 특정 패키지만 빌드
colcon build --symlink-install --packages-select roboseasy_state_estimator

# 특정 패키지 + 의존 패키지까지
colcon build --symlink-install --packages-up-to roboseasy_bringup

# 빌드 후 반드시 source
source install/setup.bash
```

| 옵션 | 효과 |
|------|------|
| `--symlink-install` | Python·설정 파일을 심볼릭 링크로 설치 → 수정 후 재빌드 불필요 |
| `--packages-select` | 지정 패키지만 빌드 |
| `--packages-up-to` | 지정 패키지 + 그 의존성까지 빌드 |

> 💡 자주 쓰는 빌드/소스 명령은 alias로 등록해 통일한다 → [.bashrc / alias 가이드](roboseasy_bashrc_alias_guide.md)

---

## 12. .gitignore (워크스페이스)

**규칙:** `build/` · `install/` · `log/`는 커밋하지 않는다

```gitignore
# colcon 산출물
build/
install/
log/

# Python
__pycache__/
*.pyc
*.egg-info/

# 에디터
.vscode/
*.swp
```

---

## 13. 패키지 README

**규칙:** 각 패키지 루트에 `README.md` — 노드 · 토픽 · 파라미터 · 실행법 명시

```markdown
# roboseasy_state_estimator

로봇 상태 추정 노드 패키지.

## 노드
- `state_estimator_node`

## 구독 (Subscribe)
- `/roboseasy/imu/data` (sensor_msgs/Imu)
- `/roboseasy/joint_states` (sensor_msgs/JointState)

## 발행 (Publish)
- `/roboseasy/robot_state` (roboseasy_msgs/RobotState)

## 파라미터
- `control_hz` (int, 100)
- `lipm_height` (float, 0.75)

## 실행
`ros2 launch roboseasy_state_estimator state_estimator.launch.py`
```

---

*RobosEasy ROS2 Project & Workspace Convention v1.0 — ROS2 Humble + colcon(ament) 기반*
