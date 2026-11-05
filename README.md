# mk_emul — STM32F407 emulator

> Builds on the original version at commit `bf0b419`, extended in pull requests #1 and #2.
> **What changed and why:** [CHANGELOG.md](CHANGELOG.md) (summary table at the top, then every fix
> with the files it touched). Diff of the code: `git diff bf0b419 main`, step by step:
> `git log --reverse bf0b419..main`. Remaining work: [docs/TODO.md](docs/TODO.md).

Desktop emulator of **STM32F407VG** (Cortex-M4): Unicorn Engine for the CPU, a custom MMIO model for peripherals, Qt 6 Quick/QML UI.

## Authors

- **LifeStyle28** — original version (commit `bf0b419`): emulator base with Qt UI, core and
  firmware examples; project supervision and review.
- **Svyatoslav Nikulin** ([Taraved](https://github.com/Taraved)) — development in
  [#1](https://github.com/LifeStyle28/mk_emul/pull/1) and [#2](https://github.com/LifeStyle28/mk_emul/pull/2): Cortex-M4 core with exact interrupt timing,
  fault handling and FPU context; TIM2–TIM7 with PWM; STM32CubeF4 HAL, FreeRTOS and CubeMX
  projects in Build & Load; source-level debugger (GDB server + gdb/MI client); optimization
  levels; Clock Configuration; 127 automated tests. Details in [CHANGELOG.md](CHANGELOG.md).

## Features

- Load **ELF / HEX / BIN** or the built-in LED demo
- **Source-level debugger** (gdb behind the UI, as in STM32CubeIDE): breakpoints on source lines
  (click a line number or F9), Step Over / Into / Out, call stack, local and global variables
  (structs and arrays expand, changed values in red, values can be edited), watch expressions,
  hex view. The emulator has its own GDB server; Build & Load starts `arm-none-eabi-gdb` on it
- **Optimization level** for Build & Load: `-O0`, `-Og`, `-O1`, `-O2`, `-O3`, `-Os`
  (Build → Optimization or the box next to Build & Load); flash/RAM size in the build log
- **Clock Configuration** (Build menu) like the CubeMX clock tab: HSI/HSE, PLL /M ×N /P /Q,
  AHB/APB prescalers, timer clocks, flash wait states, limit checks; generates
  `SystemClock_Config()` (HAL or register level) into the project and shows the clocks the
  running firmware actually set
- One menu (**File / Build / Debug / View**) with a shortcut for every action; recent projects,
  examples, New Project, Go to Line; the last project opens on start
- In-app C editor + **Build & Load** via `arm-none-eabi-gcc`, with the bundled
  **STM32CubeF4 HAL** (project has `stm32f4xx_hal_conf.h`) and **FreeRTOS** (project has
  `FreeRTOSConfig.h`); CubeMX project folders (`Core/Inc`, `Core/Src`) build as they are.
  The build runs in the background, library objects are cached.
  See [device/stm32f407/THIRD_PARTY.md](device/stm32f407/THIRD_PARTY.md)
- Editor can take the whole window: **Expand** in the Source panel or **Ctrl+M**
- Run / Pause / Step, address breakpoints, registers, memory, disassembly
- **Real-time mode** (default): 1 emulated second = 1 wall-clock second at the current HCLK;
  also ×0.1, ×0.5, ×2, ×10 and max speed; measured speed is shown in the toolbar
- Cortex-M4 core: exact interrupt latency, priorities, BASEPRI/PRIMASK, SVC/PendSV,
  HardFault/BusFault/UsageFault with CFSR/HFSR/BFAR, FPU context on exception entry,
  `WFI` sleeps until the next interrupt (status `Running (WFI sleep)`)
- Virtual Discovery board: PD12–PD15 LEDs, PA0 button, ADC slider
- TIM2–TIM7 (TIM6/TIM7 also as the HAL time base) with PWM: LED brightness follows the TIM4 duty
  cycle when PD12–15 are in AF2
- USART console, RCC/SysTick/NVIC/EXTI/TIM/ADC/SPI/I2C stubs (HAL-friendly ready flags),
  bit-band regions
- Accesses to peripherals the emulator does not have are reported in the Build log panel

## Build (Windows)

Tested with Visual Studio 2026 Community (workload "Desktop development with C++", includes CMake and Ninja) and Qt 6.8.3 `msvc2022_64`. VS 2022 Build Tools also work.

1. Qt without an account, via [aqtinstall](https://github.com/miurahr/aqtinstall):
   ```powershell
   py -m pip install --user aqtinstall
   py -m aqt install-qt windows desktop 6.8.3 win64_msvc2022_64 -O C:\Qt
   ```
2. ARM toolchain for the in-app **Build & Load** (do this before the first build: CMake copies
   `third_party/toolchain` next to the exe as `toolchain\`):
   ```powershell
   powershell -ExecutionPolicy Bypass -File scripts/fetch_toolchain.ps1
   ```
3. Build (finds `vcvars64.bat` itself; override with `QT_DIR` / `VCVARS` env vars):
   ```bat
   scripts\build.bat
   ```
   First configure downloads Unicorn and Capstone (needs network).
   Close a running `mk_emul.exe` before rebuilding, otherwise linking fails (LNK1104).

Run `build\mk_emul.exe` (desktop UI).

Console smoke run without UI: `mk_emul.exe --headless [steps] [firmware.elf|.hex|.bin]` prints
`status=... leds=0x? pc=0x...`. Without a firmware path it runs the built-in blinky.

## Quick start

1. **File → Open Example → blinky** (or File → Open Project… for your own folder).
2. **Build & Load** (Ctrl+B) → the Build log ends with `Linked ...firmware.elf`, the Debug panel
   shows `gdb · stopped`.
3. **Run** (F5) → the four LEDs on the board (top left) blink.
4. Click the number of a line in `main()` → red dot; the program stops there (yellow arrow).
   F10 / F11 / Shift+F11 step, the Debug panel shows variables and the call stack.
5. Change the code, **Build & Load** again, **Run**. A compile error is shown in the Build log with
   the line number; the previous firmware keeps running.

### Shortcuts

| Action | Keys | | Action | Keys |
|---|---|---|---|---|
| New / Open Project | Ctrl+N / Ctrl+O | | Run / Continue | F5 |
| Save All | Ctrl+S | | Pause | Shift+F5 |
| Open Firmware | Ctrl+Shift+O | | Reset | Ctrl+Shift+F5 |
| Build & Load | Ctrl+B | | Step Over / Into / Out | F10 / F11 / Shift+F11 |
| Go to Line | Ctrl+G | | Step Instruction | Ctrl+F10 |
| Expand editor | Ctrl+M | | Toggle / delete all breakpoints | F9 / Ctrl+Shift+F9 |

Troubleshooting the debugger: set `MK_EMUL_GDB_LOG=C:\path\gdb.log` before starting
`mk_emul.exe`, the gdb/MI traffic is written there.

## Tests

```bat
scripts\test.bat
```
Builds everything and runs CTest: 127 tests, ~45 s, expected `100% tests passed`
(`FirmwareTest.Speed` is a disabled benchmark).

| What | Run | Checks |
|---|---|---|
| All tests | `scripts\test.bat` | everything below |
| Core unit + firmware tests | `build\tests\mk_emul_tests.exe` | CPU, NVIC, peripherals, examples on the emulator |
| One group | `build\tests\mk_emul_tests.exe --gtest_filter=NvicTest.*` | |
| Build & Load code | `build\tests\mk_emul_builder_tests.exe` | examples, CubeMX layout, HAL time base on TIM6, compile errors, every optimization level, generated clock code |
| Debugger | `build\tests\mk_emul_debug_tests.exe` | real `arm-none-eabi-gdb` against the emulator: line breakpoints, steps, variables, watch, -O2 |
| End-to-end | `build\mk_emul.exe --headless 20000 build\tests\firmware\systick_exti.elf` | prints `status=... leds=0x? pc=0x...` |

Test groups (`--gtest_filter`):

| Group | File | Covers |
|---|---|---|
| `MachineTest.*` | `tests/test_cpu.cpp` | instruction count, breakpoints, step range, faults, lockup, WFI |
| `GdbStubTest.*` | `tests/test_gdbstub.cpp` | GDB remote protocol: registers, memory, breakpoints, continue, steps |
| `ClockConfig.*` | `tests/test_clock.cpp` | clock tree, limits, RCC registers, `SystemClock_Config()` replacement |
| `NvicTest.*` | `tests/test_nvic.cpp` | interrupt latency, priorities, SVC/PendSV, FPU context, CPACR/NOCP, FPDSCR |
| `SysTickFixture.*`, `TimerFixture.*`, `UsartFixture.*`, `Timer.*`, `Adc.*`, `Bitband.*`, `Exti.*` | `tests/test_periph_regs.cpp` | register semantics (partial writes, rc_w0/rc_w1), counters |
| `ExtiFixture.*`, `Gpio.*` | `tests/test_gpio_exti.cpp` | GPIO, button, EXTI |
| `Rcc.*` | `tests/test_rcc.cpp` | clock tree, HSE + PLL |
| `RealTimePacer.*` | `tests/test_pacer.cpp` | real-time mode |
| `Disassembler.*` | `tests/test_disasm.cpp` | disassembly panel data |
| `FirmwareTest.*`, `HalFirmwareTest.*`, `FreeRtosFirmwareTest.*` | `tests/test_firmware.cpp` | `examples/*` built with gcc and run (skipped without the toolchain) |
| `BuilderTest.*`, `HalSources.*` | `tests/test_builder.cpp` | same code as **Build & Load** |
| `DebuggerTest.*`, `DebuggerOptimizedTest.*`, `GdbMi.*` | `tests/test_debugger.cpp` | the UI debugger with gdb |

The UI itself is checked by hand: [docs/MANUAL_TESTING.md](docs/MANUAL_TESTING.md) (~25 min,
sections 15–18 cover the debugger, optimization, clock configuration and the menu).
`scripts\ui.ps1` takes window screenshots and presses buttons to go through it automatically.

## Examples

Open with **File → Open Example** → **Build & Load** → **Run**. Prebuilt ELF files (after
`test.bat`) are in `build\tests\firmware\` for **File → Open Firmware…**.

| Folder | What it shows | Expected |
|---|---|---|
| `examples/blinky` | GPIOD 12–15 toggle | all four LEDs blink |
| `examples/uart_printf` | USART2 text | `STM32F407 emulator UART2`, then `tick` in the USART console |
| `examples/systick_exti` | SysTick delay + EXTI0 | green blinks; **USER PA0** toggles red |
| `examples/hardfault` | unmapped read → `HardFault_Handler` | stops with `HardFault: read unmapped @ 0x60000000`, `SCB_CFSR = 0x00008200` |
| `examples/fpu_irq` | float math in `main()` and in a frequent SysTick interrupt | green blinks; red would mean corrupted FP registers |
| `examples/tim4_pwm` | TIM4 PWM, main loop sleeps in `WFI` | 25/50/100% brightness, green "breathes", status `Running (WFI sleep)` |
| `examples/hal_blinky` | STM32CubeF4 HAL as CubeMX generates it: 168 MHz from HSE + PLL | `HAL ok, SYSCLK=168000000`, `tick 1, 2…`; PA0 toggles red |
| `examples/freertos` | six tasks, a queue, a semaphore from an interrupt, per-task FPU context | `rx 1 a=.. b=..` every 100 ms; PA0 toggles orange |

## Repository layout

| Path | Contents |
|---|---|
| `src/core`, `src/periph` | Qt-free emulator core (`mk_emul_core`): CPU, NVIC, peripherals, GDB stub, clock tree |
| `src/debug` | GDB server (TCP) and the gdb/MI client behind the Debug panel |
| `src/app` | `main.cpp`: UI start and `--headless` mode |
| `src/toolchain` | project builder behind **Build & Load** |
| `src/ui` | Qt Quick UI (QML panels and the C++ bridge) |
| `device/stm32f407` | startup, linker script, CMSIS, STM32CubeF4 HAL, FreeRTOS |
| `examples/` | firmware examples |
| `tests/` | GoogleTest suites, `tests/asm` small Cortex-M programs |
| `scripts/` | build, tests, toolchain download, UI automation |
| `docs/` | manual checklist, remaining work |
