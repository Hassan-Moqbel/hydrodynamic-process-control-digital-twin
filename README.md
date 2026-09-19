# Hydrodynamic Level Process Control (Digital Twin & Intelligent Control)

![PLC](https://img.shields.io/badge/PLC-Siemens_S7--1200_%2F_S7--1500-00599C?style=for-the-badge)
![TIA Portal](https://img.shields.io/badge/IDE-TIA_Portal_V17-A8B9CC?style=for-the-badge)
![Digital Twin](https://img.shields.io/badge/Simulation-Factory_I%2FO_3D-4B0082?style=for-the-badge)
![Languages](https://img.shields.io/badge/Logic-Structured_Control_Language_(SCL)-28A745?style=for-the-badge)
![Advanced Control](https://img.shields.io/badge/Control-Fuzzy_Logic_%7C_NARX_%7C_IMC-FF6F00?style=for-the-badge)
![Closed-Loop Process Control](https://img.shields.io/badge/Application-Closed--Loop_Process_Control-6f42c1?style=for-the-badge)
![License](https://img.shields.io/badge/License-MIT-blue?style=for-the-badge)

## Executive Overview
Modern process industries require exceptional robustness in fluid regulation systems, demanding transition from classical PID to model-predictive and intelligent control strategies. This project functions as an advanced **Hydrodynamic Level Process Control Simulation**, utilizing Factory I/O as a high-fidelity 3D digital twin. By orchestrating five distinct controller architectures—ranging from classical On-Off hysteresis to Artificial Neural Networks (NARX)—this repository validates theoretical non-linear control mathematics against simulated real-world fluid dynamics.

> [!CAUTION]
> **Industrial Process Safety & Control Reliability Callout**
> Simulating high-inertia hydraulic processes uncovers critical safety vulnerabilities:
> - **Hydrostatic Overfill & Spill Hazards:** Failure in feedback loops or integration windup can result in tank overflow, requiring hard-coded gravity-drain interlocks and physical limit switches.
> - **Actuator Saturation & Integrator Windup:** Integral action in PIDs must be strictly clamped (`u_min`, `u_max`) to prevent deep mathematical windup when the proportional valve reaches 100% stroke.
> - **Pump Cavitation:** Dry-running centrifugal pumps destroys impellers instantly; low-level limit switches must interlock the pump motor contactors.

## System Highlights
- **Multi-Controller Benchmarking Engine**: Real-time evaluation of five distinct control paradigms:
  1. On-Off Hysteresis
  2. Classical PID (Velocity Algorithm)
  3. Internal Model Control (IMC-PID)
  4. Takagi-Sugeno Fuzzy-PI Logic
  5. Artificial Neural Networks (NARX)
- **3D Digital Twin Physical Simulation**: Factory I/O maps PLCSIM tags to continuous level sensors (0-10V scaled feedback) and proportional modulating valves.
- **Advanced SCL Automation**: All algorithms are programmed using high-level Siemens Structured Control Language (SCL) for deterministic PLC execution.

## System Architecture & Control Loop Diagram

```mermaid
flowchart TD
    SP([Level Setpoint SP(t)]) --> ENGINE[Controller Selection Engine \nOn-Off / PID / IMC / Fuzzy / NARX]
    
    ENGINE -->|Modulated Flow Rate u(t)| VALVE[Variable Inflow Actuator \nProportional Valve]
    VALVE --> PLANT[Nonlinear Hydraulic Plant \nGravitational Discharge]
    PLANT --> LEVEL[Tank Liquid Level h(t)]
    
    LEVEL --> SENSOR[Continuous Hydrostatic \nLevel Sensor]
    SENSOR -->|Feedback Signal PV(t)| ERROR[Error Junction \ne(t) = SP - PV]
    
    ERROR --> ENGINE
```

## Rigorous Theoretical & Mathematical Models

### 1. Non-Linear Hydrodynamic Mass Balance & Torricelli's Law
The volumetric rate of change equals the difference between inflow and gravity-driven orifice discharge:
$$A(h) \frac{dh(t)}{dt} = Q_{in}(t) - a \sqrt{2 g h(t)}$$
*(Where $A(h)$ is tank cross-sectional area, $Q_{in}$ is inflow rate, $a$ is orifice area, and $g$ is gravity).*

### 2. First-Order Linearized Transfer Function
Linearizing around a nominal operating level $h_0$ yields a standard first-order plant:
$$G(s) = \frac{\Delta H(s)}{\Delta Q_{in}(s)} = \frac{K_p}{\tau s + 1}, \quad \text{where } \tau = \frac{A}{a} \sqrt{\frac{2 h_0}{g}}, \quad K_p = \frac{\tau}{A}$$

### 3. Internal Model Control (IMC) Synthesis & Analytical Tuning
IMC achieves robust tuning by absorbing the inverted plant model into a low-pass filter:
$$Q_{IMC}(s) = \tilde{G}^{-1}(s) f(s) = \frac{\tau s + 1}{K_p (\lambda s + 1)}$$

### 4. Takagi-Sugeno Fuzzy Logic Rule Consequent
The continuous control effort evaluated from $M$ fuzzy rules via weighted averages:
$$u_{TS} = \frac{\sum_{i=1}^M w_i (p_{i0} + p_{i1} e + p_{i2} \dot{e})}{\sum_{i=1}^M w_i}$$

### 5. Discrete Velocity SCL PID Formulation with Anti-Windup Clamping
To prevent integrator windup and ensure smooth manual/auto transitions, the velocity form accumulates incremental adjustments:
$$u[k] = \text{clamp}\left(u[k-1] + K_p(e[k] - e[k-1]) + K_i T_s e[k] + \frac{K_d}{T_s}(e[k] - 2e[k-1] + e[k-2]), u_{min}, u_{max}\right)$$

## Comparative Performance Benchmark Matrix

| Control Strategy | Rise Time ($t_r$) | Settling Time ($t_s$) | Peak Overshoot ($M_p$) | Steady-State Error ($e_{ss}$) | CPU Computational Burden |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **On-Off Hysteresis** | Extremely Fast | N/A (Oscillates) | High | High (Bounded) | Very Low |
| **Classical PID** | Moderate | Moderate | Medium | Zero | Low |
| **IMC-PID** | Slow/Controlled | Fast | None | Zero | Low |
| **Takagi-Sugeno Fuzzy-PI** | Fast | Very Fast | Minimal | Zero | High (Fuzzification/Inference) |
| **NARX Neural Network** | Fast | Very Fast | None | Zero | Very High (Matrix Multiplication) |

## Virtual Commissioning & Reproduction Workflow
1. **Initialize TIA Portal**: Import the SCL blocks from `src/` or extract the `simulation/*.zap17` archive.
2. **Launch PLCSIM**: Start the virtual S7-1500 controller and download the program.
3. **Connect Digital Twin**: Open `simulation/level-controle.factoryio`. Map the internal I/O drivers to Siemens S7-PLCSIM.
4. **HMI Operation**: Use the Factory I/O virtual HMI panel to toggle between the 5 controller algorithms and inject setpoint disturbances.

## Authentic Artifacts Catalog
- **PLC Source Code (SCL / XML)**: Stored securely in [`src/`](src/).
- **Digital Twin Simulations**: Factory I/O scenes available in [`simulation/`](simulation/).
- **Architecture & Performance Visuals**: Captured inside [`docs/images/`](docs/images/).

---

**Hassan Moqbel Morshed Ghaleb**
Mechatronics Engineer | Mechanical Design & CAD (SolidWorks & AutoCAD) | Preventive Maintenance & Electromechanical Systems | Industrial Automation, Control Systems, Robotics & Intelligent Machines | CAD/FEA, Embedded Systems, Python & C++
[GitHub](https://github.com/Hassan-Moqbel) · [Facebook](https://www.facebook.com/share/1BqxAgVjHi/) · [LinkedIn](https://www.linkedin.com/in/hassan-moqbel)

## License
This project is licensed under the [MIT License](LICENSE).
