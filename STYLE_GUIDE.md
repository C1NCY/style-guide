# Robotics Code Style Guide

Version 1.0.0 (see [CHANGELOG.md](CHANGELOG.md)). Applies to all team robots.

Based on the Google C++ Style Guide. Target C++20. Rules marked **(Project)** are our own additions or deviations.

## 1. Naming

| Kind | Rule | Examples |
| --- | --- | --- |
| Files | `lowercase_with_underscores`; one class per `.h`/`.cpp` pair (`.cpp` is Project) | `odometry_pod.h` |
| Types (classes, structs, enums, aliases, template params) | `PascalCase`, no underscores | `DrivetrainSubsystem`, `UnitType` |
| Variables, parameters | `snake_case` | `delta_cd`, `heading_deg` |
| Class data members | `snake_case_` (trailing underscore) | `controller_` |
| Struct data members | `snake_case`, no trailing underscore | `x`, `y` |
| Constants (any `const`/`constexpr` value fixed for the whole program, whatever its type) | `kPascalCase` | `kTrackWidth`, `kImu1Port`, `kLeftMotorPorts` |
| Hardware objects (globals in `main.cpp`) | `snake_case`, no `k` prefix and no trailing underscore **(Project)** | `left_motors`, `main_imu` |
| Functions | `PascalCase` | `GetPose()` |
| Accessors/mutators | May be named like variables | `count()`, `set_count()` |
| Namespaces | `snake_case`; top level is the project name | `robot`, `drivetrain_constants` |
| Enumerators | `kPascalCase` | `AutoRoutine::kLeftSide` |
| Macros | Avoid; otherwise `ALL_CAPS` with project prefix | `ROBOT_DEBUG` |
| Vendor names | Anything overriding/mirroring a PROS or LemLib API keeps the vendor's name | `get_rotation()`, `opcontrol()` |

Also:

- **Descriptive names.** Abbreviate only universal robotics terms (`imu`, `rpm`, `pid`, `odom`). `kOdometryPodWheelRadius`, not `kOpWr`.
- **Single letters.** Only for loop indices and standard math variables (`i`, `x`, `y`).
- **No type encoding.** No `iCount` or `pMotor`; the compiler knows the type.
- **Reserved names.** Never start a name with an underscore or use a double underscore anywhere; the standard reserves them.
- **Acronyms (Project).** Treat as words in PascalCase: `AveragedImu`, not `AveragedIMU`.
- **Instances (Project).** Name an instance differently from its type: `AveragedImu averaged_imu`.
- **Booleans (Project).** Name them as yes/no questions: `is_`, `has_`, `should_`. Predicate functions follow the function rule: `IsSettled()`.
- **Units (Project).** Plain numbers carrying a unit get a suffix: `_deg`, `_rad`, `_cd`, `_in`, `_mm`, `_ms`, `_s`, `_mv`, `_rpm`, `_pct`. Rates use `_per_`: `speed_in_per_s`. Typed values (e.g. `Length`) don't need one.
- **Reference frames (Project).** If a value only makes sense in one frame, put the frame in the name: `pose_field`, `vel_robot`.

### Constants in `constants.h` (Project)

- One namespace per subsystem, named `<subsystem>_constants`. Don't repeat the subsystem in the constant: `drivetrain_constants::kTrackWidth`, not `kDrivetrainTrackWidth`.
- Constants are `const` or `constexpr`, never a plain non-const variable, and every one is `kPascalCase`. Prefer `constexpr` (as `inline constexpr`, so every file shares one copy); use plain `const` only for types that can't be `constexpr`, such as `std::vector`.
- Groups of ports are plural (`kLeftMotorPorts`); a single port is the device plus `Port` (`kImu1Port`). A port list can be a `const std::vector<std::int8_t>`; use `constexpr std::array` instead if you want it `constexpr`.
- Gains name the controller and the term: `kTurnKp`, `kLateralKd`.
- Compute derived values from other constants instead of typing the result: `kWheelCircumference = kWheelDiameter * std::numbers::pi`.
- Comment every constant: unit, frame or direction if it matters, and where the number came from (measured, tuned, datasheet).

## 2. Files, Headers and Scoping

**Headers**

- Every `.cpp` has a matching `.h` (except `main.cpp`).
- Headers are self-contained: they compile alone and include what they need.
- Use `#pragma once` instead of Google's `#define` guards **(Project)**.
- Include what you use (e.g. `<cmath>` for `std::cos`); never rely on transitive includes; remove unused ones. Umbrella headers (`api.h`, `main.h`, `lemlib/api.hpp`) are exempt: keep them even when the editor calls them unused **(Project)**.
- Avoid forward declarations; include the header.
- Include order: related header, C system, C++ standard, other libraries (PROS, LemLib, units), project. Blank line between groups, alphabetical within each. clang-format enforces this (section 6).
- `main.h`: don't reorder its includes or `#define`s (`PROS_USE_SIMPLE_NAMES` changes which macros exist). Wrap it in `// clang-format off` / `// clang-format on` **(Project)**.

**Scoping**

- Put project code in a namespace. Never use `using namespace` or add to `std`. Close with `}  // namespace name`.
- File-local code goes in an unnamed namespace in the `.cpp`, never in a header. Close with `}  // namespace`.
- Prefer namespaced free functions over classes with only static members.
- Declare locals in the narrowest scope and initialize at declaration.
- Static/global objects should be trivially destructible and constants `constexpr`. The exceptions are hardware objects in `main.cpp` (see section 5) and constants whose type can't be `constexpr`, such as a port `std::vector`.
- **Configuration (Project).** All tunables (ports, offsets, gains) live in `constants.h` (see section 1). Each port number appears once.

## 3. Classes

- **Constructors:** trivial work only; never call virtual functions. Calibration and delays go in `initialize()`.
- **`explicit`:** mark single-argument constructors and conversion operators `explicit` (not copy/move constructors).
- **Copy/move:** state in the public section whether the class is copyable, move-only, or neither; declare or delete accordingly.
- **Struct vs. class:** `struct` only for passive data with all-public fields and no invariants.
- **Inheritance:** prefer composition. Inherit publicly and only for true is-a relationships, such as a drop-in replacement for a PROS type (`AveragedImu` is a `pros::Imu`).
- **Overrides:** mark with `override` or `final`; don't repeat `virtual`.
- **Operators:** overload only when the meaning is obvious. Never overload `&&`, `||`, `,`, or unary `&`.
- **Access:** data members are private (constants excepted).
- **Declaration order:** `public:`, `protected:`, `private:`. Within each: types, constants, factory functions, constructors, destructor, other functions, data members.
- **Initialization order:** members initialize in declaration order, so keep the initializer list in that order. Declare a member built from other members after them (e.g. `chassis_` after `drivetrain_`).
- **Dependencies (Project):** pass by reference through the constructor; classes don't reach for globals.

## 4. Language Features

- **Enums:** prefer `enum class`.
- **`[[nodiscard]]`:** put it on functions that return `std::optional` or an error value, so callers can't ignore failure **(Project)**.
- **Pi:** use `std::numbers::pi` from `<numbers>`, not `M_PI`.
- **Magic numbers:** no unnamed numeric literals in logic; name them in `constants.h` or a local `constexpr`.
- **Arrays:** prefer `std::array` over C arrays, and `std::span` over pointer plus size if the toolchain supports it.

## 5. PROS and Robot Rules (Project)

| Topic | Rule | Why |
| --- | --- | --- |
| Screen output | Use `pros::screen::print` (`#include "pros/screen.hpp"`); never mix with `pros::lcd`. Pad format strings with trailing spaces; give each value a fixed line. | `print` doesn't clear the rest of the line, and the two APIs draw over each other. |
| Loops | Every `while (true)` has a delay: 20 ms in `opcontrol()`, 50–200 ms for debug displays. Comment the rate. For a steady rate, use `pros::Task::delay_until` instead of `pros::delay`. | No delay starves other tasks; a plain delay drifts by the loop's run time. |
| Blocking calls | Never block in the `opcontrol()` loop. Calibrate sensors in `initialize()`, which should finish in a few seconds. | Blocking freezes driver control. |
| Heap use | No heap allocation (`new`, growing `std::vector`, `std::string` building) inside control loops or `autonomous()`. Allocate once in `initialize()`. | Limited RAM, fragmentation, and unpredictable timing. |
| Tasks | Say in a comment who creates and who stops each `pros::Task`. Guard data shared between tasks with a `pros::Mutex` and a lock guard. | Unsynchronized shared data causes rare, hard-to-reproduce bugs. |
| Debug output | Rate-limit terminal prints (`printf`, `std::cout`) the same way as screen output. | Serial output is slow and can stall a tight loop. |
| Callbacks | `initialize`, `disabled`, `competition_initialize`, `autonomous`, `opcontrol` keep their exact names. | PROS looks them up by name. |
| Hardware objects | Define in `main.cpp`, in an unnamed namespace, in dependency order: sensors, odometry, subsystems. | Globals construct in declaration order. |
| Sensor errors | Check for `PROS_ERR` / `PROS_ERR_F` before using a value; never let error values into the math. Wrappers return the error value on failure and 1 on success. | A disconnected sensor returns infinity, not an exception. |
| Angles | Declaration comment states degrees vs. radians and wrapped heading vs. unbounded rotation. Never subtract two wrapped headings; wrap the difference or use rotation. | A 359° to 1° crossing otherwise looks like a 358° turn. |
| Unit conversion | Convert once, where the raw value is read (e.g. centidegrees to radians). | Keeps units from leaking through the code. |
| Motor ports | A negative port means a reversed motor. Note this once, next to the port lists. | The sign is easy to miss. |

## 6. Formatting and Tooling

Formatting is whatever **clang-format with the Google preset** produces, with the changes below. No need to format by hand. The config files live in this repo's `configs/` folder and are copied into each robot repo's root (see the [README](README.md)).

**Our deviations from the Google preset (Project)**

| Item | Google | Here |
| --- | --- | --- |
| Line length | 80 columns | 100 (PROS, LemLib and units names are long) |
| Include grouping | Regroup by Google's categories | Regroup by ours (section 2) |
| Wrapped calls | Bin-packed arguments | One argument/parameter per line once it doesn't fit |

**Config files**

| File | What it does |
| --- | --- |
| `.clang-format` | Formats code. Run on save. Enforces include order, line length and wrapping. |
| `.clang-tidy` | Checks naming (the table in section 1), braces on multi-line bodies, `nullptr`, `override`, and `using namespace`. Warnings show in the editor through clangd. |
| `.clangd` | Silences the "header not used directly" warning |

Each file starts with a comment naming the guide version it came from.

**Editor setup (VS Code with the PROS extension)**

- Turn on format on save: `"editor.formatOnSave": true`.
- clangd (installed by the PROS extension) reads `.clang-tidy` and `.clangd` automatically. It needs a `compile_commands.json`; the PROS extension normally creates one after a build, or run `pros build-compile-commands`. Don't commit that file, since its paths differ per machine.
- After editing any config file, run **clangd: Restart language server**.
- To silence a deliberate exception, add `// NOLINT(check-name)` on the line or `// NOLINTNEXTLINE(check-name)` above it, using the check name from the warning. Accessors named like variables (`count()`) trigger the function-naming check, so expect a few of these.
- `main.h` is protected by its `// clang-format off` / `// clang-format on` markers. Use those markers sparingly and nowhere else.
- Keep renames and reformatting in separate commits from behavior changes.

## 7. Review Checklist (Project)

Before merging:

- Ran clang-format; no formatting noise mixed into behavior changes.
- No new magic numbers; new tunables are in `constants.h` with unit and source comments.
- Every loop has a delay and nothing blocks in `opcontrol()`.
- Sensor values are checked for `PROS_ERR` / `PROS_ERR_F` before use.
- Angles state their unit and wrapping; no subtraction of two wrapped headings.
- Shared data between tasks is mutex-guarded; no heap allocation in control loops.

## 8. Known Deviations (Project)

A robot repo that has to break a rule says so in a "Known deviations" section of its README: the rule, the reason, and the date. Single lines use `// NOLINT(check-name)` with a short reason. Don't edit the copied config files without recording the change there; if the change is useful for every robot, propose it here instead.

