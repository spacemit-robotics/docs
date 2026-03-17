# SpacemiT Robotis 工程代码规范

本文档适用于整个 spacemit_robotis 工程，约束 C、C++、Python、Shell 四种语言的命名、格式与工程架构。

## 1. 通用规范

### 1.1 字符与编码

- 源文件编码：统一使用 UTF-8（无 BOM）。
- 换行符：统一使用 LF（\n），禁止 CRLF。
- 行尾：文件末尾保留一个换行符。

### 1.2 工程架构原则

- 分层清晰：按 application / components / middleware / tools / scripts 等划分，上层依赖下层，避免反向或跨层循环依赖。
- 模块化：功能按模块拆分，单模块职责单一，接口通过头文件或明确 API 暴露。
- 可构建性：根目录或各子工程提供统一入口（如 CMakeLists.txt、Makefile、setup.py / pyproject.toml），一键构建/测试。
- 可追溯：关键设计决策、模块职责在 README 或文档中说明，便于新人与后续维护。

### 1.3 代码格式要求

| 项目       | 规则                                                                 |
|------------|----------------------------------------------------------------------|
| 缩进       | 统一使用 4 个空格，禁止 Tab（Shell 脚本除外，可用 Tab）。            |
| 行宽       | 建议不超过 100 字符；超长行在合适位置换行（运算符后、逗号后等）。     |
| 空行       | 函数间空 1 行；逻辑块间可空 1 行分隔；文件末尾保留 1 个换行符。       |
| 行尾空格   | 禁止行尾有多余空格或 Tab。                                            |

### 1.4 工程目录结构示例

典型项目结构：

```
spacemit_robotis/
├── application/                        # 应用层（上层业务逻辑，依赖 components 和 middleware）
│   ├── native/                         # 原生 C/C++ 应用（不依赖 ROS2）
│   │   └── omni_agent/                 # 产品名，按实际产品命名
│   │       ├── src/
│   │       │   └── voice_chat.cpp      # C++ 统一用 .cc/.cpp 扩展名
│   │       ├── include/
│   │       │   └── voice_pipeline.hpp  # 头文件名与功能对应
│   │       └── CMakeLists.txt          # 每个可构建模块提供独立构建入口
│   └── ros2/                           # ROS2 应用
│       └── mars_robot/                 # 产品名（如 mars_robot、lekiwi 等）
│           └── hmi_display/            # 产品下的功能包，snake_case
│               ├── src/
│               │   └── hmi_display_node.cc # 节点文件名与包名对应
│               ├── include/
│               │   └── HmiDisplayManager.h # 头文件用 PascalCase（类名）
│               ├── package.xml         # ROS2 包描述（依赖、维护者等）
│               └── CMakeLists.txt
├── components/                         # 组件层（可复用模块，被上层依赖）
│   └── peripherals/                    # 按功能域分类（外设类）
│       └── motor/                      # 单一职责：电机驱动抽象
│           ├── include/                # 对外头文件，公开 API 边界
│           │   └── motor.h             # 头文件名与模块名一致
│           ├── src/                    # 内部实现，不对外暴露
│           │   ├── motor_core.c        # C 源文件用 .c，snake_case
│           │   └── drivers/            # 子目录按驱动类型拆分
│           ├── test/                   # 测试与源码同级，便于维护
│           │   └── test_motor_uart.c   # test_ 前缀标明测试文件
│           └── CMakeLists.txt
└── middleware/                         # 中间件层（不依赖 application 和 components）
    └── ros2/
        └── perception/                 # 按功能域分类（感知类）
            └── face_detection/         # 单一职责：人脸检测
                ├── src/
                │   └── face_detection_node.cc
                ├── include/
                │   └── face_detection_node.h   # 头文件与源文件除扩展名外一致
                ├── launch/
                │   └── face_detection_launch.py # launch 文件用 snake_case，避免点号分隔
                ├── package.xml
                └── CMakeLists.txt
```

依赖关系：

- application 依赖 components 和 middleware
- components 可依赖 middleware，但不依赖 application
- middleware 不依赖 components 或 application
- 同层模块间避免循环依赖

## 2. 文件命名规范

### 2.1 通用约定

| 项目     | 规则 |
|----------|------|
| 字符集   | 仅使用 小写字母、数字、下划线，必要时可使用连字符（如脚本 run-tests.sh）。 |
| 禁止     | 空格、中文、特殊符号；多个连续下划线。 |
| 长度     | 建议单个文件名不超过 40 个字符，过长时用下划线分词。 |

### 2.2 C（Linux Kernel 风格）

| 类型   | 命名方式       | 示例 |
|--------|----------------|------|
| 源文件 | snake_case.c | motor_core.c, pid_controller.c |
| 头文件 | snake_case.h | motor.h, config.h |

- 实现与头文件除扩展名外命名一致。单元测试可用 *_test.c 或置于 tests/。

### 2.3 C++（Google C++ Style）

| 类型   | 命名方式           | 示例 |
|--------|--------------------|------|
| 源文件 | 小写 + 下划线，扩展名 .cc 或 .cpp | motor_core.cc, pid_controller.cpp |
| 头文件 | 小写 + 下划线，扩展名 .h  | motor_core.h, config.h |

- 文件名需具描述性（如 http_server_logs.h 而非 logs.h）。测试文件可用 *_test.cc / *_test.cpp 或放 tests/。

### 2.4 Python

| 类型       | 命名方式           | 示例 |
|------------|--------------------|------|
| 模块/脚本  | snake_case.py    | motor_core.py, calibration_tool.py |
| 包目录     | snake_case（目录名） | robot_utils/, drivers/ |
| 测试       | test_*.py 或 *_test.py | test_motor.py |

- 包内须有 __init__.py（Python 3 也可用 namespace package，但需统一约定）。

### 2.5 Shell

| 类型   | 命名方式           | 示例 |
|--------|--------------------|------|
| 脚本   | snake_case.sh 或 kebab-case.sh | build_firmware.sh, run-tests.sh |
| 库/公共函数 | 可加后缀区分 | common_lib.sh, utils.sh |

- 可执行脚本建议在 shebang 后注明用途（注释一行）。

### 2.6 文件取名建议

- 按职责/模块取名：文件名应反映该文件的主要职责或所属模块。
  - 正例：motor_core.c、pid_controller.cc、calibration_tool.py、uart_protocol.h。
  - 反例：driver.c（未说明哪类驱动）、stuff.py（无具体含义）、m1.c（无法从名字看出模块）。
- 一词一主题：一个文件聚焦一类功能，名字用 1～3 个下划线连接的词即可。
  - 正例：sensor_imu.c、robot_kinematics.cc。
  - 反例：onnx.c、sim.h、misc.py、others.h（过于泛化，不利于定位与维护）；motor_core_pid_velocity_controller_config.c（过长，应拆文件或适当缩写）。
- 可读性优先：宁可略长、表意清晰，也不要为短而用难懂的缩写；团队内已约定的缩写（如 msg、cfg、buf）可在项目内统一使用。
  - 正例：config_parser.py、msg_queue.h。
  - 反例：cfg_prsr.py（自创缩写）、drv_mtr_pwm.c（过度缩写）、x.c、a1.h（无信息量）。
- 测试/工具类：测试文件用 *_test / test_* 标明；脚本用动词或动作短语，便于一眼看出用途。
  - 正例：motor_core_test.cc、test_pid.py、build_firmware.sh、run_tests.sh、flash_device.sh。
  - 反例：test.c（未说明测什么）、run.sh（未说明运行什么）、do_it.py（用途不明）。
- 风格与字符：仅使用小写字母、数字、下划线；扩展名与语言规范一致（C++ 用 .cc 或 .cpp，C 用 .c 等）。
  - 反例：MotorDriver.C（大写或错误扩展名）、motor-driver.c（连字符在 C/C++ 文件命名中不推荐）、电机驱动.c（中文）。

## 3. 函数与接口命名

### 3.1 C（Linux Kernel 风格）

| 对象       | 风格               | 示例 |
|------------|--------------------|------|
| 函数       | snake_case       | motor_init(), pid_compute() |
| 宏/常量    | UPPER_SNAKE_CASE | MAX_PWM, MOTOR_POLE_PAIRS |
| 类型/枚举  | snake_case，可加 _t 后缀 | motor_state_t, result_t |
| 静态/内部  | static + snake_case | static void pwm_impl_update() |

- 对外 API 建议带模块前缀：motor_*, sensor_*。命名要有描述性，避免含糊缩写。
- 参考：[Linux kernel coding style](https://www.kernel.org/doc/html/latest/process/coding-style.html)。

### 3.2 C++（Google C++ Style）

| 对象         | 风格                 | 示例 |
|--------------|----------------------|------|
| 函数/方法    | PascalCase（首字母大写） | GetVelocity(), ComputePid() |
| 类/结构体/类型 | PascalCase         | MotorDriver, PidController |
| 变量/参数/成员 | snake_case         | motor_id, speed_, kp_ |
| 宏/常量      | kConstantName 或 UPPER_SNAKE | kMaxRetry, MAX_BUFFER |
| 命名空间     | 小写（可加下划线）   | robot::motor |

- 文件名小写+下划线，类型与函数用 PascalCase，变量用 snake_case。避免缩写，名称应对读者友好。
- 参考：[Google C++ Style Guide](https://google.github.io/styleguide/cppguide.html)。

### 3.3 Python

| 对象       | 风格             | 示例 |
|------------|------------------|------|
| 函数/方法  | snake_case     | get_velocity(), compute_pid() |
| 类名       | PascalCase     | MotorDriver, PidController |
| 常量       | UPPER_SNAKE_CASE | MAX_RETRY, DEFAULT_BAUDRATE |
| 变量/参数  | snake_case     | motor_id, timeout_sec |
| 私有       | 单下划线前缀     | _internal_helper() |

- 遵循 [PEP 8](https://peps.python.org/pep-0008/) 命名建议。

### 3.4 Shell

| 对象     | 风格           | 示例 |
|----------|----------------|------|
| 函数     | snake_case   | build_firmware(), run_tests() |
| 变量     | UPPER_SNAKE_CASE（全局/环境）或 lower_snake_case（局部） | BUILD_DIR, local_count |
| 只读/常量 | readonly + 大写命名 | readonly CONFIG_PATH="/etc/app" |

### 3.5 函数与变量取名建议

#### 3.5.1 函数名（动宾或“动词+对象”）

- 做一件事的函数用动词开头，后面跟操作对象或结果含义。
- 正例：motor_init()、pid_compute()、get_velocity()、set_target_speed()、uart_send_byte()。
- 反例：do_stuff()、handle()（未说明处理什么）、process()（过于泛化）、func1()、run()（含义不清）。

返回布尔的函数用 is_* / has_* / can_*（C/Python）或 Is*() / Has*()（C++）。

- 正例：is_ready()、has_error()、CanRetry()、is_motor_enabled()。
- 反例：check()（未说明返回含义）、flag()、get_status()（若返回布尔，宜用 is_*/has_* 更直观）。

取数用 get_*/Get*()，设值用 set_*/Set*()，创建/初始化用 create_*/init_*/Init()；C 对外 API 建议带模块前缀。

- 正例：motor_start()、sensor_read()、get_encoder_count()。
- 反例：start()（易重名）、read()（未说明读什么）、m_init()（模块前缀不清晰时不如 motor_init()）。

#### 3.5.2 变量名（名词或“名词+修饰”）

- 变量表示“是什么”，用名词或“名词_修饰”；单位可放末尾。
- 正例：motor_id、target_speed、error_count、timeout_sec、pwm_freq_hz、buffer_size。
- 反例：temp、tmp、data、value（无具体含义）；x、a、n（除循环下标外过于简短）；timeout（若单位重要，宜用 timeout_sec 或 timeout_ms）。
- 循环下标等极短作用域可用 i、j、k；其他变量避免单字母（数学/物理常数如 e、pi 等除外）。

#### 3.5.3 常量/宏（表意清晰）

- 宏和全局常量名应表达“含义”而非“数值”；魔数用命名常量替代。
- 正例：MAX_RETRY_COUNT、DEFAULT_BAUDRATE、MOTOR_POLE_PAIRS、UART_TX_BUF_SIZE。
- 反例：MAX（未说明什么的上限）、N、NUM（过于泛化）；在代码中直接写 100、0x0800 等魔数而不定义常量。

#### 3.5.4 避免的写法（反例汇总）

| 问题           | 反例 | 正例或说明 |
|----------------|------|------------|
| 含糊缩写       | buf2、tmp1、data2、cve() | calc_velocity_from_encoder()、rx_buffer |
| 与关键字/类型混淆 | 变量名 class、return、int、new | 避免与语言关键字或类型名相同 |
| 同一概念多种写法 | 混用 index/idx、message/msg、configuration/config | 项目内统一择一 |
| 中英文混写或拼音 | dianji_sudu（电机速度） | 使用 motor_speed；标识符中不用中文 |

## 4. 注释规范

### 4.1 通用原则

- 注释说"为什么"，代码说"做什么"：代码本身应清晰表达逻辑，注释解释意图、原因、约束、注意事项。
- 保持同步：代码变更时同步更新注释，过时注释比没有注释更糟。
- 避免废话：不写 i++; // i 加 1 这类无信息量的注释。
- 关键位置必注释：复杂算法、非常规写法、性能优化、已知限制、TODO/FIXME。

### 4.2 文件头注释

每个源文件（.c/.cc/.py/.sh）开头应有文件头注释，说明：

- 文件用途/模块功能
- 作者/维护者（可选）
- 创建/修改日期（可选，Git 已记录可省略）
- 许可证信息（若适用）

C/C++ 示例：

```c
/**
 * @file motor_core.c
 * @brief BLDC motor driver implementation for Robotis Dynamixel protocol.
 *
 * This module provides low-level control for BLDC motors via UART,
 * including position/velocity/torque modes and PID tuning.
 */
```

Python 示例：

```python
"""
Motor driver module for Robotis Dynamixel servos.

Provides high-level API for motor control, including:
- Position/velocity/torque control
- Parameter read/write
- Bulk read/write for multi-motor sync
"""
```

### 4.3 函数/方法注释

对外 API、复杂函数必须有注释，说明：

- 功能简述
- 参数含义（类型、取值范围、可否为 NULL）
- 返回值（含义、错误码）
- 副作用/线程安全性/调用约束（若适用）

C 示例（Doxygen 风格）：

```c
/**
 * @brief Initialize motor driver with specified ID.
 *
 * @param motor_id Motor ID (1-253), must be unique on the bus.
 * @param baudrate Communication baudrate (9600, 57600, 1000000, etc.).
 * @return 0 on success, -1 on invalid ID, -2 on UART init failure.
 *
 * @note This function is NOT thread-safe. Call before starting worker threads.
 */
int motor_init(uint8_t motor_id, uint32_t baudrate);
```

C++ 示例：

```cpp
/**
 * @brief Compute PID output based on current error.
 *
 * @param error Current error (setpoint - measured).
 * @param dt Time delta since last call (seconds).
 * @return Control output (clamped to [-max_output, max_output]).
 */
double PidController::Compute(double error, double dt);
```

Python 示例（Google 风格）：

```python
def set_target_position(self, motor_id: int, position: float, timeout: float = 1.0) -> bool:
    """Set target position for specified motor.

    Args:
        motor_id: Motor ID (1-253).
        position: Target position in radians.
        timeout: Command timeout in seconds (default 1.0).

    Returns:
        True if command succeeded, False on timeout or error.

    Raises:
        ValueError: If motor_id is out of range.
    """
```

### 4.4 行内注释

- 复杂逻辑：算法关键步骤、非显而易见的判断条件。
- 魔数说明：delay_ms = 50;  // Wait for sensor stabilization。
- 临时方案：// TODO: Replace with async I/O after driver upgrade。
- 已知问题：// FIXME: Race condition when motor_count > 10。

示例：

```c
// Clamp PWM to safe range to prevent motor damage
if (pwm > MAX_SAFE_PWM) {
    pwm = MAX_SAFE_PWM;
}

// Use bit shift for fast division by 8 (compiler may optimize anyway)
int result = value >> 3;
```

### 4.5 TODO/FIXME/NOTE 标记

- TODO：待完成的功能或优化。
- FIXME：已知缺陷，需修复。
- NOTE：重要提示或说明。
- 格式：// TODO(author): description 或 // FIXME: description。

## 5. 对外头文件的要求

对外头文件指被其他模块或上层应用 #include 使用的头文件，是模块的公开 API 边界。以下要求同时适用于 C 与 C++ 工程。

### 5.1 内容与职责

- 只暴露对外接口：仅包含其他模块/应用需要使用的函数声明、类型定义（含结构体、枚举）、常量/宏。内部实现细节、未文档化的辅助类型、调试用宏等不得放入对外头文件。
- 不包含实现：不在头文件中写函数体（除 C++ 中确需的 inline/模板实现；若放在头文件中，需在文档中说明）。实现一律放在 .c / .cc 中。
- 接口稳定：对外头文件中的 API 变更会影响所有调用方，修改（删除、签名变更、语义变更）需谨慎，必要时通过版本号、命名空间或新头文件名做兼容。

### 5.2 头文件保护与包含

- 防重复包含：统一使用 #ifndef / #define / #endif 方式做 Include Guard。
  - C / Kernel 风格示例：#ifndef PROJECT_MODULE_XXX_H / #define / #endif。
  - C++ / Google 风格示例：#ifndef FOO_BAR_BAZ_H_ / #define / #endif。
- 尽量不 include 其他头文件：对外头文件中不要包含本工程其他头文件或第三方库头文件，避免调用方在包含该头时被拖入传递依赖。仅允许在确有必要时包含系统标准头文件（如 <stdint.h>、<stddef.h>、<stdbool.h> 等）。
- 用前向声明代替 include：若接口里只用到指针、引用等不完整类型，用前向声明即可，无需包含对应头文件；完整类型定义或实现所需的 #include 一律放在 .c / .cc 中。
- 包含顺序（若确有系统头）：仅包含系统头时，按字母序或项目约定排列即可。

### 5.3 C/C++ 混合与 C 链接

- 被 C++ 包含的 C 头文件：若头文件声明的是 C 接口且会被 C++ 代码包含，必须用 #ifdef __cplusplus + extern "C" 包裹声明，保证 C++ 侧按 C 链接方式解析符号。
- 纯 C++ 头文件：不暴露 C 接口时无需 extern "C"；若同一头中既有 C 接口又有 C++ 类/模板，仅对 C 接口部分使用 extern "C"。
- 命名：对外 C 接口命名保持 C 风格（snake_case），不在对外头文件中暴露仅 C++ 的语法（如 class、模板、异常）给 C 调用方。

### 5.4 放置与引用方式

- 放置位置：对外头文件建议集中在模块的 include/ 目录或工程统一的公共 include 目录，与实现（src/）分离，便于发布与引用。
- 引用方式：调用方通过 #include "module/xxx.h" 或工程约定的 include 路径引用，避免依赖实现目录内的相对路径；同一工程内引用风格（"..." 与 <...> 的用途）在项目内统一约定。

### 5.5 文档与注释

- 对外 API 须有注释：头文件中对外提供的函数、类型、重要常量/宏应有简短说明，包括：用途、参数含义、返回值（含错误码）、线程/调用约束（若适用）。
- 注释风格：项目内统一（如 Doxygen、Kernel doc 格式等），便于生成 API 文档；至少保证读者能看懂”何时用、怎么用、注意什么”。

## 6. Git 提交规范（分支命名 + Commit Message）

为保证代码历史可追溯、清晰、可读，约定代码提交规范。主要约束如下：

- 合并分支命名：新建分支命名统一为 <type>/<desc>（见 6.3 的 type 列表）。
- Commit 信息：使用业界通用的 Conventional Commits（见 6.4）。
- 平台：以 GitHub 组织仓库为准（https://github.com/spacemit-robotics），通过提交 Pull Request（PR） 合入。

### 6.1 本地开发流程

（1）更新主分支，同步最新代码

```bash
# 先清理本地的分支缓存与远程分支保持一致
git remote prune origin

# 切换到主分支
git checkout robot-dev
git pull origin robot-dev
```

（2）创建新分支，提交 commit

```bash
git checkout -b <type>/<desc>
# 例：git checkout -b feat/add-yolo-int8

git add ...
git commit -s
```

- 分支命名要求：<type>/<desc>，其中 type 见 6.3，desc 用小写英文+短横线/下划线描述意图（例如 add-yolo-int8）。
- commit 信息要求：见 6.4（Conventional Commits）。

### 6.2 推送与合入前的同步（rebase）

（1）同步远程，防止别人期间已合入

```bash
# 1. 获取远程最新代码 (不合并，只下载)
git fetch origin robot-dev

# 2. 执行 rebase：把本分支提交更新到远程 robot-dev 的最顶端
git rebase origin/robot-dev
```

（2）推送到远程分支并发起 MR/PR

```bash
git push origin <type>/<desc>
# 例：git push origin feat/add-yolo-int8
```

推送后在 GitHub 上发起 Pull Request（PR），目标分支选择 robot-dev。

### 6.3 分支类型（type）列表

<type> 只能从以下常用类型中选择（可按团队需要裁剪/扩展）：

| type | 含义 | 示例分支 |
|------|------|----------|
| feat | 新功能 | feat/add-yolo-int8 |
| fix | 修复缺陷 | fix/motor-timeout |
| perf | 性能优化 | perf/reduce-copy |
| refactor | 重构（不新增功能/不修 bug） | refactor/simplify-pipeline |
| docs | 文档变更 | docs/update-coding-standards |
| test | 测试相关 | test/add-uart-tests |
| build | 构建/编译系统 | build/cmake-options |
| ci | CI 配置 | ci/clang-format-check |
| chore | 杂项（不改业务代码） | chore/cleanup-scripts |
| revert | 回滚 | revert/bad-change |

### 6.4 Commit Message（Conventional Commits）

Commit 规范使用 Conventional Commits 标准，参考：https://www.conventionalcommits.org/en/v1.0.0/。

#### 6.4.1 核心原则

- 一条 commit 只做一件事：不要把修复问题、整理代码格式等不同目的混在同一条提交。
- 至少写 Header：复杂变更尽量补充 Body；需要关联问题时写 Footer。
- 禁止提交大文件/二进制/构建临时文件：如 .o、模型文件、build/、__pycache__/、.vscode/ 等。

#### 6.4.2 提交格式

```text
<type>[optional scope]: <description>

[optional body]

[optional footer(s)]
```

- 冒号后必须有一个空格。
- **Header**（第一行）：必填；**建议 ≤50 字符**，最多不超过 72 字符（业界 50/72 规则，便于 `git log --oneline` 与 GitHub 等界面完整展示）；与 Body 之间空一行。
- **Body**（选填）：说明“为什么改、怎么改”，适合复杂改动；**每行建议 72 字符换行**（与 Git 缩进后 80 列终端兼容）。
- **Footer**（选填）：关联 issue/需求/缺陷单号（如 `Fixes #123`）；**每行建议 72 字符**（与 Body 一致，便于终端与邮件补丁阅读）。

#### 6.4.3 Header 说明

- <type>（必填）：提交类型，只能从 6.3 列表中选择。
- <scope>（选填）：影响范围（本仓库内的模块/目录/子系统/关注点）。scope 应该能帮助读者在“同一仓库内”快速定位改动范围；不要机械地写仓库名本身。
  - 例：在 monorepo 可写 driver、llm、manifest 等；在 motor 仓库可写 dm、feetech、protocol、hal、tools 等（按实际影响面选择）。
  - 如果本仓库只有一个模块、scope 长期固定不变（例如总是 motor），建议省略 scope，直接写 `fix: ...` / `feat: ...`。
- <description>（必填）：简短说明做了什么，使用祈使句/动词开头更清晰（如 support ... / fix ... / split ...）；整行 Header 建议 ≤50 字符（包含 type、scope、description），最多不超过 72 字符。

#### 6.4.4 Body 说明

用于描述"为什么改、怎么改"，适合复杂改动；简单的 fix 或 docs 可省略。建议分点说明。

#### 6.4.5 Footer 说明

用于关联 issue/需求/缺陷单号。例如：

- Fixes #6628002552

无关联则不填；如有必须填写。

#### 6.4.6 提交示例

（1）新增功能（feat）

```text
feat(streaming): support Llama3 streaming generation

Implement token-by-token streaming output for Llama3.

Changes:
- Add streaming generator in Python wrapper
- Update C++ decode interface

Fixes #6628002552
```

（2）修复 Bug（fix）

```text
fix(queue): fix timeout when batch_size > 1

Previously, large batch sizes caused the command queue to overflow, leading to
timeouts. Now we split large batches into smaller chunks.
```

（3）杂项（chore）

```text
chore(repo): split model_zoo into independent repos

Refactor model_zoo structure to decouple cv, tts, and stt modules. This allows
independent version control for each modality.

Changes:
- Remove components/model_zoo
- Add components/model_zoo/cv
- Add components/model_zoo/tts
- Add components/model_zoo/stt
```

（4）scope 可省略（例如仓库的影响范围单一）

```text
fix: avoid crash when config file is empty

When the config file is empty, fall back to defaults instead of crashing.
```

（5）scope 表达本仓库内影响范围（示例：motor 仓库）

```text
fix(dm): clamp goal position to servo limits

Prevent invalid goal positions from being sent to the device.
```