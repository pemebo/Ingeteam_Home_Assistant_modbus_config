# Ingeteam Modbus nativo para Home Assistant

**Español** · [English](#ingeteam-native-modbus-for-home-assistant)

Una configuración de Home Assistant de solo lectura que lee un inversor híbrido
Ingeteam directamente por Modbus TCP usando la integración `modbus` integrada,
sin componente personalizado. Expone 54 sensores de registro más 13 sensores
derivados y, para las entidades que define, reproduce los `entity_id`s del
popular componente personalizado `ingeteam_modbus`, de modo que la mayoría de los
usuarios que migran desde ese componente conservan sus paneles, estadísticas y la
configuración del panel de Energía. Dos excepciones requieren un paso manual;
consulte [Migración desde el componente personalizado](#migración-desde-el-componente-personalizado).

## Por qué el Modbus nativo en lugar del componente personalizado

El componente `ingeteam_modbus` sondea todo el bloque de registros cada 5 s desde
un trabajo del ejecutor sin protección de operaciones en curso. Cuando el
dispositivo es lento, los trabajos de sondeo se encolan más rápido de lo que se
vacían, bloquean hilos y pueden agotar el pool de ejecutores compartido de Home
Assistant: la interfaz queda en blanco y se requiere un reinicio manual. La
integración `modbus` integrada serializa las lecturas con un lock de asyncio, de
modo que una lectura lenta retrasa la siguiente en lugar de generar otro
trabajador bloqueado.

## Hardware de referencia

El mapa de registros sigue el manual de Ingeteam `ABH2010IMB08` para el INGECON
SUN STORAGE 1Play TL M (híbrido monofásico con batería, cargas críticas y
vatímetro interno). Se revalidó campo por campo contra una instalación real de
ese modelo con firmware `ABH1007AE`: cada uno de los 54 sensores se decodifica
dentro del rango físico esperado.

Se espera que otros modelos de Ingeteam funcionen porque comparten el mismo mapa
de registros de entrada, pero esto no está verificado. Los registros que un
modelo no exponga deben eliminarse; consulte `docs/registers.md`.

## Requisitos

- Home Assistant con la integración `modbus` por YAML.
- Modbus TCP accesible desde Home Assistant (puerto 502).
- El firewall TCP del inversor debe permitir Modbus en la interfaz que utilice.
  Habilítelo desde el panel local del inversor con la cuenta de instalador,
  `Comms` y luego `Firewall`. Estos pasos de firewall siguen la guía de
  configuración del componente personalizado que esta configuración reemplaza.
- Una dirección fija/estática para el inversor.

## Instalación

### Ruta A (recomendada, configuración limpia)

1. Cree un directorio `packages/` junto a `configuration.yaml`.
2. Copie en él los archivos de paquete desde `packages/`. `ingeteam_modbus.yaml`
   e `ingeteam_modbus_templates.yaml` son obligatorios;
   `ingeteam_modbus_energy.yaml` es opcional, pero es el que proporciona las
   entidades de importación/exportación de red, potencia de batería y potencia
   fotovoltaica que suele usar una configuración del panel de Energía.
3. Agregue lo siguiente a `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

4. Copie los valores de `secrets.example.yaml` en `secrets.yaml`, establezca la
   dirección real y reinicie Home Assistant.

### Ruta B (configuración existente que ya define `modbus:` o `template:`)

No agregue el paquete de un dominio que ya define. Home Assistant combina los
dominios basados en listas, y tanto `modbus` como `template` se combinan, así que
el riesgo real es que el mismo nombre de hub o el mismo entity id se produzca dos
veces: la segunda copia recibe el sufijo `_2` o se ignora. Combine las
definiciones en un solo lugar en vez de definirlas dos veces:

- Pegue el hub de `packages/ingeteam_modbus.yaml` como otro elemento de lista de
  su bloque `modbus:` existente.
- Pegue el bloque `- sensor:` de cada paquete de plantilla en su configuración de
  plantillas existente.

## Qué obtiene

Las entidades se agrupan por tipo. Los sensores de registro sin procesar se leen
por Modbus; el resto son sensores de plantilla derivados de ellos.

### Sensores de registro sin procesar (54)

| entity_id | unidad |
| --- | --- |
| `sensor.ingeteam_stop_event_code` | - |
| `sensor.ingeteam_alarm_code` | - |
| `sensor.ingeteam_status_raw` | - |
| `sensor.ingeteam_waiting_time_to_connect_to_grid` | s |
| `sensor.ingeteam_battery_voltage` | V |
| `sensor.ingeteam_battery_current` | A |
| `sensor.ingeteam_battery_power_raw` | - |
| `sensor.ingeteam_battery_state_of_charge` | % |
| `sensor.ingeteam_battery_state_of_health` | % |
| `sensor.ingeteam_battery_charging_voltage` | V |
| `sensor.ingeteam_battery_discharging_voltage` | V |
| `sensor.ingeteam_battery_max_charging_current` | A |
| `sensor.ingeteam_battery_max_discharging_current` | A |
| `sensor.ingeteam_battery_status_raw` | - |
| `sensor.ingeteam_battery_temp` | °C |
| `sensor.ingeteam_battery_bms_alarm` | - |
| `sensor.ingeteam_battery_discharge_limitation_reason_raw` | - |
| `sensor.ingeteam_battery_voltage_internal_sensor` | V |
| `sensor.ingeteam_pv1_voltage` | V |
| `sensor.ingeteam_pv1_current` | A |
| `sensor.ingeteam_pv1_power` | W |
| `sensor.ingeteam_pv2_voltage` | V |
| `sensor.ingeteam_pv2_current` | A |
| `sensor.ingeteam_pv2_power` | W |
| `sensor.ingeteam_active_power` | W |
| `sensor.ingeteam_reactive_power` | W |
| `sensor.ingeteam_power_factor_cosph` | - |
| `sensor.ingeteam_active_power_reduction_ratio` | % |
| `sensor.ingeteam_ap_reduction_reason_raw` | - |
| `sensor.ingeteam_reactive_power_set_point_type` | - |
| `sensor.ingeteam_critical_loads_voltage` | V |
| `sensor.ingeteam_critical_loads_current` | A |
| `sensor.ingeteam_critical_loads_frequency` | Hz |
| `sensor.ingeteam_critical_loads_active_power` | W |
| `sensor.ingeteam_critical_loads_reactive_power` | var |
| `sensor.ingeteam_internal_meter_voltage` | V |
| `sensor.ingeteam_internal_meter_current` | A |
| `sensor.ingeteam_internal_meter_frequency` | Hz |
| `sensor.ingeteam_internal_active_power` | W |
| `sensor.ingeteam_internal_reactive_power` | var |
| `sensor.ingeteam_internal_power_factor_cosph` | - |
| `sensor.ingeteam_dc_bus_voltage` | V |
| `sensor.ingeteam_temperature_module_1` | °C |
| `sensor.ingeteam_temperature_module_2` | °C |
| `sensor.ingeteam_internal_temperature` | °C |
| `sensor.ingeteam_rms_differential_current` | mA |
| `sensor.ingeteam_do_1_status_raw` | - |
| `sensor.ingeteam_do_2_status_raw` | - |
| `sensor.ingeteam_di_drm_status_raw` | - |
| `sensor.ingeteam_di_2_status_raw` | - |
| `sensor.ingeteam_di_3_status_raw` | - |
| `sensor.ingeteam_total_loads_power` | W |
| `sensor.ingeteam_pv_external_power` | W |
| `sensor.ingeteam_ev_power` | W |

### Sensores de texto/estado (9)

| entity_id | tipo |
| --- | --- |
| `sensor.ingeteam_status` | enum de estado del inversor |
| `sensor.ingeteam_battery_status` | enum de estado de la batería |
| `sensor.ingeteam_active_power_reduction_reason` | enum de motivo de reducción |
| `sensor.ingeteam_battery_discharge_limitation_reason` | enum de motivo de limitación |
| `sensor.ingeteam_digital_output_1_status` | estado de salida digital |
| `sensor.ingeteam_digital_output_2_status` | estado de salida digital |
| `sensor.ingeteam_digital_input_drm0_status` | estado de entrada digital |
| `sensor.ingeteam_digital_input_2_status` | estado de entrada digital |
| `sensor.ingeteam_digital_input_3_status` | estado de entrada digital |

### Sensores de potencia derivados (4)

| entity_id | unidad |
| --- | --- |
| `sensor.ingeteam_battery_charging_power` | W |
| `sensor.ingeteam_battery_discharging_power` | W |
| `sensor.ingeteam_pv_internal_total_power` | W |
| `sensor.ingeteam_pv_total_power` | W |

### Ayudantes de energía opcionales (4)

`packages/ingeteam_modbus_energy.yaml` es opcional. Proporciona las entidades de
importación/exportación de red, potencia de batería y potencia fotovoltaica que
suele usar una configuración del panel de Energía. Cárguelo para crear estas
cuatro entidades.

| entity_id | unidad |
| --- | --- |
| `sensor.ingeteam_export_power` | W |
| `sensor.ingeteam_import_power` | W |
| `sensor.ingeteam_battery_power` | W |
| `sensor.ingeteam_pv_power` | W |

## Presupuesto de tasa de peticiones (lea esto antes de cambiar cualquier cosa)

La integración nativa emite una petición Modbus por entidad, por lo que la tasa
de peticiones es la cantidad de entidades dividida por `scan_interval`. Con 54
entidades a los 15 s predeterminados, eso es aproximadamente 3.6 peticiones/s,
unas 18 veces lo que necesitaba el antiguo componente que leía en bloque, lo que
sobrepasó la capacidad del inversor y produjo transaction ids obsoletos,
reintentos y una conexión cerrada.

Esta distribución clasifica las entidades por la rapidez con que su valor puede
moverse de manera significativa:

- **Nivel A, `scan_interval: 30`** - valores eléctricos y registros de
  dirección/estado. Un registro de dirección que se retrasa hace que el panel
  parezca contradictorio consigo mismo, por lo que son los que se actualizan más
  rápido.
- **Nivel B, `scan_interval: 300`** - registros lentos de alarma/enum.
- **Nivel C, `scan_interval: 1800`** - constantes y hardware ausente.

Tasa en régimen estable: 36/30 + 4/300 + 14/1800, aproximadamente 1.22
peticiones/s, una reducción de alrededor del 66%.
`message_wait_milliseconds: 250` da al dispositivo un intervalo de silencio entre
peticiones para que una pausa transitoria no se convierta en cascada en una
acumulación.

Para quien acorte un intervalo, ~1.5 peticiones/s es una barrera de seguridad no
probada, no un techo medido. Las únicas cifras verificadas son 3.6 peticiones/s,
que falló, y las 1.22 peticiones/s anteriores, que es estable.

- Ejemplo de lo que conviene mover a un nivel más rápido: si automatiza en
  función de una salida digital, mueva `sensor.ingeteam_do_1_status_raw` del
  nivel C al nivel A, porque un retraso de 30 s en ese estado se hace visible en
  su automatización.
- Ejemplo de lo que conviene dejar como está:
  `sensor.ingeteam_battery_state_of_health` cambia a lo largo de semanas; déjelo
  en el nivel C.

## Limitaciones conocidas

- **Registros no verificados contra el manual:** el manual marca PDU 55 y 56
  (`sensor.ingeteam_temperature_module_1`, `sensor.ingeteam_temperature_module_2`)
  como reservados, por lo que no tienen tipo ni escala en el manual. La
  instalación de referencia lee valores plausibles allí, así que las entradas se
  conservan con el tipo y la escala tomados de la instalación de referencia, sin
  verificar contra el manual. Elimínelos si en su inversor se leen como valores
  constantes o absurdos.
- **Registros condicionados por firmware:** algunos registros existen solo en
  ciertas versiones de firmware o variantes de hardware (vatímetro externo,
  cargador de VE, firmware más reciente, variantes TL). Esas lecturas agotan el
  tiempo de espera o devuelven datos basura en hardware que no las implementa.
  Elimine las entradas que su hardware no exponga; consulte `docs/registers.md`
  para la condición de firmware de cada registro. Tenga en cuenta el
  emparejamiento: cinco sensores de plantilla tienen una fuente modbus `_raw`,
  por lo que eliminar `sensor.ingeteam_do_1_status_raw`,
  `sensor.ingeteam_do_2_status_raw`, `sensor.ingeteam_di_drm_status_raw`,
  `sensor.ingeteam_di_2_status_raw` o `sensor.ingeteam_di_3_status_raw` deja el
  sensor de texto correspondiente permanentemente `unavailable`. Elimine también
  el sensor de plantilla correspondiente, o agréguele una protección de
  disponibilidad.
- **Alcance de solo lectura:** esta distribución solo lee registros de entrada.
  No hay bobinas, ni registros de retención, ni registros de escritura o control.
- **Discrepancia de unidad heredada:** `sensor.ingeteam_reactive_power` (PDU 38)
  declara `unit_of_measurement: W`, mientras que el manual documenta el registro
  como potencia reactiva en `var`, tal como lo hacen correctamente los dos
  sensores de potencia reactiva hermanos. El valor y la escala son correctos;
  solo la etiqueta de la unidad es incorrecta. Se conserva tal cual para que la
  entidad no cambie de unidad para los usuarios que migran desde el componente
  personalizado. Para una instalación nueva sin historial que preservar, la
  configuración correcta es `unit_of_measurement: var` con
  `device_class: reactive_power`.
- **Derivados, no medidos:** los cuatro ayudantes de
  `packages/ingeteam_modbus_energy.yaml` son derivados. `sensor.ingeteam_pv_power`
  en particular es un valor derivado de balance que absorbe el residual que el
  mapa de registros no expone. El desfase depende del régimen: el valor derivado
  queda por encima del total medido durante la descarga y por debajo durante la
  carga, y la cifra de ~125 W es la magnitud en el régimen de descarga.
  `sensor.ingeteam_pv_total_power` sigue siendo el valor medido; apunte un
  indicador a esa entidad si se prefiere la medición.

## Resolución de problemas

| Síntoma | Causa probable | Acción |
| --- | --- | --- |
| `request ask for transaction_id=N but got id=M, Skipping` | La tasa de peticiones sobrepasó la capacidad del dispositivo. | Aumente `scan_interval` o `message_wait_milliseconds`. |
| `No response received ... CLOSING CONNECTION` y luego `Not connected` | Misma sobrecarga de tasa, o el servidor TCP del inversor se detuvo. | Aumente `scan_interval` o `message_wait_milliseconds`; la integración se reconecta por sí sola. |
| Un `address N FAILED` específico en los registros de E/S, condicionados (`_P`, `_L`, `_N`) y de hardware opcional | Esta variante de hardware no expone ese registro. | Elimine esas entradas de sensor. |
| Entidades `unavailable` después de reiniciar | host incorrecto en `secrets.yaml`, el unit id no es 1, o Modbus TCP está deshabilitado en el firewall. | Verifique los tres puntos; habilite Modbus en `Comms` -> `Firewall` con la cuenta de instalador. |
| Valores plausibles pero que no coinciden con el display propio del inversor | `data_type`, `swap` o `scale` incorrectos. | Verifique la entidad contra `docs/registers.md`. |

## Migración desde el componente personalizado

Esta configuración reproduce los `entity_id`s del componente personalizado
`ingeteam_modbus` para las entidades que define, de modo que en la mayoría de los
casos los paneles, las estadísticas y la configuración del panel de Energía
siguen funcionando tras el cambio. Dos situaciones requieren un paso manual, y el
último punto cubre la eliminación del propio componente.

### 1. Colisiones de entity-id

Si una entrada obsoleta del registro ya ocupa un id que la nueva configuración
necesita (esto es lo que ocurre cuando el componente antiguo dejó un huérfano,
como un sensor de temperatura interna duplicado), Home Assistant añade el sufijo
`_2` a la nueva entidad y cualquier panel que apunte al id limpio deja de
funcionar. Después del primer reinicio, revise Settings -> Devices & services ->
Entities, luego renombre o elimine la entrada obsoleta para que prevalezca el id
limpio, y reinicie.

### 2. Estadísticas de largo plazo tras un cambio de unidad

Seis sensores cambian de unidad respecto de lo que registraba el componente
antiguo:

| sensor | unidad antigua | unidad nueva |
| --- | --- | --- |
| `sensor.ingeteam_battery_temp` | `C` | `°C` |
| `sensor.ingeteam_temperature_module_1` | `C` | `°C` |
| `sensor.ingeteam_temperature_module_2` | `C` | `°C` |
| `sensor.ingeteam_internal_temperature` | `C` | `°C` |
| `sensor.ingeteam_critical_loads_reactive_power` | `Var` | `var` |
| `sensor.ingeteam_internal_reactive_power` | `Var` | `var` |

El `recorder` de Home Assistant rechaza la nueva unidad frente a la unidad ya
almacenada en `statistics_meta` y deja de acumular estadísticas de largo plazo
para esas series hasta que se actualicen los metadatos. La solución compatible es
la herramienta de desarrollador Statistics de Home Assistant (Settings ->
Developer tools -> Statistics): enumera las series afectadas y permite
corregirlas. En una instalación grande, el mismo efecto se logra actualizando
`statistics_meta` en la base de datos del `recorder` con Home Assistant detenido.

### 3. Ayudantes duplicados

Los cuatro ayudantes de energía opcionales (`sensor.ingeteam_export_power`,
`sensor.ingeteam_import_power`, `sensor.ingeteam_battery_power`,
`sensor.ingeteam_pv_power`) no llevan `unique_id`. Si conserva sus definiciones
antiguas con los mismos nombres y además carga
`packages/ingeteam_modbus_energy.yaml`, Home Assistant crea duplicados `..._2`
mientras las entidades antiguas siguen gobernando los paneles. Elimine las
definiciones antiguas de esos cuatro antes de cargar el paquete.

### 4. Eliminación del componente antiguo

Si `ingeteam_modbus` está instalado, sus entidades y esta configuración
colisionarían en los mismos `entity_id`s. Elimine el componente y su entrada de
configuración, reinicie y deje que la nueva configuración sea propietaria de los
ids.

## Créditos y licencia

Las direcciones de registro, las escalas y las tablas de enum provienen de la
documentación de Modbus publicada por Ingeteam (`ABH2010IMB08`, `ABH2010IMC14`).
El nombrado de entidades y la asignación de enum a texto siguen
`vortizhe/home-assistant-ingeteam-modbus`, un componente con licencia MIT
(`Copyright (c) 2022 Victor Ortiz`) que esta configuración reemplaza.

Esta distribución tiene licencia MIT (consulte `LICENSE`, que registra los avisos
de terceros). No está afiliada ni respaldada por Ingeteam.

---

# Ingeteam native Modbus for Home Assistant

A read-only Home Assistant configuration that reads an Ingeteam hybrid inverter
directly over Modbus TCP using the built-in `modbus` integration, with no custom
component. It exposes 54 register sensors plus 13 derived sensors, and for the
entities it defines it reproduces the `entity_id`s of the popular
`ingeteam_modbus` custom component, so most users migrating away from that
component keep their dashboards, statistics and Energy-dashboard setup. Two
exceptions need a manual step; see
[Migrating from the custom component](#migrating-from-the-custom-component).

## Why native instead of the custom component

The `ingeteam_modbus` component polls the whole register block every 5 s from an
executor job with no in-flight guard. When the device is slow, the polling jobs
queue faster than they drain, block threads, and can starve Home Assistant's
shared executor pool: the frontend goes blank and a manual restart is required.
The built-in `modbus` integration serialises reads with an asyncio lock, so a
slow read delays the next one instead of spawning another blocked worker.

## Reference hardware

The register map follows Ingeteam manual `ABH2010IMB08` for the INGECON SUN
STORAGE 1Play TL M (single-phase hybrid with battery, critical loads and internal
wattmeter). It was revalidated field by field against a live installation of
that model running firmware `ABH1007AE`: every one of the 54 sensors decodes into
the expected physical range.

Other Ingeteam models are expected to work because they share the same
input-register map, but this is unverified. Registers a model does not expose
must be deleted; see `docs/registers.md`.

## Requirements

- Home Assistant with the YAML `modbus` integration.
- Modbus TCP reachable from Home Assistant (port 502).
- The inverter's TCP firewall must allow Modbus on the interface you use. Enable
  it from the inverter's local dashboard with the installer account, `Comms`
  then `Firewall`. These firewall steps follow the setup guide of the custom
  component this configuration replaces.
- A fixed/static address for the inverter.

## Installation

### Path A (recommended, clean config)

1. Create a `packages/` directory next to `configuration.yaml`.
2. Copy the package files from `packages/` into it. `ingeteam_modbus.yaml` and
   `ingeteam_modbus_templates.yaml` are required; `ingeteam_modbus_energy.yaml`
   is optional but is what provides the grid import/export, battery power and PV
   power entities that an Energy-dashboard setup typically uses.
3. Add the following to `configuration.yaml`:

   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```

4. Copy the values from `secrets.example.yaml` into `secrets.yaml`, set the real
   address, and restart Home Assistant.

### Path B (existing config that already defines `modbus:` or `template:`)

Do not add the package for a domain you already define. Home Assistant merges
list-based domains, and both `modbus` and `template` merge, so the real risk is
that the same hub name or the same entity id is produced twice: the second copy
is suffixed `_2` or ignored. Merge the definitions into one place instead of
defining them twice:

- Paste the hub from `packages/ingeteam_modbus.yaml` as another list item of your
  existing `modbus:` block.
- Paste the `- sensor:` block from each template package into your existing
  template configuration.

## What you get

Entities are grouped by kind. The raw register sensors are read over Modbus; the
rest are template sensors derived from them.

### Raw register sensors (54)

| entity_id | unit |
| --- | --- |
| `sensor.ingeteam_stop_event_code` | - |
| `sensor.ingeteam_alarm_code` | - |
| `sensor.ingeteam_status_raw` | - |
| `sensor.ingeteam_waiting_time_to_connect_to_grid` | s |
| `sensor.ingeteam_battery_voltage` | V |
| `sensor.ingeteam_battery_current` | A |
| `sensor.ingeteam_battery_power_raw` | - |
| `sensor.ingeteam_battery_state_of_charge` | % |
| `sensor.ingeteam_battery_state_of_health` | % |
| `sensor.ingeteam_battery_charging_voltage` | V |
| `sensor.ingeteam_battery_discharging_voltage` | V |
| `sensor.ingeteam_battery_max_charging_current` | A |
| `sensor.ingeteam_battery_max_discharging_current` | A |
| `sensor.ingeteam_battery_status_raw` | - |
| `sensor.ingeteam_battery_temp` | °C |
| `sensor.ingeteam_battery_bms_alarm` | - |
| `sensor.ingeteam_battery_discharge_limitation_reason_raw` | - |
| `sensor.ingeteam_battery_voltage_internal_sensor` | V |
| `sensor.ingeteam_pv1_voltage` | V |
| `sensor.ingeteam_pv1_current` | A |
| `sensor.ingeteam_pv1_power` | W |
| `sensor.ingeteam_pv2_voltage` | V |
| `sensor.ingeteam_pv2_current` | A |
| `sensor.ingeteam_pv2_power` | W |
| `sensor.ingeteam_active_power` | W |
| `sensor.ingeteam_reactive_power` | W |
| `sensor.ingeteam_power_factor_cosph` | - |
| `sensor.ingeteam_active_power_reduction_ratio` | % |
| `sensor.ingeteam_ap_reduction_reason_raw` | - |
| `sensor.ingeteam_reactive_power_set_point_type` | - |
| `sensor.ingeteam_critical_loads_voltage` | V |
| `sensor.ingeteam_critical_loads_current` | A |
| `sensor.ingeteam_critical_loads_frequency` | Hz |
| `sensor.ingeteam_critical_loads_active_power` | W |
| `sensor.ingeteam_critical_loads_reactive_power` | var |
| `sensor.ingeteam_internal_meter_voltage` | V |
| `sensor.ingeteam_internal_meter_current` | A |
| `sensor.ingeteam_internal_meter_frequency` | Hz |
| `sensor.ingeteam_internal_active_power` | W |
| `sensor.ingeteam_internal_reactive_power` | var |
| `sensor.ingeteam_internal_power_factor_cosph` | - |
| `sensor.ingeteam_dc_bus_voltage` | V |
| `sensor.ingeteam_temperature_module_1` | °C |
| `sensor.ingeteam_temperature_module_2` | °C |
| `sensor.ingeteam_internal_temperature` | °C |
| `sensor.ingeteam_rms_differential_current` | mA |
| `sensor.ingeteam_do_1_status_raw` | - |
| `sensor.ingeteam_do_2_status_raw` | - |
| `sensor.ingeteam_di_drm_status_raw` | - |
| `sensor.ingeteam_di_2_status_raw` | - |
| `sensor.ingeteam_di_3_status_raw` | - |
| `sensor.ingeteam_total_loads_power` | W |
| `sensor.ingeteam_pv_external_power` | W |
| `sensor.ingeteam_ev_power` | W |

### Text/status sensors (9)

| entity_id | kind |
| --- | --- |
| `sensor.ingeteam_status` | inverter status enum |
| `sensor.ingeteam_battery_status` | battery status enum |
| `sensor.ingeteam_active_power_reduction_reason` | reduction reason enum |
| `sensor.ingeteam_battery_discharge_limitation_reason` | limitation reason enum |
| `sensor.ingeteam_digital_output_1_status` | digital output state |
| `sensor.ingeteam_digital_output_2_status` | digital output state |
| `sensor.ingeteam_digital_input_drm0_status` | digital input state |
| `sensor.ingeteam_digital_input_2_status` | digital input state |
| `sensor.ingeteam_digital_input_3_status` | digital input state |

### Derived power sensors (4)

| entity_id | unit |
| --- | --- |
| `sensor.ingeteam_battery_charging_power` | W |
| `sensor.ingeteam_battery_discharging_power` | W |
| `sensor.ingeteam_pv_internal_total_power` | W |
| `sensor.ingeteam_pv_total_power` | W |

### Optional energy helpers (4)

`packages/ingeteam_modbus_energy.yaml` is optional. It provides the grid
import/export, battery power and PV power entities that an Energy-dashboard setup
typically uses. Load it to create these four entities.

| entity_id | unit |
| --- | --- |
| `sensor.ingeteam_export_power` | W |
| `sensor.ingeteam_import_power` | W |
| `sensor.ingeteam_battery_power` | W |
| `sensor.ingeteam_pv_power` | W |

## Request-rate budget (read this before changing anything)

The native integration issues one Modbus request per entity, so the request rate
is entities divided by `scan_interval`. With 54 entities at the default 15 s that
is about 3.6 requests/s, roughly 18x what the old block-reading component needed,
which overran the inverter and produced stale transaction ids, retries and a
closed connection.

This distribution tiers the entities by how fast their value can meaningfully
move:

- **Tier A, `scan_interval: 30`** - electrical values and direction/status
  registers. A direction register that lags makes the dashboard look
  self-contradictory, so these move fastest.
- **Tier B, `scan_interval: 300`** - slow alarm/enum registers.
- **Tier C, `scan_interval: 1800`** - constants and absent hardware.

Steady-state rate: 36/30 + 4/300 + 14/1800, about 1.22 requests/s, a reduction
of about 66%. `message_wait_milliseconds: 250` gives the device a quiet gap between
requests so a transient stall cannot cascade into a backlog.

For anyone shortening an interval, ~1.5 requests/s is an untested guardrail, not
a measured ceiling. The only verified figures are 3.6 requests/s, which failed,
and the 1.22 requests/s above, which is stable.

- Example of what to move into a faster tier: if you automate on a digital
  output, move `sensor.ingeteam_do_1_status_raw` from tier C to tier A, because a
  30 s delay in that state is otherwise visible in your automation.
- Example of what to leave alone: `sensor.ingeteam_battery_state_of_health`
  changes over weeks; leave it in tier C.

## Known limitations

- **Unverified registers against the manual:** the manual marks PDU 55 and 56
  (`sensor.ingeteam_temperature_module_1`, `sensor.ingeteam_temperature_module_2`)
  as reserved, so they have no manual type and no manual scale. The reference
  installation reads plausible values there, so the entries are kept with the
  type and scale taken from the reference installation, unverified against the
  manual. Delete them if they read as constant or absurd values on your inverter.
- **Firmware-gated registers:** some registers exist only on certain firmware
  versions or hardware variants (external wattmeter, EV charger, newer firmware,
  TL variants). Those reads time out or return garbage on hardware that does not
  implement them. Delete the entries your hardware does not expose; see
  `docs/registers.md` for the gate of each register. Note the pairing: five
  template sensors have a `_raw` modbus source, so deleting
  `sensor.ingeteam_do_1_status_raw`, `sensor.ingeteam_do_2_status_raw`,
  `sensor.ingeteam_di_drm_status_raw`, `sensor.ingeteam_di_2_status_raw` or
  `sensor.ingeteam_di_3_status_raw` leaves the matching text sensor permanently
  unavailable. Delete the matching template sensor too, or add an availability
  guard to it.
- **Read-only scope:** this distribution only reads input registers. There are no
  coils, no holding registers, and no write or control registers.
- **Inherited unit discrepancy:** `sensor.ingeteam_reactive_power` (PDU 38) declares
  `unit_of_measurement: W`, while the manual documents the register as reactive
  power in `var`, as the two sibling reactive-power sensors correctly do. The value
  and scale are correct; only the unit label is wrong. It is kept verbatim so the
  entity does not change unit for users migrating from the custom component. For a
  new installation with no history to preserve, the correct configuration is
  `unit_of_measurement: var` with `device_class: reactive_power`.
- **Derived, not measured:** the four helpers in `packages/ingeteam_modbus_energy.yaml`
  are derived. `sensor.ingeteam_pv_power` in particular is a balance-derived value
  that absorbs the residual the register map does not expose. The offset is
  regime-dependent: the derived value sits above the measured total while
  discharging and below it while charging, and the ~125 W figure is the magnitude
  in the discharging regime. `sensor.ingeteam_pv_total_power` remains the measured
  value; point a display at that entity if the measurement is wanted instead.

## Troubleshooting

| Symptom | Likely cause | Action |
| --- | --- | --- |
| `request ask for transaction_id=N but got id=M, Skipping` | The request rate overran the device. | Raise `scan_interval` or `message_wait_milliseconds`. |
| `No response received ... CLOSING CONNECTION` then `Not connected` | Same rate overrun, or the inverter's TCP server stalled. | Raise `scan_interval` or `message_wait_milliseconds`; the integration reconnects on its own. |
| A specific `address N FAILED` on the I/O, gated (`_P`, `_L`, `_N`) and optional-hardware registers | This hardware variant does not expose that register. | Delete those sensor entries. |
| Entities `unavailable` after restart | Wrong host in `secrets.yaml`, unit id is not 1, or Modbus TCP is disabled in the firewall. | Check all three; enable Modbus in `Comms` -> `Firewall` with the installer account. |
| Values plausible but not matching the inverter's own display | Wrong `data_type`, `swap` or `scale`. | Check the entity against `docs/registers.md`. |

## Migrating from the custom component

This configuration reproduces the `entity_id`s of the `ingeteam_modbus` custom
component for the entities it defines, so in most cases dashboards, statistics
and the Energy-dashboard setup keep working after the switch. Two situations need
a manual step, and the last point covers the component removal itself.

### 1. Entity-id collisions

If a stale registry entry already occupies an id the new configuration wants
(this is what happens when the old component left an orphan such as a duplicate
internal-temperature sensor), Home Assistant suffixes the new entity with `_2`
and any dashboard pointing at the clean id breaks. After the first restart, check
Settings -> Devices & services -> Entities, then rename or delete the stale entry
so the clean id wins, and restart.

### 2. Long-term statistics after a unit change

Six sensors change unit relative to what the old component registered:

| sensor | old unit | new unit |
| --- | --- | --- |
| `sensor.ingeteam_battery_temp` | `C` | `°C` |
| `sensor.ingeteam_temperature_module_1` | `C` | `°C` |
| `sensor.ingeteam_temperature_module_2` | `C` | `°C` |
| `sensor.ingeteam_internal_temperature` | `C` | `°C` |
| `sensor.ingeteam_critical_loads_reactive_power` | `Var` | `var` |
| `sensor.ingeteam_internal_reactive_power` | `Var` | `var` |

Home Assistant's recorder rejects the new unit against the unit already stored in
`statistics_meta` and stops accruing long-term statistics for those series until
the metadata is updated. The supported fix is Home Assistant's Statistics
developer tool (Settings -> Developer tools -> Statistics): it lists the affected
series and lets you correct them. On a large installation the same effect is
achieved by updating `statistics_meta` in the recorder database while Home
Assistant is stopped.

### 3. Duplicate helpers

The four optional energy helpers (`sensor.ingeteam_export_power`,
`sensor.ingeteam_import_power`, `sensor.ingeteam_battery_power`,
`sensor.ingeteam_pv_power`) carry no `unique_id`. If you keep your old definitions
of the same names and also load `packages/ingeteam_modbus_energy.yaml`, Home
Assistant creates `..._2` duplicates while the old entities keep driving the
dashboards. Remove the old definitions of those four before loading the package.

### 4. Removing the old component

If `ingeteam_modbus` is installed, its entities and this configuration would
collide on the same `entity_id`s. Remove the component and its config entry,
restart, and let the new configuration own the ids.

## Credits and license

The register addresses, scales and enum tables come from Ingeteam's published
Modbus documentation (`ABH2010IMB08`, `ABH2010IMC14`). The entity naming and the
enum-to-text mapping follow `vortizhe/home-assistant-ingeteam-modbus`, an
MIT-licensed component (`Copyright (c) 2022 Victor Ortiz`) that this
configuration replaces.

This distribution is MIT licensed (see `LICENSE`, which records the third-party
notices). It is not affiliated with or endorsed by Ingeteam.
