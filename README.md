![image](https://github.com/mytechnotalent/ERP2-DAY002/blob/main/DAY002.png?raw=true)

## FREE Reverse Engineering Self-Study Course [HERE](https://github.com/mytechnotalent/Reverse-Engineering-Tutorial)
VIDEO PROMO [HERE](https://www.youtube.com/watch?v=aD7X9sXirF8)

<br>

# ⭐ DAY002: Blink LEDs in Sequence 

<br>

**Difficulty**: Beginner  
**Date**: Day 2 of 365  
**Components**: 4 LEDs, 4 Resistors  
**Concepts**: GPIO, Digital Output, Sequential Control, Async Programming

<br>

# 🔋 Schematic
![image](https://github.com/mytechnotalent/Embedded-Hacking/blob/main/EHP2_bb.png?raw=true)

<br>

# 📋 Project Overview
This is the second project in the **365 Pico2 RP2350 Project Ideas** series. Building on Day 1's single LED blink, we now control multiple LEDs in sequence. This project introduces you to managing multiple GPIO outputs and creating sequential patterns with Embassy Rust.

## What You'll Learn
- Controlling multiple GPIO pins as digital outputs
- Creating sequential LED patterns
- Managing state across multiple components
- Building reusable controller abstractions
- Understanding circular/wrapping sequences
- Using Embassy's async/await for timing multiple outputs

<br>

# 🧩 Hardware
## Raspberry Pi Pico 2 w/ Header [BUY](https://www.pishop.us/product/raspberry-pi-pico-2-with-header)
## USB A-Male to USB Micro-B Cable [BUY](https://www.pishop.us/product/usb-a-male-to-usb-micro-b-cable-6-inches)
## Raspberry Pi Pico Debug Probe [BUY](https://www.pishop.us/product/raspberry-pi-debug-probe)
## Complete Component Kit for Raspberry Pi [BUY](https://www.pishop.us/product/complete-component-kit-for-raspberry-pi)
## 10pc 25v 1000uF Capacitor [BUY](https://www.amazon.com/Cionyce-Capacitor-Electrolytic-CapacitorsMicrowave/dp/B0B63CCQ2N?th=1)
### 10% PiShop DISCOUNT CODE - KVPE_HS320548_10PC

<br>

# 🔌 Hardware Requirements

## Components Needed
| Quantity | Component                    | Notes                                 |
| -------- | ---------------------------- | ------------------------------------- |
| 1        | Raspberry Pi Pico 2 (RP2350) |                                       |
| 4        | LEDs (any color)             | Red, Yellow, Green, or White from kit |
| 4        | 100Ω Resistors               | Current-limiting resistors            |
| 1        | Breadboard                   | For prototyping                       |
| 10       | Jumper Wires                 | Male-to-Male                          |

## Wiring Diagram
```
┌─────────────────────────┐
│  Raspberry Pi Pico 2    │
│                         │
│  ┌────────┐             │
│  │  GP16  ├─────┐       │  LED 0
│  └────────┘     │       │
│  ┌────────┐     │       │
│  │  GP17  ├─────┼───┐   │  LED 1
│  └────────┘     │   │   │
│  ┌────────┐     │   │   │
│  │  GP18  ├─────┼───┼───┐  LED 2
│  └────────┘     │   │   │
│  ┌────────┐     │   │   │
│  │  GP19  ├─────┼───┼───┼───┐  LED 3
│  └────────┘     │   │   │   │
│                 │   │   │   │
│  ┌────────┐     │   │   │   │
│  │  GND   ├──┐  │   │   │   │
│  └────────┘  │  │   │   │   │
└──────────────┼──┼───┼───┼───┘
               │  │   │   │   │
               │  └[100Ω]─[LED0]─┐
               │      │          │
               │  └[100Ω]─[LED1]─┤
               │      │          │
               │  └[100Ω]─[LED2]─┤
               │      │          │
               │  └[100Ω]─[LED3]─┤
               │                 │
               └─────────────────┘
                     GND

Connection Steps:
1. GP16 (Pin 21) → 100Ω Resistor → LED0 Anode (+)
2. GP17 (Pin 22) → 100Ω Resistor → LED1 Anode (+)
3. GP18 (Pin 24) → 100Ω Resistor → LED2 Anode (+)
4. GP19 (Pin 25) → 100Ω Resistor → LED3 Anode (+)
5. All LED Cathodes (-) → GND
```

## Pin Information
- **GP16**: LED 0 (first in sequence)
- **GP17**: LED 1 (second in sequence)
- **GP18**: LED 2 (third in sequence)
- **GP19**: LED 3 (fourth in sequence)
- **GND**: Ground connection (any GND pin works)

<br>

# 🛠️ Software Requirements

## Prerequisites
1. **Rust Toolchain**
   ```bash
   curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
   ```
2. **ARM Cortex-M Target**
   ```bash
   rustup target add thumbv8m.main-none-eabihf
   ```
3. **probe-rs** (for flashing and debugging)
   ```bash
   cargo install probe-rs-tools --locked
   ```
4. **flip-link** (stack overflow protection)
   ```bash
   cargo install flip-link
   ```

## Dependencies
All dependencies are specified in `Cargo.toml`:
- **embassy-executor**: Async task executor for embedded systems (git version)
- **embassy-time**: Time and timer abstractions (git version)
- **embassy-rp**: Hardware Abstraction Layer (HAL) for RP2350 with `rp235xa` chip feature (git version for full RP2350 support)
- **cortex-m**: Low-level Cortex-M utilities
- **panic-halt**: Panic handler for embedded systems
> **Important Note**: We're using git versions of the Embassy framework because the crates.io releases don't yet have full RP2350 support. The RP2350 uses ARMv8-M architecture with different MPU registers than earlier chips. We specifically enable the `rp235xa` feature for Pico 2 (RP2350-A revision) and `critical-section-impl` for proper interrupt handling.

<br>

# 📝 Code Structure

## Project Files
```
DAY002/
├── Cargo.toml           # Project dependencies and configuration
├── build.rs             # Build script for linker configuration
├── memory.x             # Memory layout for RP2350
├── Makefile             # Build and test automation
├── .cargo/
│   └── config.toml      # Target and runner configuration
├── src/
│   ├── main.rs          # Main application code
│   ├── lib.rs           # Library module exports
│   ├── config.rs        # Configuration constants
│   └── led.rs           # LED sequence controller
└── README.md            # This file
```

## Key Code Section

### 1. Main Function (`main.rs`)
```rust
#[embassy_executor::main]
async fn main(_spawner: Spawner) {
    let p = embassy_rp::init(Default::default());
    let mut led0 = Output::new(p.PIN_16, Level::Low);
    let mut led1 = Output::new(p.PIN_17, Level::Low);
    let mut led2 = Output::new(p.PIN_18, Level::Low);
    let mut led3 = Output::new(p.PIN_19, Level::Low);
    let mut controller = LedSequenceController::new();
    loop {
        if led_state_to_level(controller.led_state(0)) {
            led0.set_high();
        } else {
            led0.set_low();
        }
        if led_state_to_level(controller.led_state(1)) {
            led1.set_high();
        } else {
            led1.set_low();
        }
        if led_state_to_level(controller.led_state(2)) {
            led2.set_high();
        } else {
            led2.set_low();
        }
        if led_state_to_level(controller.led_state(3)) {
            led3.set_high();
        } else {
            led3.set_low();
        }
        Timer::after_millis(controller.delay_ms()).await;
        controller.advance();
    }
}
```

<br>

# 🚀 Building and Flashing

## Step 1: Connect the Pico 2
1. Hold the **BOOTSEL** button on the Pico 2
2. Connect the Pico to your computer via USB
3. Release the BOOTSEL button
4. The Pico appears as a USB drive (RPI-RP2)

## Step 2: Build the Project
```bash
cd DAY002
cargo build --release
```
This compiles the code for the RP2350 target.

## Step 3: Flash and Run
```bash
cargo run --release
```
This will:
1. Compile the code
2. Flash it to the Pico 2 using probe-rs
3. Start the LED sequence

## Expected Output
Each LED stays on for 250ms before the next one lights up. The pattern repeats continuously, creating a "chasing" or "running" light effect.

<br>

# 🔧 Troubleshooting

## Issue: `probe-rs` not found
**Solution**: Install probe-rs tools
```bash
cargo install probe-rs-tools --locked
```

## Issue: Can't find device
**Solution**: 
1. Ensure BOOTSEL was pressed during connection
2. Try a different USB cable (some are power-only)
3. Check USB permissions on Linux:
   ```bash
   sudo usermod -a -G plugdev $USER
   ```

## Issue: Build errors about missing target
**Solution**: Add the ARM target
```bash
rustup target add thumbv8m.main-none-eabihf
```

## Issue: LEDs don't light up
**Solutions**:
1. Check the wiring (resistors and polarity)
2. Verify you're using the correct GPIO pins (16, 17, 18, 19)
3. Check if the LEDs are functional (test with a battery)
4. Ensure proper ground connection

## Issue: Only one LED works
**Solutions**:
1. Check individual LED wiring
2. Verify each GPIO pin connection
3. Test each LED circuit independently

## Issue: Linker errors
**Solution**: Install flip-link
```bash
cargo install flip-link
```

<br>

# 📚 Understanding the Code

## Sequential Control Pattern
The sequence controller uses modular arithmetic for wrapping:
```rust
self.current_index = (self.current_index + 1) % self.led_count;
```
This ensures the index always stays within bounds (0 to LED_COUNT-1) and automatically wraps from the last LED back to the first.

## State Management
Each LED's state is determined by comparing its index to the current sequence position:
```rust
pub fn led_state(&self, index: usize) -> LedState {
    if index == self.current_index {
        LedState::On
    } else {
        LedState::Off
    }
}
```
This ensures exactly one LED is on at any time, creating a clean sequential pattern.

## GPIO Control
Each GPIO pin is configured as an output:
```rust
let mut led0 = Output::new(p.PIN_16, Level::Low);
```
- `Output::new()`: Configures the pin as a digital output
- `Level::Low`: Initial state is OFF (0V)

## Timing
Embassy's async timer handles the delay between LED transitions:
```rust
Timer::after_millis(controller.delay_ms()).await;
```

This non-blocking delay allows the CPU to potentially handle other tasks while waiting.

<br>

# 🎯 Experiments and Modifications

## 1. Change Sequence Speed
Modify the delay value in `config.rs`:
```rust
// Fast sequence (100ms per LED)
pub const SEQUENCE_DELAY_MS: u64 = 100;

// Slow sequence (500ms per LED)
pub const SEQUENCE_DELAY_MS: u64 = 500;
```
## 2. Reverse Direction
Modify the `advance()` function in `led.rs`:
```rust
pub fn advance(&mut self) -> usize {
    if self.current_index == 0 {
        self.current_index = self.led_count - 1;
    } else {
        self.current_index -= 1;
    }
    self.current_index
}
```
## 3. Ping-Pong Effect
Create a back-and-forth pattern (preview of Day 24):
```rust
// Add direction field to controller
direction: bool,  // true = forward, false = backward

pub fn advance(&mut self) -> usize {
    if self.direction {
        if self.current_index == self.led_count - 1 {
            self.direction = false;
            self.current_index -= 1;
        } else {
            self.current_index += 1;
        }
    } else {
        if self.current_index == 0 {
            self.direction = true;
            self.current_index += 1;
        } else {
            self.current_index -= 1;
        }
    }
    self.current_index
}
```
## 4. All LEDs On with One Off
Invert the pattern so all LEDs are on except the current one:
```rust
pub fn led_state(&self, index: usize) -> LedState {
    if index == self.current_index {
        LedState::Off  // Current LED is OFF
    } else {
        LedState::On   // All others are ON
    }
}
```

<br>

# 🧪 Challenges
1. **Variable Speed**: Make the sequence speed up over time
2. **Random Sequence**: Light LEDs in random order (preview of Day 8)
3. **Multiple Lit**: Have 2 LEDs lit at once, chasing each other
4. **Knight Rider**: Create the famous back-and-forth pattern (preview of Day 6)

<br>

# 📖 Next Steps
Once you've mastered blinking multiple LEDs in sequence, you're ready for:
- **DAY003**: Traffic light simulation (Red, Yellow, Green)
- **DAY004**: LED brightness control with PWM
- **DAY006**: LED chaser/Knight Rider effect

<br>

# 🔗 Resources

## Official Documentation
- [Embassy Book](https://embassy.dev/book/)
- [embassy-rp Documentation](https://docs.embassy.dev/embassy-rp/)
- [RP2350 Datasheet](https://datasheets.raspberrypi.com/rp2350/rp2350-datasheet.pdf)
- [Pico 2 Documentation](https://www.raspberrypi.com/documentation/microcontrollers/pico-series.html)

## Rust Embedded
- [Embedded Rust Book](https://rust-embedded.github.io/book/)
- [probe-rs Documentation](https://probe.rs/)

## Community
- [Embassy GitHub](https://github.com/embassy-rs/embassy)
- [Rust Embedded Matrix Chat](https://matrix.to/#/#rust-embedded:matrix.org)

<br>

# ✅ Completion Checklist
- [ ] Hardware assembled correctly (4 LEDs with resistors)
- [ ] Rust toolchain installed
- [ ] probe-rs and flip-link installed
- [ ] Project builds without errors
- [ ] LEDs blink in sequence (250ms per LED)
- [ ] Experimented with different sequence speeds
- [ ] Tried reversing the direction
- [ ] Understood the circular sequence pattern
- [ ] Ready to move to DAY003

<br>

**Congratulations!** 🎉 You've completed the second Embassy Rust project on the Pico 2. You now understand how to control multiple GPIO outputs and create sequential patterns. These skills will be essential for many future projects including the Knight Rider effect, traffic lights, and LED games.

*"A journey of 365 projects begins with a single blink... and continues with a sequence."* 🦀

<br>

**Project Status**: ⭐ Beginner | **Estimated Time**: 45-75 minutes | **Success Rate**: 98%

<br>

# 📄 License
MIT License - Copyright (c) 2025
