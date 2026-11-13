## Respnnse 1

For a dual-component ESP32 and single-page web app project using GitHub Copilot or Claude, OpenSpec and GitHub Spec Kit are the strongest Spec-Driven Development (SDD) frameworks. Because your firmware (C/C++) and frontend (JavaScript) share a strict communication boundary—such as REST endpoints or WebSockets—an SDD framework keeps both components synchronized around a single source of truth so your AI agent does not hallucinate mismatched API payloads or pin states. [1] (https://ranthebuilder.cloud/blog/i-tested-three-spec-driven-ai-tools-here-s-my-honest-take/), [2] (https://www.thebcms.com/blog/spec-driven-development/), [3] (https://github.com/github/spec-kit)

## Top SDD Frameworks for Your Stack

|Framework	|Best For	|Agent Support	|Key Strength for ESP32 + SPA |
|---|---|---|---|   
|<u>OpenSpec</u>	|Iterative feature changes and API updates	|Claude Code, GitHub Copilot, Cursor	|Uses compact "delta specs" that make it easy to update a shared JSON/WebSocket contract across C++ and JS simultaneously.|
|<u>GitHub Spec Kit</u>	|Greenfield projects with strict guardrails	|Native GitHub Copilot & Claude Code	|Enforces a constitution.md file where you can lock in hardware constraints like RAM limits, LittleFS bundle sizes, and FreeRTOS rules.|
|<u>BMAD Method</u>	|Complex architectures needing deep upfront planning	|Claude Code, GitHub Copilot	|Generates full architecture docs and story breakdowns before coding, though the heavy artifact set takes longer to review.|


## Why OpenSpec or Spec Kit Fit Best

- **OpenSpec for Cross-Component Syncing.** When you add a hardware feature—like a new PWM slider or sensor readout—you modify both firmware and UI. OpenSpec keeps specs compact and reviewable, allowing you to define the exact data contract once and have Claude or Copilot implement both sides accurately. [1] (https://esp32io.com/tutorials/diyables-esp32-web-apps), [2] (https://ranthebuilder.cloud/blog/i-tested-three-spec-driven-ai-tools-here-s-my-honest-take/)

- **GitHub Spec Kit for Embedded Constraints.** Microcontrollers have strict memory and flash limits that general-purpose AI prompts often ignore. Spec Kit’s /speckit-constitution step lets you establish permanent project rules—such as "no dynamic heap allocation in C++ loops" and "build the SPA into a single gzipped asset under 150 KB"—that the agent checks before every task. [1] (https://developer.microsoft.com/blog/spec-driven-development-spec-kit/), [2] (https://github.com/github/spec-kit)

## Recommended Setup for ESP32 + Web UI

Whichever framework you choose, structure your repository and specs to bridge the two languages cleanly:

- **Use a Monorepo Layout.** Keep /firmware (PlatformIO or ESP-IDF) and /web (e.g., Preact, Svelte, or vanilla JS built with Vite) in the same repository so Claude or Copilot can read both folders in one context window. [1] (https://www.reddit.com/r/esp32/comments/eqatjl/introducing_createesp32app_the_fastest_way_to/), [2] (https://github.com/github/spec-kit), [3] (https://developer.espressif.com/blog/how-to-create-an-ui-application-for-the-esp32-s3/)
- **Make the Interface Contract a First-Class Spec.** Dedicate a spec file (such as specs/protocol.md or an AsyncAPI/OpenAPI schema) strictly to the HTTP routes, WebSocket message types, and JSON payloads shared between the ESP32 and the browser. [1] (https://medium.com/@haquemafidul786/github-copilot-spec-driven-development-sdd-rethinking-the-software-development-life-cycle-7707ef137929)
- **Add Build Verification to Your Constitution.** Instruct your agent to run both pio run (or idf.py build) and npm run build as mandatory acceptance gates after generating code so broken C++ types or oversized JS bundles are caught immediately. [1] (https://developer.espressif.com/blog/how-to-create-an-ui-application-for-the-esp32-s3/), [2] (https://testdino.com/blog/spec-driven-testing)
If you'd like, let me know:
- 
- Which **ESP32 framework** you are using (Arduino/PlatformIO or ESP-IDF)
- Whether you prefer **Claude Code** in the terminal or **GitHub Copilot** in VS Code
- If the project is **brand new** or an **existing codebase**

- I can generate a starter `constitution.md` and directory setup tailored to your exact workflow.


## Response 2
Here is a hardware-specific, agent-agnostic constitution.md tailored to the SparkFun Thing Plus - ESP32 WROOM (Micro-B), followed by instructions on how to wire it into your repository so both Claude Code and GitHub Copilot respect it automatically.
Because you are not committed to a single AI tool, the cleanest approach—recommended by developers in the GitHub Spec Kit community—is the adapter pattern. You keep one canonical .specify/memory/constitution.md file as your single source of truth, and use tiny pointer files (CLAUDE.md and .github/copilot-instructions.md) so whichever tool you open reads the exact same rules without sync debt. [1] (https://github.com/github/spec-kit/discussions/2476), [2] (https://www.augmentcode.com/guides/github-spec-kit-practical-guide-sdd-workflow), [3] (https://htek.dev/articles/github-spec-kit-english-to-production-specs)

## 1. Copy-Pasteable `constitution.md`

Save this file at ``.specify/memory/constitution.md` (the standard path used by GitHub Spec Kit so you can adopt its `/speckit` workflows anytime without moving files): [1] (https://github.com/github/spec-kit), [2] (https://www.augmentcode.com/guides/github-spec-kit-practical-guide-sdd-workflow)
``` markdown
# Project Constitution: SparkFun Thing Plus - ESP32 WROOM (Micro-B)

**Version:** 1.0.0 | **Target Hardware:** SparkFun Thing Plus ESP32 WROOM (Micro-B / WRL-15663)  
**MCU:** Espressif ESP32-D0WDQ6 (Xtensa Dual-Core 32-bit LX6 @ up to 240 MHz, 520 KB SRAM, 16 MB Flash)

---

## 1. Core Engineering Principles

1. **Deterministic & Non-Blocking Execution:** Never block the main loop or FreeRTOS tasks with `delay()` in production paths. Use non-blocking state machines, FreeRTOS timers/queues, or event groups.
2. **Hardware Safety First:** All GPIO logic is strictly **3.3V** and **NOT 5V-tolerant**. Never generate code or wiring instructions that apply 5V directly to any GPIO, ADC, or communication pin.
3. **Memory Discipline:** Dynamic heap allocation (`malloc`, `new`, Arduino `String` concatenation in loops) is prohibited in long-running tasks to prevent heap fragmentation within the 520 KB SRAM ceiling. Prefer static allocation, fixed-size `char[]` buffers, and `std::array` / `etl` containers.
4. **Dual-Core Awareness (FreeRTOS):** Explicitly pin time-critical hardware I/O and interrupts to Core 1 (`APP_CPU`), leaving Core 0 (`PRO_CPU`) for the Wi-Fi/BLE protocol stack and background system tasks.
5. **Fail-Safe & Watchdog Compliance:** Every task loop must yield (`vTaskDelay` or event wait) so the Task Watchdog Timer (TWDT) never starves. All peripheral reads (I2C, SPI, UART, ADC) must handle timeouts and bus errors gracefully without panicking.

---

## 2. Target Hardware Pinout & Electrical Constraints

AI agents MUST strictly obey the SparkFun Thing Plus ESP32 WROOM (Micro-B) hardware map:

### Fixed Onboard Pins
- **Onboard Status LED (Blue):** `GPIO 13` (Active High)
- **Default I2C / Qwiic Bus:** `SDA = GPIO 21`, `SCL = GPIO 22` (3.3V logic; Qwiic connector shares this bus)
- **Default SPI Bus (VSPI):** `SCK = GPIO 18`, `MISO = GPIO 19`, `MOSI = GPIO 23`, `CS = GPIO 5`
- **Primary UART (USB-Serial Debug/Flash):** `TX0 = GPIO 1`, `RX0 = GPIO 3` (Do not reassign unless USB serial logging is explicitly disabled)
- **Secondary Hardware UART:** `TX1 = GPIO 17`, `RX1 = GPIO 16`

### Restricted & Special-Purpose Pins (Enforce Strictly)
- **Input-Only Pins:** `GPIO 34`, `GPIO 35`, `GPIO 36`, `GPIO 39` — Do NOT configure internal pull-ups/pull-downs (`INPUT_PULLUP` / `INPUT_PULLDOWN` do not exist in silicon on these pins) and never set them as outputs.
- **Strapping Pins:** `GPIO 0`, `GPIO 2`, `GPIO 5`, `GPIO 12`, `GPIO 15` — Do not attach external hardware that pulls `GPIO 0` or `GPIO 2` low or `GPIO 12` high during boot, as this will break flashing or flash-voltage selection.
- **ADC2 Conflict Rule:** `ADC2` channels (`GPIO 0, 2, 4, 12, 13, 14, 15, 25, 26, 27`) CANNOT be used while Wi-Fi is active. Any analog sensor reading in a Wi-Fi-enabled build MUST be assigned to **`ADC1` (`GPIO 32, 33, 34, 35, 36, 39`)**.
- **Flash SPI Pins:** `GPIO 6` through `GPIO 11` are connected to the internal 16 MB SPI flash. Never reference or expose them in application code.

---

## 3. Firmware Architecture & Code Standards

- **Hardware Abstraction:** Isolate all pin numbers, I2C addresses, and peripheral handles in a dedicated `board_config.h` (or `pins.h`) header. Business logic must never hardcode magic pin numbers.
- **Interrupt Service Routines (ISRs):** Keep ISRs minimal. Only set `volatile` flags or push to FreeRTOS `xQueueSendFromISR` / `vTaskNotifyGiveFromISR`. Never call `Serial.print`, I2C/SPI transfers, or heap allocators inside an ISR.
- **Wireless & Power Hygiene:** When Wi-Fi or BLE is used, isolate network reconnection logic into a dedicated non-blocking state machine with exponential backoff. If deep sleep (`2.5 µA` target) is required, explicitly power down external peripherals and configure RTC wake sources before calling `esp_deep_sleep_start()`.
- **Secrets & Credentials:** Wi-Fi SSIDs, passwords, API tokens, and TLS certificates must never be committed to source files. Load them from NVS (`Preferences` / `nvs_flash`) or an untracked `secrets.h` template.

---

## 4. Build, Testing & Verification Gates

- **Clean Compilation:** Every change must compile cleanly with zero warnings (`-Wall -Wextra`) using the project's build system (PlatformIO or ESP-IDF).
- **Host-Side Unit Testing:** Pure logic (parsers, state machines, ring buffers, math/calibration) must be decoupled from hardware headers so it can run on the host via native unit tests without requiring physical hardware attached.
- **Hardware-in-the-Loop (HIL) Checklist:** Any PR or feature modifying pin assignments, bus speeds, or sleep modes must include a verification note confirming serial boot log cleanliness (no Brownout detector triggers or Guru Meditation panics).

---

## 5. Governance & Agent Workflow

- **Constitution Supremacy:** This document supersedes default framework examples and ad-hoc prompts. If a requested feature conflicts with Section 2 (Hardware Constraints), the agent must halt and flag the pin/electrical conflict before writing code.
- **Spec-Before-Code:** For any non-trivial feature, draft or update a concise feature specification (`spec.md`) and implementation plan (`plan.md`) referencing this constitution before modifying firmware source files.
- **Amendments:** Changes to pin mappings, RTOS task budgets, or memory rules require updating this file and incrementing the version header.
  Use code with caution.
```

## 2. Setting Up Your Repository (Claude + Copilot Ready)

Since you are open to clearing your repository and want seamless compatibility with both Claude Code (which your workplace uses) and GitHub Copilot, set up this minimal structure:
1. **Initialize the Spec Kit scaffold (optional, recommended)**:
   You can run GitHub's Specify CLI in your cleared repo to create the standard ``.specify/`` directory and slash commands:
 ```bash
   uvx --from git+https://github.com specify init --here --integration claude
   ```
   (You can also pass `--integration copilot` or *initialize both integrations in the same repo.*) [1] (https://www.augmentcode.com/guides/github-spec-kit-practical-guide-sdd-workflow), [2] (https://github.com/github/spec-kit), [3] (https://speckit.org/)
2. **Add the two thin adapter files**:
   Instead of duplicating rules, create these two short files at the root of your repo so both tools automatically load your constitution on startup:
   1. `CLAUDE.md` (for Claude Code):
    ``` markdown
    # Project Instructions
    Read and strictly enforce `.specify/memory/constitution.md` on all hardware, pinout, memory, and architecture decisions before planning or writing firmware.
    ```
   2. `.github/copilot-instructions.md` (for GitHub Copilot):
    ``` markdown
    Read and strictly enforce `.specify/memory/constitution.md` on all hardware, pinout, memory, and architecture decisions before planning or writing firmware.
    ```

3. **Hardware Pinout Sanity Check**

Note that your **SparkFun Thing Plus - ESP32 WROOM (Micro-B)** (SKU WRL-15663) differs slightly from the newer USB-C revision (WRL-20168): [1] (https://www.sparkfun.com/sparkfun-thing-plus-esp32-wroom-micro-b.html), [2] (https://www.sparkfun.com/sparkfun-thing-plus-esp32-wroom-usb-c.html)
- Your **Micro-B** model has the blue status LED on `GPIO 13` and does not have the onboard WS2812 RGB LED or microSD slot found on the USB-C revision. The constitution above is locked specifically to your Micro-B board's electrical and pinout profile. [1] (https://www.sparkfun.com/sparkfun-thing-plus-esp32-wroom-micro-b.html), [2] (https://cdn.sparkfun.com/assets/learn_tutorials/8/5/2/ESP32ThingPlus_GraphicalDatasheet.pdf), [3] (https://docs.sparkfun.com/SparkFun_Thing_Plus_ESP32_WROOM_C/hardware_overview/), [4] (https://learn.sparkfun.com/tutorials/esp32-thing-plus-hookup-guide/hardware-overview)

To tailor Section 3 and 4 of the constitution even further, could you let me know:

- Which **firmware framework** you plan to use (**PlatformIO + Arduino**, **PlatformIO + ESP-IDF**, or **native ESP-IDF)**?
- What **peripherals or sensors** you are connecting (e.g., Qwiic I2C sensors, SPI displays, LiPo battery monitoring)?

I can update the constitution.md with framework-specific build commands, static analysis flags, and reserved pin allocations for your exact hardware setup.
