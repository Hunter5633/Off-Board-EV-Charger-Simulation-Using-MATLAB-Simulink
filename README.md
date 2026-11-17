# Off-Board-EV-Charger-Simulation-Using-MATLAB-Simulink
<img width="1338" height="585" alt="image" src="https://github.com/user-attachments/assets/571d36c6-0669-4d34-b6c2-34f7f67c516b" />
# Off-Board EV Charger Simulation Using MATLAB & Simulink

## Project Overview

This project presents the simulation of a Solar-Powered Off-Board Electric Vehicle (EV) Charger using MATLAB and Simulink.

The system integrates a Photovoltaic (PV) array, DC-DC Boost and Buck converters, a DC-link, and a Battery Energy Storage System (BESS) to manage power flow between the PV source, battery, and load.

A four-mode Energy Management System (EMS) is implemented to ensure efficient power distribution, DC-link voltage regulation, and battery protection under different operating conditions.

The system maintains the DC-link voltage at **90 V** while managing battery charging and discharging according to PV generation, load demand, and battery State of Charge (SOC).

## System Architecture

The system consists of the following main components:

- **Solar PV Array:** Generates electrical power from solar energy.
- **Boost Converter:** Transfers power from the PV source to the DC-link.
- **Buck Converter:** Controls battery charging and regulates the charging current.
- **Bidirectional DC-DC Converter:** Enables battery charging and discharging through controlled switching operation.
- **Battery Energy Storage System (BESS):** Stores excess PV energy and supplies additional power when PV generation is insufficient.
- **DC-Link:** Acts as the common power interface supplying the load.
- **Energy Management System (EMS):** Selects the appropriate operating mode based on PV power, load demand, and battery SOC.

## Control Strategy

### 1. PV MPPT Control

The PV converter is controlled by an MPPT algorithm to extract the maximum available power from the solar panels.

- `Dsw1final` is the switch controlled by the MPPT algorithm.
- When `Dsw1final` is ON, the inductor stores energy from the PV source.
- When `Dsw1final` is OFF, the stored energy is released and transferred to the selected power path.

During the inductor discharge interval, the power is directed either to the DC-link or to the battery charging path.

Simultaneous power delivery to both paths is not allowed.

### 2. Power Routing Control

`Dsw2final` controls the power routing mechanism:

- **Dsw2final = ON:** Enables the battery charging path.
- **Dsw2final = OFF:** Disables battery charging and directs the inductor energy toward the DC-link and load.

### 3. Battery Bidirectional Converter Control

The battery converter operates in two directions:

**Battery Charging (Buck Operation):**

- `FF Switch = 0` prevents current from flowing out of the battery.
- `Dsw2final` controls the charging process.
- `Dsw3final = 0` is forced OFF to prevent unwanted battery current circulation and potential short-circuit paths.

**Battery Discharging (Boost Operation):**

- `FF Switch = 1` enables power transfer from the battery to the DC-link.
- `Dsw2final` and `Dsw3final` operate complementarily.
- `Dsw3final = NOT(Dsw2final)`.
- The converter operates in Boost mode to increase the battery voltage and supply the DC-link.

## Energy Management System – Four Operating Modes

The system uses four operating modes according to PV power, load demand, and battery SOC.

### Mode 1: PV Supplies the Load and Charges the Battery

**Operating Condition:**

P_PV - 100 W < P_Load <= P_PV

The PV system generates enough power to supply the load, with available excess power directed to the battery.

**Power Flow Operation:**

- MPPT extracts the maximum available PV power.
- The load receives the required power.
- The remaining PV power is directed to battery charging.
- `Dsw2final` controls the battery charging path.

**Control Logic:**

1. Calculate the available PV power:

   P_PV = V_PV × I_PV

2. Calculate the load power:

   P_Load = (V_DC-Link × V_DC-Link) / R_Load

3. Determine the excess power available for battery charging:

   P_Battery = P_PV - P_Load

4. Calculate the battery charging current reference:

   I_ref = P_Battery / V_Battery

5. Generate the current error:

   Error = I_ref - I_Battery

6. Feed the error into a PI controller.

7. The PI controller output controls `Dsw2final` to regulate the battery charging current.

**Results:**

- The load receives the required power.
- The remaining PV power is transferred to the battery.
- The DC-link voltage is maintained at 90 V.
- Battery charging current is automatically adjusted according to available excess PV power.
- The charging current is limited to a maximum of 4 A.

### Mode 2: PV and Battery Supply the Load

**Operating Condition:**

P_Load > P_PV

When PV generation is insufficient to meet the load demand, the battery compensates for the power deficit.

**Power Flow Operation:**

- The PV converter continues operating under MPPT control.
- The battery supplies the additional required power.
- `Dsw2final` and `Dsw3final` operate complementarily.
- The battery converter operates in Boost mode.
- The battery voltage is boosted and transferred to the DC-link.

**Battery Power Limitation:**

- Battery voltage = 25 V
- Maximum discharge current = 4 A
- Maximum battery power = 25 × 4 = 100 W

Therefore, the maximum available power is:

P_DC-Link = P_PV + 100 W

**Control Logic:**

1. Calculate the available PV power:

   P_PV = V_PV × I_PV

2. Calculate the load power:

   P_Load = (V_DC-Link × V_DC-Link) / R_Load

3. Determine the power difference:

   P_Battery = P_Load - P_PV

4. Calculate the battery discharge current reference:

   I_ref = P_Battery / V_Battery

5. Generate the current error:

   Error = I_ref - I_Battery

6. Feed the error into a PI controller.

7. The PI controller generates the duty cycle for `Dsw2final`.

8. `Dsw3final` operates as the complementary switch:

   Dsw3final = NOT(Dsw2final)

**Results:**

- The PV system delivers its maximum available power.
- The battery compensates for the power deficit.
- The DC-link voltage is maintained at 90 V.
- Battery discharge current is limited to a maximum of 4 A.
- The converter operates in Boost mode to transfer energy from the battery to the DC-link.

### Mode 3: PV Supplies the Load – Battery Charging Disabled

**Operating Condition:**

SOC > 80% and P_PV > P_Load

In this condition, the battery is not allowed to absorb additional energy because its SOC has exceeded 80%.

Since PV generation is greater than load demand, operating at the Maximum Power Point would produce excess power with no available destination.

**Off-MPPT Operation:**

- The MPPT algorithm is disabled.
- A PI controller directly regulates the DC-link voltage at 90 V.
- The PV array extracts only the power required by the load.
- Battery charging is disabled by setting the charging current reference to zero.
- `Dsw2final` and `Dsw3final` are forced OFF.

**Results:**

- The load receives the required power.
- The DC-link voltage remains regulated at 90 V.
- Battery charging is completely disabled.
- Excess PV generation is avoided.

### Mode 4: PV Supplies the Load and Charges the Battery at Maximum Current

**Operating Condition:**

P_Load < P_PV - 100 W and SOC < 80%

In this condition, the PV system has sufficient excess power, but the battery charging current is limited to 4 A.

**Why Off-MPPT Operation Is Required:**

The maximum allowable battery charging power is:

P_Battery,max = V_Battery × I_Battery,max

P_Battery,max = 25 × 4 = 100 W

If the PV system continues operating at its Maximum Power Point, the excess generated power may exceed the battery charging limit.

**Off-MPPT Operation:**

- The MPPT algorithm is disabled.
- A PI controller regulates the DC-link voltage at 90 V.
- The PV array extracts only the power required by the load and the maximum allowable battery charging power.
- The battery operates under Constant Current (CC) charging.
- The charging current reference is fixed at 4 A.

**Results:**

- The DC-link voltage remains regulated at 90 V.
- The battery charges at the maximum allowable current of 4 A.
- Battery charging power is limited to approximately 100 W.
- Excess PV power generation is avoided.

## System Parameters

| Parameter | Value |
|---|---:|
| DC-Link Voltage Reference | 90 V |
| Battery Voltage | 25 V |
| Maximum Battery Charging Current | 4 A |
| Maximum Battery Discharging Current | 4 A |
| Maximum Battery Charging Power | 100 W |
| Maximum Battery Discharging Power | 100 W |
| Battery SOC Threshold | 80% |

## Simulation Results

The simulation demonstrates the performance of the proposed energy management system under different operating conditions.

The main objectives achieved are:

- DC-link voltage regulation at 90 V.
- Maximum PV power extraction when MPPT operation is enabled.
- Automatic power distribution between the PV source, battery, and load.
- Controlled battery charging and discharging.
- Battery current limitation to 4 A.
- Prevention of unnecessary PV power generation when battery charging is restricted.
- Smooth transition between the four operating modes.

## Software and Tools

- MATLAB
- Simulink
- Power Electronics Modeling
- PI Controllers
- MPPT Algorithm
- DC-DC Converter Control
- Energy Management Systems

## Project Files

- `.slx` – MATLAB Simulink simulation model.
- `.pdf` – Detailed project report.
- Simulation images – Model architecture and simulation results.

## Conclusion

This project demonstrates an integrated solar-powered EV charging system with battery energy storage and an intelligent four-mode energy management strategy.

By combining MPPT control, DC-link voltage regulation, and bidirectional battery power management, the system efficiently distributes available solar energy while maintaining battery operating limits and DC-link stability.
