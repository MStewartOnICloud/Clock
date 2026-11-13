# Project Constitution: Embedded Firmware Architecture

**Version:** 1.1.0 | **Baseline Hardware Profile:** SparkFun Thing Plus - ESP32 WROOM (Micro-B / WRL-15663)  
**Target Family:** Espressif ESP32 Architecture (Upgradeable via Board Profiles)

---

## 1. Core Engineering Principles (Hardware & Framework Invariant)

1. **Deterministic & Non-Blocking Execution:** Never block the main loop or RTOS tasks with busy-waits or `delay()` in production paths. Use non-blocking state machines, RTOS timers/queues, or event groups.
2. **Hardware Safety First:** All GPIO logic is strictly **3.3V** and **NOT 5V-tolerant** unless explicitly isolated by external level-shifting hardware. Never generate code or wiring instructions that apply 5V directly to MCU pins.
3. **Memory Discipline:** Dynamic heap allocation (`malloc`, `new`, unbounded `String` concatenation in loops) is prohibited in long-running tasks to prevent heap fragmentation. Prefer static allocation, fixed-size `char[]` buffers, and bounded containers.
4. **RTOS & Multi-Core Hygiene:** Isolate time-critical hardware I/O and interrupts from network/wireless protocol stacks. Every task loop must yield (`vTaskDelay` or event wait) so the Task Watchdog Timer (TWDT) never starves.
5. **Defensive Peripheral I/O:** All bus reads and writes (I2C, SPI, UART, ADC) must implement explicit timeouts and error handling without panicking or hanging the system.

---

## 2. Hardware Abstraction & Upgradeability

To allow seamless migration to higher-specification hardware (e.g., ESP32-S3 or PSRAM-equipped boards) without rewriting application logic:

- **Strict HAL Separation:** Application and business logic MUST NOT include framework-specific headers (`Arduino.h`, `driver/gpio.h`) or hardcode GPIO numbers directly. All hardware interactions must pass through a thin project HAL interface (`hal_*.h`).
- **Active Board Profile (`board_config.h`):** Pin mappings, bus assignments, and resource budgets live exclusively in a board-specific header selected by the active build configuration.
- **Resource Headroom Trigger:** If compiled firmware exceeds **75% of available Flash** or runtime static/stack allocation exceeds **70% of internal SRAM** on the baseline board, the agent must flag a resource constraint warning so a hardware upgrade can be evaluated in the SDD cycle.

### Active Baseline Profile: SparkFun Thing Plus ESP32 WROOM (Micro-B)
Until amended by a hardware-upgrade `plan.md`, enforce these physical limits:
- **Compute & Memory:** ESP32-D0WDQ6 (Dual-Core 240 MHz, 520 KB SRAM, 16 MB Flash, No PSRAM).
- **Fixed Pins:** Onboard Blue LED = `GPIO 13`; Default I2C/Qwiic = `SDA 21 / SCL 22`; Default VSPI = `SCK 18 / MISO 19 / MOSI 23 / CS 5`; USB UART0 = `TX 1 / RX 3`; Secondary UART1 = `TX 17 / RX 16`.
- **Silicon Restrictions:**
  - `GPIO 34, 35, 36, 39` are **input-only** with **no internal pull-ups/pull-downs**.
  - `GPIO 0, 2, 5, 12, 15` are **strapping pins**; do not attach external circuitry that alters their boot state.
  - `GPIO 6–11` are wired to internal SPI flash; **never reference them**.
  - `ADC2` (`GPIO 0, 2, 4, 12–15, 25–27`) **cannot be used while Wi-Fi is active**; assign analog sensors to `ADC1` (`GPIO 32–36, 39`).

---

## 3. Framework Selection & Build Verification Gates

- **SDD-Governed Tooling:** The choice of build system and SDK (e.g., PlatformIO + Arduino, PlatformIO + ESP-IDF, or native ESP-IDF) is decided during the Specification/Planning phase (`plan.md`) based on feature requirements.
- **Mandatory Build Gate:** Once a build framework is defined in the active `plan.md`, the agent MUST execute that plan's specified build command (e.g., `pio run`, `idf.py build`, or `arduino-cli compile`) after every code modification.
- **Zero-Warning Policy:** Builds must pass with exit code `0` and zero compiler warnings (`-Wall -Wextra` enabled) before any task or pull request is marked complete.
- **Host-Side Unit Testing:** Core business logic, parsers, and state machines must remain pure C/C++ with zero hardware dependencies so they can be compiled and verified on the host machine.

---

## 4. Security & Secrets Hygiene

- **No Hardcoded Secrets:** Wi-Fi credentials, API keys, and TLS certificates must never be committed to version control. Load them from non-volatile storage (NVS) or an untracked local template (`secrets.h.example`).

---

## 5. Governance & SDD Workflow

- **Constitution Supremacy:** This document overrides default SDK examples and ad-hoc prompts. If a feature request violates Section 1 or the active Board Profile in Section 2, the agent must halt and report the conflict before writing code.
- **Spec-Before-Code:** Framework selection, major peripheral additions, and hardware upgrades must be documented and approved in `spec.md` and `plan.md` before modifying firmware source files.
- **Amendments:** Upgrading the target hardware board requires updating the Active Baseline Profile in Section 2 and incrementing the constitution version.
