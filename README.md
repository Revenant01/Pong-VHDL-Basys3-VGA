# Pong — FPGA Implementation in VHDL

A fully hardware-implemented, two-player Pong game synthesized on the **Digilent Basys3 (Artix-7)** FPGA board. The game renders in real-time over a **VGA 640×480 @ 60 Hz** display using a pixel-driven state machine written entirely in **VHDL**.

---

## Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Hardware Requirements](#hardware-requirements)
- [Project Structure](#project-structure)
- [Architecture](#architecture)
- [Pin Mapping](#pin-mapping)
- [Getting Started](#getting-started)
- [Gameplay](#gameplay)
- [Parameters & Constants](#parameters--constants)
- [Authors](#authors)
- [License](#license)

---

## Overview

This project implements the classic arcade game **Pong** at the register-transfer level (RTL) using VHDL. No soft processors, operating systems, or external libraries are involved — every pixel rendered on screen is driven directly by hardware logic synthesized onto the FPGA fabric.

The design targets Xilinx Vivado and the Basys3 development board, leveraging its onboard 100 MHz oscillator and VGA port.

---

## Features

| Feature | Detail |
|---|---|
| Display | VGA 640×480 @ 60 Hz (12-bit color — 4-bit per channel) |
| Clock | 25 MHz pixel clock derived from 100 MHz board clock |
| Players | 2-player simultaneous local multiplayer |
| Controls | 4 dedicated push buttons (up/down per player) |
| Score Tracking | Independent per-player score counters |
| Game Reset | Dedicated hardware reset button |
| Ball Physics | Constant-velocity bounce with paddle deflection |
| Court Lines | Centre divider and boundary lines rendered in hardware |

---

## Hardware Requirements

- **Board:** Digilent Basys3 (Xilinx Artix-7 XC7A35T)
- **Display:** Any VGA monitor (640×480 minimum)
- **Cable:** Standard 15-pin VGA cable
- **Tools:** Xilinx Vivado 2020.1 or later

---

## Project Structure

```
PingPong_Game/
├── constraint.txt                          # Basys3 XDC pin constraints
├── README.md                               # This file
└── PingPongGame/
    ├── PingPongGame.xpr                    # Vivado project file
    ├── PingPongGame.hw/
    │   └── hw_1/
    │       └── hw.xml                      # Hardware definition
    ├── PingPongGame.srcs/
    │   └── sources_1/
    │       └── new/
    │           └── Game.vhd                # Top-level VHDL source (entire design)
    └── PingPongGame.runs/
        └── .jobs/                          # Vivado implementation run artefacts
```

---

## Architecture

The entire design is a single VHDL entity (`Game`) with one behavioral architecture. It contains the following concurrent processes:

```
┌─────────────────────────────────────────────────────┐
│                    Game (Top-Level)                  │
│                                                      │
│  ┌──────────────────┐   ┌───────────────────────┐   │
│  │ clock_generation │   │     main_process       │   │
│  │                  │   │                        │   │
│  │  100 MHz → 25 MHz│   │  VGA Timing Generator  │   │
│  │  (÷4 divider)    │   │  Paddle Renderer       │   │
│  └────────┬─────────┘   │  Ball Renderer         │   │
│           │ clk25        │  Collision Detection   │   │
│           └─────────────►  Score Tracking         │   │
│                         │  Input Handling         │   │
│                         └───────────┬─────────────┘   │
│                                     │                  │
│                            R[3:0], G[3:0], B[3:0]     │
│                            Hsync, Vsync                │
└─────────────────────────────────────────────────────┘
```

### VGA Timing

The design follows the industry-standard 640×480 @ 60 Hz VGA timing:

| Parameter | Value |
|---|---|
| Pixel clock | 25.175 MHz (approximated as 25 MHz) |
| Horizontal total | 800 pixels |
| Vertical total | 525 lines |
| Active area | 640 × 480 |
| H-sync pulse | Pixels 656–751 (negative polarity) |
| V-sync pulse | Lines 490–491 (negative polarity) |

### Paddle & Ball Rendering

Rendering is achieved by comparing the current beam position `(horizontal, vertical)` against the bounding boxes of each game object every pixel clock cycle:

- **Paddle 1** (left) — rendered in **Cyan**
- **Paddle 2** (right) — rendered in **Magenta**
- **Ball** — rendered in **White**
- **Court elements** (boundaries, centre line) — rendered in **White**

---

## Pin Mapping

| Signal | FPGA Pin | Description |
|---|---|---|
| `clk` | W5 | 100 MHz system clock |
| `reset` | U18 | Centre button — game reset |
| `Player1_up` | W19 | Player 1 move paddle up |
| `Player1_down` | U17 | Player 1 move paddle down |
| `Player2_up` | T18 | Player 2 move paddle up |
| `Player2_down` | T17 | Player 2 move paddle down |
| `R[3:0]` | G19, H19, J19, N19 | VGA Red channel (4-bit) |
| `G[3:0]` | J17, H17, G17, D17 | VGA Green channel (4-bit) |
| `B[3:0]` | N18, L18, K18, J18 | VGA Blue channel (4-bit) |
| `Hsync` | P19 | VGA Horizontal sync |
| `Vsync` | R19 | VGA Vertical sync |

---

## Getting Started

### 1. Clone the repository

```bash
git clone https://github.com/Revenant01/FPGA-Pong.git
cd FPGA-Pong
```

### 2. Open in Vivado

1. Launch **Xilinx Vivado**.
2. Select **Open Project** and open `PingPongGame/PingPongGame.xpr`.
3. The project will load with `Game.vhd` as the top-level source and `constraint.txt` as the XDC constraints file.

### 3. Synthesise & Implement

```
Flow Navigator → Run Synthesis → Run Implementation → Generate Bitstream
```

### 4. Program the board

1. Connect the Basys3 to your PC via USB.
2. Connect a VGA monitor to the Basys3 VGA port.
3. In Vivado: **Open Hardware Manager → Open Target → Auto Connect → Program Device**.
4. Select the generated `.bit` file and click **Program**.

---

## Gameplay

| Action | Player 1 | Player 2 |
|---|---|---|
| Move paddle up | Left button (W19) | Up button (T18) |
| Move paddle down | Right button (U17) | Down button (T17) |
| Reset game | Centre button (U18) | — |

- The ball spawns at the centre of the screen and moves at a constant speed.
- A point is scored when the ball passes a paddle and reaches the boundary.
- Press **Reset** at any time to restart the game.

---

## Parameters & Constants

All game parameters are defined as VHDL constants at the top of `Game.vhd` for easy tuning:

| Constant | Value | Description |
|---|---|---|
| `paddle_length` | 80 px | Height of each paddle |
| `paddle_width` | 5 px | Width of each paddle |
| `Paddle_speed` | 3 px/frame | Paddle movement speed |
| `ball_Par` | 10 px | Ball side length (square) |
| `ball_speed` | 4 px/frame | Ball movement speed |
| `left_paddle1` | 6 px | X position of Player 1 paddle |
| `left_paddle2` | 629 px | X position of Player 2 paddle |

---

## Authors

**Khalid Abdelaziz**
German University in Cairo — Faculty of Information Engineering & Technology
*May 2023*

---

## License

This project is open source and available under the [MIT License](LICENSE).
