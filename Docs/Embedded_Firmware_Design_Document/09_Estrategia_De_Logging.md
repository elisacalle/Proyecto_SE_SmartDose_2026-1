# Estrategia de Logging

## 1. Descripción General

El sistema SmartDose implementa una estrategia de logging estructurado con el objetivo de permitir trazabilidad y monitoreo en tiempo real del comportamiento del sistema.

El sistema de logs registra eventos relevantes durante la operación, incluyendo:

- Inicio de dispensación
- Reintentos de caída de pastilla
- Fallos de dispensación
- Retiro de pastilla
- Configuración del sistema

El logging está diseñado para integrarse con el monitor serial, la interfaz HMI (Nextion) y los sistemas de alerta (LED y buzzer).

---

## 2. Niveles de Logging

El sistema define tres niveles de severidad:

| Nivel   | Descripción                      |
|---------|----------------------------------|
| `INFO`  | Eventos normales del sistema     |
| `WARN`  | Situaciones especiales no críticas |
| `ERROR` | Fallos críticos del sistema      |

---

## 3. Formato de los Logs

Todos los eventos siguen un formato estructurado:

```
HH:MM:SS | NIVEL [CODIGO] | TUBO | MENSAJE
```

### Ejemplos reales del sistema

```
08:30:12 | ERROR 01 | T1 | Falla de dispensacion
08:30:05 | WARN     | T1 | No se detecto pastilla. Reintento 2/4
08:29:59 | INFO     | T1 | Iniciando dispensacion
```

---

## 4. Implementación en Firmware

El sistema de logging se implementa mediante la función:

```c
smartdose_log(int tubo, const char *nivel, const char *codigo, const char *msg);
```

Esta función:

1. Obtiene la hora desde el RTC (DS3231)
2. Construye el mensaje formateado
3. Lo imprime por UART0
4. Lo almacena en el buffer circular en memoria
5. Lo envía a la pantalla Nextion

---

## 5. Almacenamiento de Logs

Los logs se almacenan en un buffer circular en memoria RAM:

```c
#define MAX_LOGS 5
static char g_logs[MAX_LOGS][LOG_LEN];
```

Características:

- Se almacenan los **últimos 5 eventos**
- Los logs más antiguos se sobrescriben 
- Se prioriza eficiencia en memoria, acorde a las limitaciones del sistema embebido

---

## 6. Canales de Salida

### 6.1 Monitor Serial (UART0)

Los logs se imprimen usando:

```c
printf("%s\n", linea);
```

Uso principal: pruebas y validación del sistema.

### 6.2 Interfaz HMI (Nextion)

Los logs se muestran en pantalla mediante:

```c
nextion_actualizar_logs();
```

Distribución en pantalla:

| Componente | Función          |
|------------|------------------|
| `tLog1`    | Log más reciente |
| `tLog5`    | Log más antiguo  |

### 6.3 Alertas físicas (LED + Buzzer)

Los eventos críticos activan señales físicas (LED encendido, buzzer activo), controladas mediante:

```c
actualizar_salidas_alarma();
```

---

## 7. Eventos Registrados

### 7.1 Eventos INFO

| Evento                 | Descripción                    |
|------------------------|--------------------------------|
| Inicio de dispensación | Se inicia un ciclo de dispensa |
| Horario guardado       | Configuración exitosa en NVS   |
| Pastilla retirada      | Fin del ciclo de dispensación  |

### 7.2 Eventos WARN

| Evento                    | Descripción                     |
|---------------------------|---------------------------------|
| Reintento de dispensación | No se detectó caída de pastilla |
| Pastilla no recogida      | Timeout de espera al usuario    |

### 7.3 Eventos ERROR

| Código     | Descripción                                    |
|------------|------------------------------------------------|
| `ERROR 01` | Falla de dispensación tras 4 intentos fallidos |

Implementación:

```c
static void log_error_01(int tubo)
{
    smartdose_log(tubo, "ERROR", "01",
        "Falla de dispensacion: no se detecto dosis; "
        "posible atasco, tubo vacio o falla del motor");
}
```

---

## 8. Integración con el Sistema

El logging está directamente conectado con la lógica de dispensación. Flujo típico de un ciclo:

```
Inicio de ciclo     →  INFO   (Iniciando dispensacion)
Reintento N/4       →  WARN   (No se detecto pastilla)
Fallo definitivo    →  ERROR  (ERROR 01) + activación de alarma
Espera de retiro    →  WARN   (si se supera TIMEOUT_RECOGIDA_MS)
Pastilla retirada   →  INFO   (Pastilla retirada)
```

---

## 9. Retroalimentación en Tiempo Real

Cada log también actualiza el mensaje principal de la pantalla Nextion:

```c
nextion_set_txt("tMsg", resumen);
```

Esto permite que el usuario tenga visibilidad inmediata del estado del dispositivo sin necesidad de un PC conectado.

---

## 10. Decisiones de Diseño

### Buffer de 5 logs

Se eligió este tamaño por:

- Limitaciones de memoria RAM del ESP32 en contexto multitarea
- Suficiente para diagnóstico inmediato de un ciclo completo
- Compatible con el espacio disponible en la interfaz Nextion

### Uso de niveles (INFO / WARN / ERROR)

Permite:

- Clasificación clara de eventos por severidad
- Escalabilidad futura (posible filtrado por nivel)
- Mejor experiencia de depuración durante desarrollo

### Integración con HMI

Permite:

- Monitoreo del sistema sin necesidad de PC
- Diagnóstico visual inmediato para el usuario final

### Uso de RTC para timestamps

Garantiza:

- Trazabilidad real de eventos referenciada al tiempo absoluto
- Independencia del uptime del sistema ante reinicios
