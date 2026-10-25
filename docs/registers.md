# Register reference

The register map read by this configuration, with the manual description, the
scaling, the tier and the firmware gate for each entity. Device register =
30001 + PDU address.

| entity_id | PDU address | device register | manual description | manual type | scale | HA `data_type` | tier (`scan_interval`) | firmware gate | notes |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| `sensor.ingeteam_stop_event_code` | 9 | 30010 | Stop Event [Note 1] | UINT16 | 1 | `uint16` | B (300 s) | — | Stop-event code; interpret with ABH2010IMC14 |
| `sensor.ingeteam_alarm_code` | 10 | 30011 | Alarms [Note 1] | UINT32 | 1 | `uint32` | B (300 s) | — | Two words, low word first (`swap: word`). This word order is inherited from the replaced custom component, is not documented by the manual, and was not confirmed (the validation sample was zero); confirm it with a non-zero alarm. Interpret with ABH2010IMC14 |
| `sensor.ingeteam_status_raw` | 15 | 30016 | Inverter Status [Note 2] | UINT16 | 1 | `uint16` | A (30 s) | — | Raw enum; mapped to text by the Status template sensor |
| `sensor.ingeteam_waiting_time_to_connect_to_grid` | 16 | 30017 | Waiting Time to Connect to Grid [sec] | UINT16 | 1 | `uint16` | B (300 s) | — |  |
| `sensor.ingeteam_battery_voltage` | 17 | 30018 | Battery. Voltage [V x 10] | UINT16 | 0.1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_battery_current` | 18 | 30019 | Battery. Current [A x 100] | INT16 | 0.01 | `int16` | A (30 s) | — | Signed |
| `sensor.ingeteam_battery_power_raw` | 19 | 30020 | Battery. Power [W] | INT16 | 1 | `int16` | A (30 s) | — | Backing register for the battery charging/discharging power templates |
| `sensor.ingeteam_battery_state_of_charge` | 20 | 30021 | Battery. SOC [%] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_battery_state_of_health` | 21 | 30022 | Battery. SOH [%] | UINT16 | 1 | `uint16` | C (1800 s) | — |  |
| `sensor.ingeteam_battery_charging_voltage` | 22 | 30023 | Battery. Charging Voltage [V x 10] | UINT16 | 0.1 | `uint16` | C (1800 s) | — |  |
| `sensor.ingeteam_battery_discharging_voltage` | 23 | 30024 | Battery. Discharging Voltage [V x 10] | UINT16 | 0.1 | `uint16` | C (1800 s) | — |  |
| `sensor.ingeteam_battery_max_charging_current` | 24 | 30025 | Battery. Max. Charging Current [A x 100] | UINT16 | 0.01 | `uint16` | C (1800 s) | — |  |
| `sensor.ingeteam_battery_max_discharging_current` | 25 | 30026 | Battery. Max. Discharging Current [A x 100] | UINT16 | 0.01 | `uint16` | C (1800 s) | — |  |
| `sensor.ingeteam_battery_status_raw` | 26 | 30027 | Battery. Status [Note 3] | UINT16 | 1 | `uint16` | A (30 s) | — | Raw enum; mapped to text by the Battery Status template sensor |
| `sensor.ingeteam_battery_temp` | 27 | 30028 | Battery. Temperature [ºC x 10] | INT16 | 0.1 | `int16` | A (30 s) | — | Signed |
| `sensor.ingeteam_battery_bms_alarm` | 28 | 30029 | Battery. BMS Alarms [Note 4] | UINT16 | 1 | `uint16` | B (300 s) | — | Bitfield exposed as a raw integer |
| `sensor.ingeteam_battery_discharge_limitation_reason_raw` | 29 | 30030 | Battery. Discharge Limitation Reason [Note 9] | UINT16 | 1 | `uint16` | A (30 s) | — | Raw enum; mapped to text by the Battery Discharge Limitation Reason template sensor |
| `sensor.ingeteam_battery_voltage_internal_sensor` | 30 | 30031 | Battery. Voltage Internal Sensor [V x 10] | UINT16 | 0.1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv1_voltage` | 31 | 30032 | PV1. Voltage [V] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv1_current` | 32 | 30033 | PV1. Current [A x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv1_power` | 33 | 30034 | PV1. Power [W] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv2_voltage` | 34 | 30035 | PV2. Voltage [V] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv2_current` | 35 | 30036 | PV2. Current [A x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_pv2_power` | 36 | 30037 | PV2. Power [W] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_active_power` | 37 | 30038 | Inverter. Active Power [W] | INT16 | 1 | `int16` | A (30 s) | — | Signed |
| `sensor.ingeteam_reactive_power` | 38 | 30039 | Inverter. Reactive Power [Var, Note 5] | INT16 | 1 | `int16` | A (30 s) | — | Sign convention in Note 5 |
| `sensor.ingeteam_power_factor_cosph` | 39 | 30040 | Inverter. Cosφ [x1000, Note 6] | INT16 | 0.001 | `int16` | A (30 s) | — | Absolute value x1000; take the sign from reactive power |
| `sensor.ingeteam_active_power_reduction_ratio` | 40 | 30041 | Active Power Reduction Ratio [% x10] | UINT16 | 0.1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_ap_reduction_reason_raw` | 41 | 30042 | Active Power Reduction Reason [Note 7] | UINT16 | 1 | `uint16` | A (30 s) | — | Raw enum; mapped to text by the Active Power Reduction Reason template sensor |
| `sensor.ingeteam_reactive_power_set_point_type` | 42 | 30043 | Reactive Power Set-Point Type [Note 8] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw enum |
| `sensor.ingeteam_critical_loads_voltage` | 43 | 30044 | Critical Loads. Voltage [V] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_critical_loads_current` | 44 | 30045 | Critical Loads. Current [A x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_critical_loads_frequency` | 45 | 30046 | Critical Loads. Frequency [Hz x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_critical_loads_active_power` | 46 | 30047 | Critical Loads. Active Power [W] | INT16 | 1 | `int16` | A (30 s) | — | Signed |
| `sensor.ingeteam_critical_loads_reactive_power` | 47 | 30048 | Critical Loads. Reactive Power [Var, Note 5] | INT16 | 1 | `int16` | A (30 s) | — | Sign convention in Note 5 |
| `sensor.ingeteam_internal_meter_voltage` | 48 | 30049 | Internal Wattmeter Grid. Voltage [V] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_internal_meter_current` | 49 | 30050 | Internal Wattmeter Grid. Current [A x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_internal_meter_frequency` | 50 | 30051 | Internal Wattmeter Grid. Frequency [Hz x100] | UINT16 | 0.01 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_internal_active_power` | 51 | 30052 | Internal Wattmeter Grid. Active Power [W] | INT16 | 1 | `int16` | A (30 s) | — | Signed grid meter; positive = export, negative = import |
| `sensor.ingeteam_internal_reactive_power` | 52 | 30053 | Internal Wattmeter Grid. Reactive Power [Var, Note 5] | INT16 | 1 | `int16` | A (30 s) | — | Sign convention in Note 5 |
| `sensor.ingeteam_internal_power_factor_cosph` | 53 | 30054 | Internal Wattmeter Grid. Cosφ [x1000, Note 6] | INT16 | 0.001 | `int16` | A (30 s) | — | Absolute value x1000; take the sign from reactive power |
| `sensor.ingeteam_dc_bus_voltage` | 54 | 30055 | DC Bus Voltage [V] | UINT16 | 1 | `uint16` | A (30 s) | — |  |
| `sensor.ingeteam_temperature_module_1` | 55 | 30056 | Reserved | — | 0.1 (unverified) | `int16` | A (30 s) | — | Reserved in the manual, so it has no manual type and no manual scale. Type `int16` and scale `0.1` are taken from the reference installation and are unverified; delete if constant or absurd |
| `sensor.ingeteam_temperature_module_2` | 56 | 30057 | Reserved | — | 0.1 (unverified) | `int16` | A (30 s) | — | Reserved in the manual; type and scale taken from the reference installation, unverified. Same as PDU 55 |
| `sensor.ingeteam_internal_temperature` | 57 | 30058 | Temperature. Internal Inverter [ºC x10] | INT16 | 0.1 | `int16` | A (30 s) | — | No firmware gate. The 23/09/2022 document revision added this register |
| `sensor.ingeteam_rms_differential_current` | 61 | 30062 | RMS Differential Current [mA x10] | UINT16 | 0.1 | `uint16` | C (1800 s) | _P | Gate `_P`; the 02/01/2023 document revision changed the scale to mA x10 |
| `sensor.ingeteam_do_1_status_raw` | 62 | 30063 | Digital Output 1. Status [0: OFF, 1:ON] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw 0/1; mapped to Off/On by the Digital Output 1. Status template sensor |
| `sensor.ingeteam_do_2_status_raw` | 63 | 30064 | Digital Output 2. Status [0: OFF, 1:ON] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw 0/1; mapped to Off/On by the Digital Output 2. Status template sensor |
| `sensor.ingeteam_di_drm_status_raw` | 64 | 30065 | Digital Input DRM0. Status [0: OFF, 1:ON] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw 0/1; mapped to Off/On by the Digital Input DRM0 Status template sensor |
| `sensor.ingeteam_di_2_status_raw` | 65 | 30066 | Digital Input 2. Status [0: OFF, 1:ON] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw 0/1; mapped to Off/On by the Digital Input 2. Status template sensor |
| `sensor.ingeteam_di_3_status_raw` | 66 | 30067 | Digital Input 3. Status [0: OFF, 1:ON] | UINT16 | 1 | `uint16` | C (1800 s) | — | Raw 0/1; mapped to Off/On by the Digital Input 3. Status template sensor |
| `sensor.ingeteam_total_loads_power` | 78 | 30079 | Total Loads. Active Power (Critical + No Critical) [W] | UINT16 | 1 | `uint16` | A (30 s) | _L | Firmware-gated |
| `sensor.ingeteam_pv_external_power` | 79 | 30080 | External PV. Power (Ingeteam Inverters) [W] | UINT16 | 1 | `uint16` | C (1800 s) | _L | Firmware-gated |
| `sensor.ingeteam_ev_power` | 80 | 30081 | EV Charger. Active Power [W] | INT16 | 1 | `int16` | C (1800 s) | _N | Firmware-gated; signed |

## Source

Input-registers block beginning at device register 30001 (PDU address 0), Modbus
unit id 1, function `0x04` (read input registers). Manual: INGECON SUN STORAGE
1Play TL M, `ABH2010IMB08`, revision `_I` (23/04/2025), "Input registers, Li-ion
batteries". Alarm and stop-event codes are interpreted with `ABH2010IMC14`
(Alarm Interpretation and Troubleshooting Guide). The map was revalidated against
a live INGECON SUN STORAGE 1Play TL M running firmware `ABH1007AE`, where all 54
sensors decoded into the expected physical range. 16-bit signed values use
`data_type: int16`. The manual documents `alarm_code` only as `UINT32`; the
low-word-first order used here (`uint32` plus `swap: word`) is inherited from the
replaced custom component, is not manual-verified, and should be confirmed with a
non-zero alarm.

## Discrepancy to verify on your unit

The manual marks PDU 55 and 56 (`sensor.ingeteam_temperature_module_1`,
`sensor.ingeteam_temperature_module_2`) as reserved, so those two registers have
no manual type and no manual scale. The reference installation reads plausible
non-zero temperatures there, so the entries are kept with the type and scale
taken from the reference installation, but treat them as unverified against the
manual. If they read as constant or absurd values on your inverter, delete them.

PDU 57 (`sensor.ingeteam_internal_temperature`) is documented as "Temperature.
Internal Inverter". It was added in a later document revision (23/09/2022) and
carries no firmware gate.

## Firmware-gated registers

The manual's "Since FW Ver." column gates device 30069 (`_B`), 30070-30073
(`_F`), 30074-30077 (`_K`), 30078-30080 (`_L`) and 30081 (`_N`). Inverter
variants that do not implement them (no external wattmeter, no EV charger, older
firmware, non-TL variants) will time out or return garbage on those reads. That
is expected; delete the entries your hardware does not expose.

The gates that apply to what this distribution reads are PDU 61 (`_P`), PDU 78
and 79 (`_L`) and PDU 80 (`_N`). The manual also gates device 30069 (`_B`) and
devices 30070-30077 (`_F`, `_K`), which this distribution does not read. Device
30068 shows `-----` in that column; that marker is not a gate.

## Not read / reserved by the manual

Current date and time (30001-30006), total operation time (30007), registers
reserved for Ingeteam 30008-30009 and 30068, code 1/2/3 (30013-30015), 30059
(reserved), positive/negative isolation resistance (30060-30061), and 30082 and
above.

## Read-only

The distribution contains only input-register reads. There are no coils, no
holding registers, and no write or control registers.

## Conventions from the manual

Reactive power sign: positive values mean the current is lagging the voltage,
negative values mean the current is leading the voltage. Cos-phi is transmitted
as an absolute value multiplied by 1000, so the sign must be taken from reactive
power. Battery BMS alarms and flags are bitfields exposed as raw integers.

## Enum maps

The enum maps used by the template sensors, transcribed from the manual notes.

### Inverter status (Note 2)

| Value | Meaning |
| --- | --- |
| 0 | Inverter Stopped |
| 1 | Starting |
| 2 | Off-grid |
| 3 | On-grid |
| 4 | On-grid (Standby Battery) |
| 5 | Waiting to connect to Grid |
| 6 | Critical Loads Bypassed to Grid |
| 7 | Emergency Charge from PV |
| 8 | Emergency Charge from Grid |
| 9 | Inverter Locked waiting for Reset |
| 10 | Error Mode |

### Battery status (Note 3)

| Value | Meaning |
| --- | --- |
| 0 | Standby |
| 1 | Discharging |
| 2 | Constant Current Charging |
| 3 | Constant Voltage Charging |
| 4 | Floating |
| 5 | Equalizing |
| 6 | Error Communication with BMS |
| 7 | No Configured |
| 8 | Capacity Calibration (Step 1) |
| 9 | Capacity Calibration (Step 2) |
| 10 | Standby Manual |

### Active power reduction reason (Note 7)

| Value | Meaning |
| --- | --- |
| 0 | No limitation |
| 1 | Communication |
| 2 | PCB Temperature |
| 3 | Heat Sink Temperature |
| 4 | Pac vs Fac Algorithm |
| 5 | Soft Start |
| 6 | Charge Power Configured |
| 7 | PV Surplus injected to the Loads |
| 8 | Pac vs Vac Algorithm |
| 9 | Battery Power Limited |
| 10 | AC Grid Power Limited |
| 11 | Self-Consumption Mode |
| 12 | High Bus Voltage Protection |
| 13 | LVRT or HVRT Process |
| 14 | Nominal AC Current |
| 15 | Grid Consumption Protection |
| 16 | PV Surplus Injected to the Grid |

### Reactive power set-point type (Note 8)

| Value | Meaning |
| --- | --- |
| 0 | Cos(phi) Configuration |
| 1 | Qac Communication |
| 2 | Cos(phi) Communication |
| 3 | Qac vs Vac Algorithm |
| 4 | Cos(phi) vs Pac Algorithm |

### Battery charge/discharge limitation reason (Note 9)

| Value | Meaning |
| --- | --- |
| 0 | No limitation |
| 1 | Heat Sink Temperature |
| 2 | PT100 Temperature |
| 3 | Low Bus Voltage Protection |
| 4 | Battery Settings |
| 5 | BMS Communication |
| 6 | SOC Max Configured |
| 7 | SOC Min Configured |
| 8 | Maximum Battery Power |
| 9 | Modbus command |
| 10 | Digital Input 2 |
| 11 | Digital Input 3 |
| 12 | PV Charging scheduling |
| 13 | EMS Strategy |
