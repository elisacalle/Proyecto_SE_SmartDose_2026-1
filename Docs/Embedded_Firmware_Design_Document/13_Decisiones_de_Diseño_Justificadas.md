# Decisiones de diseño justificadas - SmartDose

## 1. Descripción general del proyecto

SmartDose es un prototipo de dispensador automático de pastillas basado en un ESP32, una pantalla Nextion NX4832K035, un módulo de reloj DS3231, servomotores, sensor infrarrojo de bandeja, buzzer y LED de alerta.

El sistema permite configurar horarios de dispensación para cuatro tubos independientes. Para cada tubo se define una hora, un minuto y una cantidad de activaciones del mecanismo. Cuando llega la hora programada, el ESP32 acciona el servomotor correspondiente, verifica si la dosis llegó a la bandeja y muestra el resultado en la interfaz gráfica.

## 2. Arquitectura general

La arquitectura se organiza en cuatro bloques:

| Bloque | Componente | Función |
|---|---|---|
| Control central | ESP32 | Ejecuta la lógica, controla servos, lee sensores y actualiza la interfaz |
| Interfaz de usuario | Nextion NX4832K035 | Permite configurar horarios, visualizar alertas y consultar logs |
| Tiempo real | RTC DS3231 | Mantiene la hora real del sistema |
| Dispensación | Servomotores + sensor IR de bandeja | Ejecutan y validan la dispensación |

Esta distribución permite que la pantalla no controle directamente el proceso crítico, la Nextion recibe y envía información, pero el ESP32 decide cuándo dispensar, cuándo activar alarmas y cómo registrar los eventos.

## 3. Elección del ESP32

Se seleccionó el ESP32 porque ofrece suficientes pines GPIO, comunicación UART, comunicación I2C y soporte de multitarea mediante FreeRTOS.

Esta decisión permite:

- Controlar cuatro servomotores de forma independiente.
- Leer sensores digitales de bandeja.
- Comunicarse con la Nextion mediante UART.
- Comunicarse con el DS3231 mediante I2C.
- Guardar horarios en memoria no volátil.
- Ejecutar tareas separadas para dispensación, monitoreo, comunicación y actualización de pantalla.

La multitarea es útil porque el sistema debe seguir recibiendo comandos y actualizando la interfaz mientras una dispensación está en curso.

## 4. Elección de la pantalla Nextion

Se incorporó una pantalla Nextion NX4832K035 para mejorar la interacción con el usuario. Esta pantalla permite diseñar una interfaz táctil con páginas, botones, textos y mensajes de alerta.

La Nextion permite:

- Mostrar la hora actual del RTC.
- Configurar horarios y cantidades de dispensación.
- Mostrar alertas de dosis lista.
- Mostrar errores de dispensación.
- Visualizar logs del sistema.

La pantalla se comporta como una HMI. El ESP32 conserva el control lógico del proceso y la Nextion solo actúa como interfaz de entrada y salida.

## 5. Comunicación ESP32–Nextion

La comunicación entre ESP32 y Nextion se realiza mediante UART1 a 9600 baudios.

Configuración usada:

```c
#define UART_NUM_NEXTION UART_NUM_1
#define NEXTION_TX_GPIO  5   // ESP32 TX -> RX Nextion
#define NEXTION_RX_GPIO  4   // ESP32 RX <- TX Nextion
#define NEXTION_BAUD     9600
```

Conexión física:

| Nextion | ESP32 |
|---|---|
| TX | GPIO4 |
| RX | GPIO5 |
| 5V | 5V |
| GND | GND común |


## 6. Uso del RTC DS3231

El sistema usa un RTC DS3231 para mantener una hora confiable. Esto es necesario porque SmartDose depende directamente de horarios programados.

La pantalla Nextion no asigna la hora del sistema. La hora se configura desde el ESP32 y se conserva en el DS3231. La Nextion únicamente muestra la hora que el ESP32 lee del RTC.

Esta decisión evita inconsistencias y mantiene una única fuente de verdad para el tiempo del sistema.


## 7. Retiro del sensor superior de almacenamiento

Inicialmente se planteó usar dos sensores por tubo:

1. Sensor superior para detectar si había pastillas almacenadas.
2. Sensor inferior para detectar si una pastilla llegó a la bandeja.

Durante el desarrollo se decidió retirar el sensor superior porque su detección lineal no era adecuada para inferir almacenamiento. Podía generar falsos estados dependiendo de la posición de las pastillas dentro del tubo.
Los posibles problemas del sensor superior eran indicar vacío aunque aún hubiera pastillas, indicar presencia aunque el compartimiento estuviera casi vacío, depender demasiado de la geometría interna del tubo, fallar si ninguna pastilla interrumpía exactamente el haz del sensor.
Por esto, desde la parte de Firmware se decidió que se intentara la dispensación en el horario programado con 4 intentos antes de marcar un error, este error puede estar relacionado a falta de suficientes pastillas o ubicación de las mismas.

Por eso se decidió validar la funcionalidad con el sensor de bandeja, que confirma el evento más importante: si la dosis llegó realmente al punto de retiro.


## 8. Uso del sensor de bandeja

El sistema final usa únicamente el sensor IR inferior ubicado en la bandeja de dispensación.

Este sensor tiene dos funciones:
1. Confirmar si una dosis fue dispensada.
2. Detectar cuándo el usuario retiró la dosis.

La lógica queda así:

```text
Llega la hora programada
→ Se activa el servomotor
→ El sistema espera detección en bandeja
→ Si hay detección, se muestra dosis lista
→ Si no hay detección, se reintenta la dispensación (4 intentos)
```

Esta solución reduce complejidad de hardware y permite validar el resultado real de la dispensación.

## 9. Reintentos de dispensación

Se implementó una lógica de reintentos. Si después de activar el servomotor no se detecta pastilla en la bandeja durante una ventana de 3 segundos, el sistema vuelve a accionar el motor.

El número máximo de intentos es:

```c
#define MAX_REINTENTOS_CAIDA 4
```

Esta decisión permite manejar situaciones como fricción, retrasos mecánicos o una pastilla parcialmente trabada. Si después de cuatro intentos no hay detección, el sistema declara un error.


## 10. Error 01

El sistema define un error crítico llamado Error 01.

Este error ocurre cuando, después de cuatro intentos de dispensación, el sensor de bandeja no detecta ninguna dosis.

El error se formula como un error general porque con un solo sensor no es posible saber con certeza si la causa fue ausencia de pastillas, atasco o falla del motor.

Causas posibles:

- Tubo vacío.
- Atasco mecánico.
- Falla del servomotor.
- Pastilla bloqueada.
- Problema en el mecanismo de salida.


## 11. Manejo de alertas

El sistema usa un LED Rojo y un buzzer.


### Dosis dispensada correctamente

Cuando el sensor de bandeja detecta una pastilla:

- Se muestra la página de dosis lista.
- Se enciende el LED.
- Se activa el buzzer.
- Ambos permanecen activos mientras la pastilla siga en la bandeja.

Cuando la persona retira la pastilla:

- El sensor deja de detectar.
- Se apagan LED y buzzer.
- La pantalla vuelve a la página principal.
- Se registra el evento en logs.

### Error de dispensación

Cuando ocurre el Error 01:

- Se muestra la página de error.
- Se encienden LED y buzzer.
- Se registra el error en logs.
- La alarma se mantiene durante 15 segundos y luego se apaga.

Se eligió limitar la alarma de error para evitar que el buzzer quede activo indefinidamente.

## 12. Interfaz gráfica

La interfaz Nextion se organiza en páginas.

### page0 — Pantalla principal

Muestra:
- Hora actual.
- Tubos.
- Horario programado de cada tubo.
- Mensaje general.
- Acceso a configuración, limpieza y logs.

### page1 — Configuración

Permite configurar:
- Hora.
- Minuto.
- Cantidad de dispensaciones.

Cada botón envía comandos como:

```text
T1=HH:MM:C
```

### page2 — Dosis lista

Se muestra cuando una dosis llega correctamente a la bandeja.

### page3 — Error de dispensación

Se muestra cuando ocurre Error 01.

### page4 — Logs

Muestra los últimos eventos del sistema.

## 13. Sistema de logs

El sistema registra eventos con niveles de severidad.

| Nivel | Uso |
|---|---|
| INFO | Eventos normales |
| WARN | Advertencias que requieren atención |
| ERROR | Fallas críticas |

Ejemplo de registros:

```text
15:42:01 | INFO | T1 | Horario guardado T1 15:45 x1
15:45:00 | INFO | T1 | Iniciando dispensación
15:45:03 | WARN | T1 | Dosis en bandeja, esperando retiro
15:45:12 | INFO | T1 | Pastilla retirada
15:50:08 | ERROR 01 | T2 | Falla general de dispensación
```

La pantalla muestra los últimos cinco logs, con el evento más reciente en la parte superior.

## 14. Seguridad y validación

El prototipo no debe probarse con medicamentos reales. Para pruebas se recomienda usar elementos simulados, como Tic Tac, dulces pequeños o piezas impresas.

El sistema valida funcionalidad, pero no corresponde a un dispositivo médico certificado. La validación debe centrarse en:

- Comunicación con pantalla.
- Configuración de horarios.
- Activación de servomotores.
- Detección de bandeja.
- Manejo de error.
- Registro de logs.


## 15. Conclusión

SmartDose integra control embebido, interfaz gráfica, temporización en tiempo real y validación por sensor. Las decisiones de diseño tomadas permiten mantener un prototipo funcional, modular y justificable.

La decisión más importante fue retirar el sensor superior y validar la dispensación con el sensor de bandeja, ya que esto prioriza el evento real de interés: que la dosis llegue al usuario. La integración con Nextion mejora la usabilidad al permitir configurar horarios, visualizar alertas y consultar logs sin depender de la terminal de desarrollo.
