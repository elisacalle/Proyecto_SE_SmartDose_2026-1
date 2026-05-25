# SmartDose — Arquitectura de Firmware

Sistema automático de dispensación de medicamentos basado en **ESP32** con **FreeRTOS**, desarrollado con ESP-IDF y PlatformIO.

---

## Tabla de contenidos

1. [Visión general](#1-visión-general)
2. [Hardware y pines](#2-hardware-y-pines)
3. [Estructura del firmware](#3-estructura-del-firmware)
4. [Tareas FreeRTOS](#4-tareas-freertos)
5. [Módulos de software](#5-módulos-de-software)
6. [Integración ESP32–Nextion](#6-integración-esp32nextion)
7. [Máquina de estados por tubo](#7-máquina-de-estados-por-tubo)
8. [Flujo de dispensación](#8-flujo-de-dispensación)
9. [Protocolo de comandos](#9-protocolo-de-comandos)
10. [Persistencia — NVS](#10-persistencia--nvs)
11. [Parámetros configurables](#11-parámetros-configurables)

---

## 1. Visión general

<img width="512" height="164" alt="image" src="https://github.com/user-attachments/assets/7504e5a8-7ee9-40ea-922f-c6e962006526" />


SmartDose controla hasta **4 tubos dispensadores** de forma independiente. La **Nextion** actúa exclusivamente como interfaz gráfica: recibe toques del usuario y muestra el estado del sistema. El **ESP32** centraliza toda la lógica: valida comandos, guarda horarios, ejecuta dispensaciones y actualiza la pantalla. La Nextion no controla motores ni sensores directamente.

---

## 2. Hardware y pines

| Componente | GPIO | Interfaz |
|---|---|---|
| Servo Tubo 1 | 14 | LEDC / PWM |
| Servo Tubo 2 | 27 | LEDC / PWM |
| Servo Tubo 3 | 16 | LEDC / PWM |
| Servo Tubo 4 | 17 | LEDC / PWM |
| IR Bandeja Tubo 1 | 36 | GPIO entrada |
| IR Bandeja Tubo 2 | 39 | GPIO entrada |
| IR Bandeja Tubo 3 | 25 | GPIO entrada |
| IR Bandeja Tubo 4 | 26 | GPIO entrada |
| Buzzer | 13 | GPIO salida |
| LED de alerta | 12 | GPIO salida |
| I2C SDA (DS3231) | 21 | I2C Master |
| I2C SCL (DS3231) | 22 | I2C Master |
| UART0 TX/RX | — | Consola / depuración (115200 bps) |
| UART1 TX → Nextion | 5 | UART1 (9600 bps) |
| UART1 RX ← Nextion | 4 | UART1 (9600 bps) |

| Periférico | Driver ESP-IDF | Uso |
|---|---|---|
| Servos x4 | `driver/ledc` | PWM 50 Hz, 14 bits |
| Sensores IR x4 | `driver/gpio` | Entrada digital con pull-up |
| Buzzer + LED | `driver/gpio` | Salida digital |
| DS3231 | `driver/i2c` | Lectura/escritura de hora |
| Consola | `driver/uart` UART0 | Comandos de depuración |
| Nextion HMI | `driver/uart` UART1 | Interfaz táctil |
| Persistencia | `nvs_flash` / `nvs` | Horarios y flag RTC |

---

## 3. Estructura del firmware

Todo el firmware reside en `main.c`, organizado en secciones funcionales:

```
main.c
│
├── Definiciones de pines y parámetros (#define)
├── Estructuras de datos (horario_t, info_tubo_t, estado_tubo_t)
├── Variables globales + mutex FreeRTOS + buffer de logs
│
├── [Módulo] Servo         — angulo_a_duty(), servo_set(), servo_dispensar()
├── [Módulo] Sensor IR     — sensor_leer() con debounce, bandeja_tiene_pastilla()
├── [Módulo] Alertas       — buzzer_set(), led_set(), actualizar_salidas_alarma()
├── [Módulo] RTC DS3231    — ds3231_leer(), ds3231_get_hora(), ds3231_set_hora()
├── [Módulo] Nextion HMI   — nextion_cmd(), nextion_set_txt(), nextion_set_val()
│                            nextion_actualizar_pantalla(), nextion_actualizar_logs()
│                            nextion_actualizar_campos_config(), nextion_actualizar_horarios_page0()
│                            nextion_limpiar_campos_config()
├── [Módulo] NVS           — nvs_guardar(), nvs_cargar(), nvs_borrar_horarios()
│                            rtc_ya_configurado(), marcar_rtc_configurado()
├── [Módulo] Log           — smartdose_log(), log_evento(), log_error_01()
├── [Módulo] Dispensación  — ciclo_dispensacion(), solicitar_dispensacion()
│                            dispensar_con_reintentos_y_confirmar_bandeja()
├── [Módulo] Comandos      — procesar_comando(), configurar_horario()
│                            ack_tubo(), borrar_todos_los_horarios()
│
└── Tareas FreeRTOS
    ├── tarea_scheduler
    ├── tarea_monitor
    ├── tarea_uart_monitor
    ├── tarea_nextion_rx
    ├── tarea_nextion_update
    └── tarea_dispensacion (dinámica por tubo)
```

### Estructuras de datos

```c
typedef enum { TUBO_OK, TUBO_DISPENSANDO, TUBO_FALLO } estado_tubo_t;

typedef struct {
    bool activo;
    int  hora, minuto, cantidad;  // cantidad: 1–10 pastillas
} horario_t;

typedef struct {
    estado_tubo_t estado;
    bool          pastilla_en_bandeja;
    bool          alerta_activa;
} info_tubo_t;
```

Todas las lecturas/escrituras sobre `g_horarios[]` y `g_info[]` están protegidas con un **mutex FreeRTOS** (`g_mutex`). Los últimos 5 eventos del sistema se mantienen en `g_logs[5][120]`.

---

## 4. Tareas FreeRTOS

| Tarea | Prioridad | Stack | Período | Función |
|---|---|---|---|---|
| `tarea_scheduler` | 6 | 4 KB | 1 s | Compara hora RTC con horarios; lanza dispensación automática |
| `tarea_nextion_rx` | 5 | 4 KB | Reactiva | Lee comandos enviados por la Nextion vía UART1 |
| `tarea_monitor` | 4 | 4 KB | 7 s | Verifica bandejas y actualiza alertas |
| `tarea_uart_monitor` | 3 | 4 KB | Reactiva | Lee comandos desde consola UART0 |
| `tarea_nextion_update` | 2 | 4 KB | 1.5 s | Refresca la pantalla Nextion periódicamente |
| `tarea_dispensacion` | 7 | 4 KB | Bajo demanda | Creada por tubo; se auto-elimina al terminar |

> `tarea_dispensacion` se crea con `xTaskCreate()` por horario o comando manual, y se elimina con `vTaskDelete(NULL)` al completar el ciclo.

---

## 5. Módulos de software

### Servo (LEDC PWM)
- **50 Hz**, resolución **14 bits**, pulso **600–2500 µs**.
- Apertura: **180°** → Cierre: **0°**, con `SERVO_MS_ACCION = 400 ms` entre posiciones.
- Conversión: `us = 600 + (2500-600) * angulo / 180` → `duty = (us * 16383) / 20000`

### Sensor IR de bandeja (FC-51)
- Anti-rebote por software: **3 muestras** con 10 ms de separación; mayoría decide.
- `SENSOR_ACTIVO_EN_LOW = 1` → LOW = objeto presente.

### RTC DS3231
- I²C a **400 kHz**, dirección `0x68`. Conversión BCD ↔ decimal implementada manualmente.
- Si no fue configurado previamente (flag en NVS), se inicializa a hora por defecto al arrancar.

### Alertas
- `actualizar_salidas_alarma()` enciende LED y buzzer si **cualquier** tubo tiene `alerta_activa = true` o `pastilla_en_bandeja = true`.

### Sistema de logs
- Formato: `HH:MM:SS | NIVEL [CODIGO] | T<n>/SYS | Mensaje`
- Buffer circular de 5 entradas en `g_logs[]`. El más reciente queda en `g_logs[0]`.
- Cada log se escribe por `printf`, se guarda en el buffer y se envía a la Nextion.

### NVS
- Namespace `"smartdose"`, claves: `"horarios"` (string) y `"rtc_set"` (uint8).
- `nvs_borrar_horarios()` escribe `"VACIO"` explícitamente para evitar datos corruptos tras reinicios inesperados.

---

## 6. Integración ESP32–Nextion

### Comunicación física

```
UART0 → Consola VSCode (115200 bps) — depuración y comandos manuales
UART1 → Nextion HMI  (9600 bps)    — interfaz táctil bidireccional

Nextion TX → GPIO4 (ESP32 RX)
Nextion RX → GPIO5 (ESP32 TX)
```

TX y RX deben estar cruzados. GND común entre todos los módulos.

### Protocolo de comunicación

**Nextion → ESP32:** cadenas ASCII terminadas en `\r\n` o `0xFF 0xFF 0xFF`.

Ejemplo de evento táctil en Nextion (botón Guardar T1):
```
prints "T1=",0
prints nT1H.val,0
prints ":",0
prints nT1M.val,0
prints ":",0
prints nT1C.val,0
printh 0D 0A
```

**ESP32 → Nextion:** comandos ASCII terminados siempre en `0xFF 0xFF 0xFF`.

```c
// Toda escritura pasa por nextion_write_raw(), que agrega 0xFF 0xFF 0xFF
nextion_set_txt("tHora", "14:23:08");   // → tHora.txt="14:23:08" + FF FF FF
nextion_set_val("jAlerta", 100);        // → jAlerta.val=100 + FF FF FF
nextion_cmd("page page2");              // → page page2 + FF FF FF
```

### Funciones de envío (ESP32 → Nextion)

| Función | Uso |
|---|---|
| `nextion_write_raw(s)` | Envía string crudo + terminador `0xFF 0xFF 0xFF` |
| `nextion_cmd(fmt, ...)` | Construye comando con formato y llama a `write_raw` |
| `nextion_set_txt(obj, txt)` | Actualiza texto de un componente. Filtra comillas dobles |
| `nextion_set_val(obj, val)` | Actualiza valor numérico o barra de progreso |

### Funciones de sincronización de pantalla

| Función | Qué actualiza |
|---|---|
| `nextion_actualizar_pantalla()` | Hora RTC, horarios `page0`, `tMsg`, `jAlerta`, logs |
| `nextion_actualizar_horarios_page0()` | `tT1Hor`…`tT4Hor` con horario activo o `"Sin horario"` |
| `nextion_actualizar_campos_config()` | `nT1H`, `nT1M`, `nT1C`…`nT4C` con valores reales de RAM |
| `nextion_limpiar_campos_config()` | Resetea campos numéricos a 0/0/1 |
| `nextion_actualizar_logs()` | `tLog1`…`tLog5` con el buffer circular `g_logs[]` |

> **Fuente de verdad:** el ESP32, no la Nextion. Los valores de la pantalla solo son válidos después de que el ESP32 los recibe, valida y guarda en memoria.

### Páginas de la Nextion

| Página | Cuándo se muestra | Contenido principal |
|---|---|---|
| `page0` | Funcionamiento normal | Hora, horarios activos, mensaje general |
| `page1` | Configuración de horarios | Campos numéricos hora/minuto/cantidad por tubo |
| `page2` | Dosis en bandeja | Alerta de retiro de pastilla |
| `page3` | Error de dispensación | Mensaje ERROR 01 y descripción de fallo |
| `page4` | Historial | Últimos 5 eventos (`tLog1`…`tLog5`) |

### Recepción de comandos (tarea_nextion_rx)

<img width="512" height="315" alt="image" src="https://github.com/user-attachments/assets/1d0fdd0d-9b2c-41e0-b873-3b5339037264" />


`procesar_comando()` es compartida entre UART0 y UART1. El prefijo `nxt:` enviado desde Nextion se elimina automáticamente antes de interpretar el comando.

---

## 7. Máquina de estados por tubo

<img width="512" height="280" alt="image" src="https://github.com/user-attachments/assets/f281a282-d3e1-4c46-81cd-69027a1aded1" />


---

## 8. Flujo de dispensación

```
solicitar_dispensacion(idx)
        │
        ├── Tubo en TUBO_DISPENSANDO → rechazar
        └── xTaskCreate(tarea_dispensacion)
                    │
                    ▼
            ciclo_dispensacion(idx)
                    │
                    ├── [1] Estado → TUBO_DISPENSANDO
                    │
                    ├── [2] Hasta 4 reintentos:
                    │       servo: 180° → 400ms → 0° → 400ms
                    │       esperar sensor bandeja (máx 3 s)
                    │       ├── Detectado → Éxito
                    │       └── Timeout → WARN log → siguiente intento
                    │
                    ├── [3a] ÉXITO → TUBO_OK, alerta ON, page2
                    │       Esperar retiro (confirmación doble 300 ms)
                    │       └── Bandeja vacía → alarma OFF, page0, log INFO
                    │
                    └── [3b] FALLO → TUBO_FALLO, alerta ON, page3, log ERROR 01
                              15 s → alarma OFF, TUBO_FALLO persiste hasta ACK
```

---

## 9. Protocolo de comandos

Aceptados desde **UART0** (consola) y **UART1** (Nextion). No distinguen mayúsculas/minúsculas.

| Comando | Descripción | Ejemplo |
|---|---|---|
| `T<n>=HH:MM:C` | Configurar horario de un tubo | `T1=08:30:1` |
| `CFG,tubo,hora,min,cantidad` | Formato alternativo | `CFG,1,8,30,1` |
| `DISP<n>` | Dispensación manual | `DISP2` |
| `ACK<n>` | Confirmar y limpiar alerta | `ACK1` |
| `SETTIME=HH:MM:SS` | Sincronizar RTC | `SETTIME=09:15:00` |
| `STATUS` / `REFRESH` | Estado en consola + refresco Nextion | `STATUS` |
| `CLEAR` / `BORRAR` | Borrar todos los horarios | `CLEAR` |

**Validaciones:** Tubo 1–4 · Hora 0–23 · Minuto 0–59 · Cantidad 1–10

---

## 10. Persistencia — NVS

```
NVS namespace: "smartdose"
│
├── "horarios" → "T1=08:30:1;T3=20:00:2"   (string, máx 256 bytes)
└── "rtc_set"  → 0x01                        (uint8, flag de configuración)
```

- Los horarios se serializan como texto separado por `;`. Si no hay horarios, se almacena `"VACIO"`.
- Al arrancar, `nvs_cargar()` parsea el string y reconstruye `g_horarios[]`.
- `nvs_borrar_horarios()` escribe `"VACIO"` explícitamente para evitar datos residuales tras reinicios inesperados.

---

## 11. Parámetros configurables

| Parámetro | Valor | Descripción |
|---|---|---|
| `MAX_TUBOS` | 4 | Número de tubos dispensadores |
| `SERVO_FREQ_HZ` | 50 | Frecuencia PWM |
| `SERVO_MIN_US` | 600 | Pulso mínimo servo (µs) |
| `SERVO_MAX_US` | 2500 | Pulso máximo servo (µs) |
| `SERVO_ANGULO_ABIERTO` | 180° | Ángulo de apertura |
| `SERVO_ANGULO_CERRADO` | 0° | Ángulo de cierre |
| `SERVO_MS_ACCION` | 400 ms | Tiempo entre posiciones del servo |
| `TIMEOUT_CAIDA_MS` | 3000 ms | Espera máxima para detectar pastilla en bandeja |
| `TIMEOUT_RECOGIDA_MS` | 60000 ms | Espera máxima antes de re-alertar pastilla no retirada |
| `ERROR_ALARMA_MS` | 15000 ms | Duración de alarma en fallo de dispensación |
| `MAX_REINTENTOS_CAIDA` | 4 | Intentos antes de declarar ERROR 01 |
| `DEBOUNCE_COUNT` | 3 | Muestras para anti-rebote del sensor IR |
| `TICK_SCHEDULER_MS` | 1000 ms | Período del scheduler |
| `TICK_MONITOR_MS` | 7000 ms | Período de monitoreo de bandejas |
| `TICK_NEXTION_MS` | 1500 ms | Período de refresco de pantalla |
| `MAX_LOGS` | 5 | Entradas del buffer circular de logs |
| `NEXTION_BAUD` | 9600 | Baudrate UART1 Nextion |
| `SENSOR_ACTIVO_EN_LOW` | 1 | Lógica del sensor IR (1 = LOW activo) |
