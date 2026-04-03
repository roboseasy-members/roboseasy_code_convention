# RobosEasy C++ 코드 규칙서

> **기반 프레임워크:** ROS2 Humble &nbsp;|&nbsp; **언어 표준:** C++17 &nbsp;|&nbsp; **파생 출처:** RobosEasy 코드베이스

---

## 📋 규칙 요약표

| 대상 | 규칙 | 예시 |
|------|------|------|
| 클래스 / 구조체 | PascalCase | `StateEstimator`, `JointID` |
| 함수 / 메서드 | PascalCase | `TimerCallback`, `GetEndpointPose` |
| 멤버 변수 | snake_case + 후행 `_` | `imu_tare_request_`, `control_time_` |
| 로컬 변수 | snake_case | `latest_time`, `urdf_path` |
| **Boolean 변수** | **`is_` / `has_` 접두사** | `is_l_foot_contact_`, `has_velocity_state_interface` |
| 함수 파라미터 | snake_case | `node_name`, `joint_name` |
| enum 타입명 | PascalCase | `FilterType`, `Filter` |
| enum 값 (enum class) | PascalCase | `FilterType::Pos`, `FilterType::Vel` |
| enum 값 (일반 enum) | UPPER_SNAKE_CASE | `NO_FILTERING`, `LOW_PASS_FILTER` |
| static constexpr | snake_case | `joint_count`, `l_hip_y` |
| 매크로 (#define) | UPPER_SNAKE_CASE | `ALICE`, `ALICE_VERSION` |
| 네임스페이스 | lowercase_snake | `roboseasy`, `roboseasy_joint_controller` |
| 타입 별칭 (using) | PascalCase | `using Imu = sensor_msgs::msg::Imu` |
| 파일명 | snake_case | `roboseasy_state_estimator.hpp` |
| 헤더 가드 | UPPER_SNAKE_CASE + `_` | `ROBOSEASY_STATE_ESTIMATOR_HPP_` |

---

## 1. 클래스 / 구조체

**규칙:** PascalCase (대문자로 시작, 단어마다 대문자)

모든 클래스와 구조체는 PascalCase를 따른다.
약어도 첫 글자만 대문자로 처리하는 것이 원칙이나, 로봇명 등 고유명사는 전체 대문자를 허용한다.

```cpp
// ✅ GOOD
class StateEstimator : public rclcpp::Node {};
class RobosEasyJointController {};
struct JointID {};
class ALICE4State {};     // 로봇 고유명사
class JointFilter {};

// ❌ BAD
class state_estimator {}; // snake_case X
class stateEstimator {};  // camelCase X
class STATEESTIMATOR {};  // 전체 대문자 X
struct joint_id {};       // snake_case X
```

---

## 2. 함수 / 메서드

**규칙:** PascalCase — 동사로 시작하는 것을 권장

모든 멤버 함수와 자유 함수는 PascalCase를 사용한다.
동작을 명확히 표현하는 동사로 시작하는 것이 권장된다.

| 접두사 | 용도 |
|--------|------|
| `Get` / `Set` | 접근자 |
| `Is` / `Has` | bool 반환 |
| `Update` / `Refresh` | 갱신 |
| `Calculate` / `Compute` | 계산 |
| `Handle` | 이벤트 처리 |
| `Load` / `Save` | 입출력 |
| `Publish` / `Subscribe` | ROS2 |

```cpp
// ✅ GOOD
void TimerCallback();
void UpdateStateFromInterfaces();
bool ReadStateFromCommandInterfaces();
EndpointState GetEndpointPose(const std::string &name);
void HandleShutdownSignal(int signum);
void ImuCallback(const Imu::SharedPtr msg);
double GetTotalMass();

// ❌ BAD
void timer_callback();   // snake_case X
void timerCallback();    // camelCase X
void update();           // 동작 불명확
void cb();               // 약어 X
```

---

## 3. 멤버 변수

**규칙:** `snake_case_` (후행 밑줄 필수)

클래스 멤버 변수는 소문자 snake_case로 작성하고, 이름 끝에 `_`를 붙여 로컬 변수와 구분한다.
스마트 포인터, atomic 변수, 필터 객체 모두 동일 규칙을 따른다.

> ⚠️ **중요:** 후행 `_`는 멤버 변수임을 표시하는 핵심 규칙이다. 로컬 변수와의 구분이 즉시 가능해진다.

```cpp
// ✅ GOOD
private:
  int    odometry_mode_;
  double control_time_;
  std::atomic<bool>              imu_tare_request_;
  bool                           is_l_foot_contact_;
  std::unique_ptr<LowPassFilter> com_x_lpf_;
  rclcpp::TimerBase::SharedPtr   control_timer_;

// ❌ BAD
private:
  int    odometryMode;    // camelCase X
  double ControlTime;     // PascalCase X
  bool   m_isContact;     // m_ 헝가리안 X
  bool   isContact;       // 후행 _ 누락 X
```

---

## 4. 로컬 변수 / 함수 파라미터

**규칙:** `snake_case` (후행 밑줄 없음)

함수 내부의 로컬 변수와 함수 파라미터는 snake_case를 사용하되 후행 `_` 없이 쓴다.

```cpp
// ✅ GOOD
void ImuCallback(const Imu::SharedPtr msg)   // 파라미터
{
  double            time_tolerance = 0.1;    // 로컬 변수
  Imu::SharedPtr    imu_msg        = nullptr;
  std::string       urdf_path;
  rclcpp::Time      latest_time    = now();
}

// ❌ BAD
void ImuCallback(const Imu::SharedPtr Msg)   // 대문자 X
{
  double timeTolerance = 0.1;  // camelCase X
  double time_tolerance_;      // 후행 _ X (멤버변수로 오해)
  std::string p;               // 의미 없는 단문자 X
}
```

---

## 4+. Boolean 변수 네이밍 ⭐

**규칙:** 반드시 `is_` 또는 `has_` 접두사로 시작

`bool` 타입 변수는 반드시 `is_` 또는 `has_` 접두사로 시작한다.
이 규칙 덕분에 변수명을 보는 순간 **타입 선언 없이도** `true/false` 값을 가진다는 것을 즉시 알 수 있다.
**네이밍만으로 타입을 전달하는 가장 기본적인 가독성 규칙이다.**

### 접두사 선택 기준

| 접두사 | 사용 상황 | 질문 형태 |
|--------|-----------|-----------|
| `is_` | 상태 / 조건 여부 | "~인가? / ~한 상태인가?" |
| `has_` | 인터페이스 / 기능 보유 여부 | "~를 가지고 있는가?" |

```cpp
// ✅ GOOD — is_ (상태/조건)
// 멤버 변수 (후행 _ 포함)
bool is_l_foot_contact_;
bool is_r_foot_contact_;
bool is_active_;
bool is_initialized_;

// 로컬 변수 (후행 _ 없음)
bool is_valid        = true;
bool is_empty        = queue.empty();
bool is_linear_mode  = true;

// ✅ GOOD — has_ (인터페이스 보유)
// 멤버 변수
bool has_velocity_state_interface_;
bool has_acceleration_state_interface_;
bool has_effort_command_interface_;

// 로컬 변수
bool has_values          = true;
bool has_linear_actuator = false;
bool has_valid_data      = !queue.empty();

// ❌ BAD
bool left_foot_contact;  // is_ 누락 — bool인지 불분명
bool velocity_interface; // has_ 누락 — bool인지 불분명
bool linear;             // 타입 불분명
bool bIsContact;         // 헝가리안 표기 b X
bool IsContact;          // PascalCase X (함수처럼 보임)
bool check_contact;      // 동사 시작 X
bool contact_status;     // _status 접미사 X
```

> 💡 **한 눈에 알아보는 예시:**
> ```cpp
> if (is_l_foot_contact_ && has_velocity_state_interface_)
> ```
> 변수명만 봐도 둘 다 `bool`임을 즉시 인식할 수 있다.

---

## 5. 열거형 (Enum)

**규칙:** 타입명은 PascalCase, 값은 enum 종류에 따라 다름

`enum class` 값은 PascalCase, 일반 `enum` 값은 UPPER_SNAKE_CASE.
타입 안전성을 위해 `enum class` 사용을 권장한다.

```cpp
// ✅ GOOD — enum class (권장)
enum class FilterType
{
  Pos,     // 위치 필터
  Vel,     // 속도 필터
  Torque,  // 토크 필터
  Count
};

// 사용
FilterType t = FilterType::Vel;

// ✅ GOOD — 일반 enum
enum Filter
{
  NO_FILTERING    = 0,
  LOW_PASS_FILTER = 1,
  ONE_EURO_FILTER = 2,
};

// 사용
Filter f = LOW_PASS_FILTER;
```

---

## 6. 상수 / constexpr

**규칙:** `static constexpr` 멤버는 snake_case / 전역 상수는 UPPER_SNAKE_CASE

```cpp
// ✅ GOOD — 구조체 내 static constexpr
struct JointID
{
  static constexpr uint8_t fixed_base  = 0;
  static constexpr uint8_t l_hip_y     = 11;
  static constexpr uint8_t r_knee_p    = 20;
  static constexpr uint8_t joint_count = 40;
};

// ✅ GOOD — 전역 constexpr
constexpr double GRAVITY_ACCEL = 9.80665;
```

---

## 7. 매크로 / 전처리기 (#define)

**규칙:** UPPER_SNAKE_CASE — 조건부 컴파일 플래그에만 사용

매크로는 대문자 UPPER_SNAKE_CASE로 정의한다.
단순 상수에는 사용하지 않고 `constexpr`로 대체한다.

```cpp
// ✅ GOOD
#define ALICE         1
#define ALICE_VERSION 2

#if ALICE
  // Alice 로봇 전용 코드
#endif

// ❌ BAD
#define gravity    9.80665  // 소문자 X
#define Alice      1        // PascalCase X
#define MAX_JOINTS 40       // constexpr로 대체할 것
```

---

## 8. 네임스페이스

**규칙:** `lowercase_snake_case` — 패키지명과 동일하게

닫는 괄호에 반드시 `// namespace 이름` 주석을 붙인다.

```cpp
// ✅ GOOD
namespace roboseasy
{
  class StateEstimator {};
} // namespace roboseasy

namespace roboseasy_joint_controller
{
  class RobosEasyJointController {};
} // namespace roboseasy_joint_controller
```

---

## 9. 타입 별칭 (using)

**규칙:** PascalCase — ROS2 메시지 타입 단축명에 주로 사용

긴 ROS2 메시지 타입은 `using`으로 별칭을 선언해 코드 가독성을 높인다.

```cpp
// ✅ GOOD
using Imu        = sensor_msgs::msg::Imu;
using JointState = sensor_msgs::msg::JointState;
using RobotState = roboseasy_state_msgs::msg::RobotState;
using FsrState   = roboseasy_state_msgs::msg::FsrState;
using RobotPose  = alice_walking_module_msgs::msg::RobotPose;
```

---

## 10. 파일명

**규칙:** `lowercase_snake_case` + 확장자 (`.hpp` / `.cpp`)

```
# ✅ GOOD
roboseasy_state_estimator.hpp
roboseasy_state_estimator.cpp
roboseasy_joint_controller.hpp
alice4_state_estimator.hpp

# ❌ BAD
StateEstimator.hpp       # PascalCase X
stateEstimator.h         # .h 확장자 X
RobosEasyController.HPP   # 대문자 확장자 X
```

---

## 11. 헤더 가드

**규칙:** `#ifndef` 방식 — `PACKAGE_FILENAME_HPP_` 형식 (후행 `_` 포함)

```cpp
// 파일: roboseasy_state_estimator.hpp
#ifndef ROBOSEASY_STATE_ESTIMATOR_HPP_
#define ROBOSEASY_STATE_ESTIMATOR_HPP_

// ... 헤더 내용 ...

#endif // ROBOSEASY_STATE_ESTIMATOR_HPP_

// 파일: alice4_state.hpp
#ifndef ALICE4STATE_HPP
#define ALICE4STATE_HPP
// ...
#endif // ALICE4STATE_HPP
```

> `#pragma once`도 허용되나, 팀 내 통일 필요. 현 코드베이스는 `#ifndef` 방식이 표준.

---

## 12. 인클루드 순서

**규칙:** 표준 라이브러리 → ROS2 → 외부 라이브러리 → 자체 헤더

각 그룹 사이에 빈 줄을 둔다.

```cpp
// 1) 표준 라이브러리 (C++ STL)
#include <atomic>
#include <string>
#include <vector>
#include <memory>
#include <mutex>

// 2) ROS2 메시지 / 코어
#include <rclcpp/rclcpp.hpp>
#include <sensor_msgs/msg/imu.hpp>
#include <nav_msgs/msg/odometry.hpp>

// 3) 외부 라이브러리 (Eigen, Pinocchio 등)
#include <Eigen/Dense>
#include <pinocchio/parsers/urdf.hpp>

// 4) 자체(패키지 내) 헤더
#include "roboseasy_toolbox/basic_tools.hpp"
#include "roboseasy_math/math_tool.hpp"
```

---

## 13. 스마트 포인터

**규칙:** `unique_ptr` 기본 / `shared_ptr`는 공유 소유권이 필요한 경우만

Raw 포인터 사용을 금지하고 스마트 포인터를 사용한다.

```cpp
// ✅ GOOD — 단독 소유: unique_ptr
std::unique_ptr<ALICE4State>    alice4_state_;
std::unique_ptr<LowPassFilter>  com_x_lpf_;
std::unique_ptr<OneEuroFilter>  dcm_x_oef_;

// 생성 — make_unique 사용
alice4_state_ = std::make_unique<ALICE4State>();
com_x_lpf_    = std::make_unique<LowPassFilter>(clock, cutoff);

// ROS2 공유 포인터
rclcpp::Publisher<RobotState>::SharedPtr state_publisher_;
rclcpp::Subscription<Imu>::SharedPtr     imu_subscriber_;
```

---

## 14. 조건부 컴파일

**규칙:** `#if` / `#endif` — 로봇 버전 분기에 사용, `#endif`에 주석 필수

```cpp
#define ALICE 1  // 파일 상단 또는 CMake에서 정의

#if ALICE
  std::unique_ptr<ALICE4State> alice4_state_;
  void UpdateJointsFromJointState(ALICE4State &state,
                                  const JointState &js);
#endif // ALICE

// 구현부에서도 동일하게
#if ALICE
  alice4_state_ = std::make_unique<ALICE4State>();
#endif // ALICE
```

---

## 15. Doxygen 주석

**규칙:** 헤더 파일에서 `/** */` 블록 주석 — `@brief`, `@param`, `@return` 필수

헤더 파일의 모든 public 클래스, 구조체, 함수에 Doxygen 주석을 작성한다.

```cpp
/**
 * @file roboseasy_state_estimator.hpp
 * @brief 로봇 상태 추정기 클래스 정의
 */

/**
 * @class StateEstimator
 * @brief 각종 센서 데이터를 받아 상태를 추정하는 클래스
 */
class StateEstimator : public rclcpp::Node
{
public:
  /**
   * @brief 생성자
   * @param node_name ROS2 노드명 (기본값: "roboseasy_state_estimator")
   */
  explicit StateEstimator(const std::string &node_name
                          = "roboseasy_state_estimator");

  /**
   * @brief ZMP 계산
   * @return 반환값 없음
   */
  void CalculateZMP();

  /**
   * @brief 전체 질량 계산
   * @return 로봇 전체 질량 (kg)
   */
  double GetTotalMass();
};
```

---

## 16. 인라인 주석

**규칙:** 구현부 한글 주석 허용 — 알고리즘/좌표계/물리 의미 설명에 활용

구현 파일(.cpp) 내 인라인 주석은 한국어를 허용한다.
임시 코드에는 `// 일단주석` 등 추후 정리 예정임을 명시한다.

```cpp
// (1) pitch *= -1 처리
pitch *= -1.0;

// (2) 점 B를 "한 번에" 계산
// p_b_x = (-r + 0.064)*sin(pitch) - 0.05*cos(pitch)
const double p_b_x = (-r + 0.064) * std::sin(pitch)
                     - 0.05 * std::cos(pitch);

// (3) BC = BO + OC (벡터 합산)
const double bc_x = bo_x + oc_x;

// 일단주석 — 추후 position interface 활성화 시 해제
// CheckAndAssign(has_position_command_interface, ...);
```

---

## 17. 섹션 구분자 (ASCII Art)

**규칙:** 대형 구현 파일에서 기능 그룹을 ASCII 배너로 구분

500줄 이상의 구현 파일에서 기능 단위를 ASCII Art 배너로 구분하면 가독성이 높아진다.
`#endif`에 반드시 주석을 달고, 사용 플래그는 파일 상단에 정의한다.

```cpp
/*********************************************************
    _____ _ _ _
   |  ___(_) | |_ ___ _ __
   | |_  | | | __/ _ \ '__|
   |  _| | | | ||  __/ |
   |_|   |_|_|\__\___|_|

  *********************************************************/

/*********************************************************
 __     ___                 _
 \ \   / (_)___ _   _  __ _| |
  \ \ / /| / __| | | |/ _` | |
   \ V / | \__ \ |_| | (_| | |
    \_/  |_|___/\__,_|\__,_|_|

  *********************************************************/
```

> **권장 사용 시점:** 클래스 하나의 구현이 여러 `.cpp`로 분할되거나, 한 파일 내에서
> 초기화 / 콜백 / 계산 / 퍼블리시 섹션이 뚜렷이 구분될 때.

---

*RobosEasy C++ Coding Convention v1.0 — RobosEasy 코드베이스 기반*
