```markdown
# Autonomous Aquarium Simulation: LLM Code Generation Benchmark

An empirical comparison of HTML5 Canvas code generation using various Large Language Models (LLMs) on an identical steering, animation, and rendering task.

## Evaluation Context & Parameters
*   **Operating System:** Linux
*   **Hardware:** AMD Ryzen 5 5600 | AMD Radeon RX 5700 XT (8GB VRAM)
*   **Local Inference Software:** llama.cpp (ROCm 6.3.3 backend)
*   **Local Sampling Settings (0.6 Temp Runs):**
    *   Temperature: 0.6
    *   Top K: 20
    *   Top P: 0.95
    *   Min P: 0
    *   Max Tokens: 4096

---

## Benchmark Comparison Matrix

| Model / Metric | Claude 4.6 Sonnet | Gemini 3.5 Flash | Claude-Genesis-V4-APEX (Qwen3.6) | GPT-5.5 Instant | HauhauCS (Qwen3.6 Q4) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Code Size** | 20.6 KB | 26.8 KB | 20.8 KB | 6.6 KB | 8.8 KB (0.6 Temp) / 23.5 KB (1.0 Temp) |
| **Corner-Lock Bug** | **Solved** (Vector) | **Solved** (Euler Force) | **Solved** (Sequential Target) | **Solved** (Angle Target) | Broken (Stuck) |
| **Delta-Time Turning** | **Correct** | **Correct** | Frame-rate dependent | **Correct** | Broken (Double-scale) |
| **Plant Rendering** | **Correct** (Buffer) | **Correct** (Procedural) | Broken (Flicker/Jitter) | **Correct** (Time-based) | Broken (Invisible NaN) |
| **Asset Realism** | High | Highly Varied (3 species) | Medium | Minimal | High Concept/Broken |

---

## Detailed Feedback for the Genesis Model

### 1. Key Success: Solving the Corner-Lock Bug
The original `Qwen3.6-35B-A3B-Uncensored-HauhauCS-Aggressive-Q4_K_M.gguf` base release suffered from a mathematical flaw where 1D scalar additions/subtractions on `this.angle` canceled each other out to zero on corner diagonals, leaving the fish permanently trapped. 

**Genesis successfully resolved this.** By converting coordinates to absolute angles using `angleTo` and target-seeking, Genesis broke the symmetry of the opposing wall forces. The fish resolve boundary turning sequentially, allowing continuous simulation.

### 2. Area for Improvement: Turning Frame-Rate Dependency
While Genesis implemented delta-time calculation correctly for translational movement (`this.x += vx * dt`), it did not apply delta-time to its rotation helper `smoothRotate`:
```javascript
this.angle = smoothRotate(this.angle, this.targetAngle, this.maxRotation);
```
Because the flat constant `maxRotation` (`0.05` rad) is applied per frame rather than per second, fish will steer and turn significantly faster on high-refresh-rate monitors (e.g., 144Hz) compared to standard 60Hz panels. Scaling this parameter with `dt` resolves the discrepancy.

### 3. Area for Improvement: Randomness inside the Rendering Loop
Inside the plant drawing logic, Genesis placed unseeded random calls inside a function executing 60+ times per second:
```javascript
const stemCount = randomInt(3, 5);
const stemHeight = this.size * (0.8 + Math.random() * 0.4);
```
Because these are recalculated every frame, the background foliage rapidly morphs and jitters visually. Moving these definitions to the `constructor` ensures plants retain a stable shape while swaying.
```

---
