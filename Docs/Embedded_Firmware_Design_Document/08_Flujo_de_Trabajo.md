# SmartDose — Estructura del Repositorio y Flujo de Trabajo

---

## Tabla de contenidos

1. [Estructura de directorios](#1-estructura-de-directorios)
2. [Descripción de carpetas](#2-descripción-de-carpetas)
   - [Raíz del repositorio](#raíz-del-repositorio)
   - [Docs/](#docs)
   - [Firmware/](#firmware)
   - [Hardware/](#hardware)
   - [Images/](#images)
3. [Flujo de trabajo del repositorio](#3-flujo-de-trabajo-del-repositorio)

---

## 1. Estructura de directorios

```
SmartDose/
│
├── Docs/
│   └── Embedded_firmware_design_document/
│       ├── Introduccion
│       ├── Objetivos_proyecto
│       ├── Hardware_Architecture
│       ├── Diagrama_de_bloques
│       ├── Arquitectura_firmware
│       ├── Diagrama_de_estados
│       ├── Estrategia_de_manejo_de_errores
│       ├── Flujo_de_trabajo
│       ├── Estrategia_de_logging
│       ├── Gestion_de_comunicaciones
│       ├── Matriz_de_trazabilidad_de_requerimientos
│       ├── Casos_de_prueba
│       └── Decisiones_de_diseno_justificadas
│
├── Firmware/
│   ├── include/
│   ├── lib/
│   │   └── src/
│   │       └── main/
│   │           └── main.c
│   ├── test/
│   ├── .gitignore
│   ├── CMakeLists.txt
│   ├── platformio.ini
│   └── sdkconfig.az-delivery-devkit-v4
│
├── Hardware/
│   ├── Interfaz_grafica/
│   ├── Dispensador.step
│   ├── Esquematico_SmartDose.md
│   └── Prototipo_funcional/
│
├── Imágenes/
│   └── (diseño físico del sistema)
│
└── README.md
```

---

## 2. Descripción de carpetas

### Raíz del repositorio

| Elemento | Descripción |
|---|---|
| `README.md` | Punto de entrada del repositorio. Contiene la descripción general del proyecto, problema que resuelve, objetivos, tecnologías usadas, instrucciones de instalación y referencias a las demás secciones. |
| `Docs/` | Documentación técnica completa del sistema. |
| `Firmware/` | Todo el código fuente del ESP32 y archivos de configuración del entorno de desarrollo. |
| `Hardware/` | Archivos de diseño físico, esquemático e interfaz gráfica. |
| `Imágenes/` | Registro fotográfico del prototipo físico construido. |

---

### Docs/

Contiene el **Embedded Firmware Design Document**, documento técnico que cubre la totalidad del diseño del sistema desde sus objetivos hasta sus casos de prueba.

| Sección | Contenido |
|---|---|
| `Introduccion` | Contexto del proyecto, problema que resuelve y alcance del sistema. |
| `Objetivos_proyecto` | Objetivos generales y específicos del sistema SmartDose. |
| `Hardware_Architecture` | Descripción de componentes físicos, pines, interfaces y periféricos del ESP32. |
| `Diagrama_de_bloques` | Representación visual de la interacción entre subsistemas (ESP32, servos, sensores, RTC, Nextion, NVS). |
| `Arquitectura_firmware` | Estructura del código, módulos de software, tareas FreeRTOS y estructuras de datos. |
| `Diagrama_de_estados` | Máquina de estados por tubo: `TUBO_OK`, `TUBO_DISPENSANDO`, `TUBO_FALLO` y sus transiciones. |
| `Estrategia_de_manejo_de_errores` | Catálogo de errores detectados, flujo de manejo automático, alarmas y acciones del operador. |
| `Flujo_de_trabajo` | Estructura del repositorio y flujo de desarrollo del proyecto. |
| `Estrategia_de_logging` | Formato del sistema de logs en memoria, niveles de severidad y visualización en Nextion. |
| `Gestion_de_comunicaciones` | Protocolo UART0 (consola), UART1 (Nextion) e I²C (DS3231): tramas, baudrates y parsing. |
| `Matriz_de_trazabilidad_de_requerimientos` | Relación entre requerimientos del sistema y su implementación en el firmware. |
| `Casos_de_prueba` | Pruebas funcionales definidas para validar el comportamiento del sistema. |
| `Decisiones_de_diseno_justificadas` | Registro de decisiones técnicas tomadas durante el desarrollo y su justificación. |

---

### Firmware/

Contiene el proyecto completo de firmware desarrollado con **ESP-IDF** y gestionado con **PlatformIO**.

| Elemento | Descripción |
|---|---|
| `include/` | Archivos de cabecera (`.h`) del proyecto. |
| `lib/src/main/main.c` | Archivo principal del firmware. Contiene la totalidad de la lógica del sistema: inicialización de periféricos, tareas FreeRTOS, módulos de servo, sensor IR, RTC, Nextion, NVS, logs y manejo de comandos. |
| `test/` | Carpeta reservada para pruebas unitarias del firmware. |
| `.gitignore` | Define los archivos y carpetas excluidos del control de versiones (`.pio/`, `build/`, archivos temporales). |
| `CMakeLists.txt` | Archivo de configuración del sistema de build de ESP-IDF. Define los componentes y fuentes del proyecto. |
| `platformio.ini` | Archivo de configuración de PlatformIO. Define la plataforma (`espressif32`), el board (`az-delivery-devkit-v4`), el framework (`espidf`) y opciones de compilación. |
| `sdkconfig.az-delivery-devkit-v4` | Configuración del SDK de ESP-IDF específica para el módulo AZ-Delivery DevKit V4 (ESP32). Contiene parámetros de particiones, frecuencia de CPU, flash y más. |

---

### Hardware/

Contiene los archivos de diseño físico y electrónico del sistema.

| Elemento | Descripción |
|---|---|
| `Interfaz_grafica/` | Archivos del proyecto Nextion HMI (`.HMI`): páginas, componentes, eventos touch y lógica de la pantalla táctil. |
| `Dispensador.step` | Modelo 3D del mecanismo dispensador en formato STEP, compatible con Fusion 360, SolidWorks y otros CAD. |
| `Esquematico_SmartDose.md` | Esquemático del circuito electrónico: conexiones entre ESP32, servos, sensores IR, DS3231, buzzer, LED y Nextion. |
| `Prototipo_funcional/` | Fotografías y/o archivos de referencia del prototipo físico ensamblado y funcionando. |

---

### Imágenes/

| Elemento | Descripción |
|---|---|
| Fotografías | Registro visual del diseño físico del sistema SmartDose ensamblado. Usadas como referencia en el `README.md` y la documentación técnica. |

---

## 3. Flujo de trabajo del repositorio

<img width="512" height="280" alt="image" src="https://github.com/user-attachments/assets/814e04e2-3dc0-48dc-8c2c-f4c85679c939" />


### Flujo detallado

**1. Configuración del entorno**

- Instalar [PlatformIO](https://platformio.org/) como extensión de VS Code.
- Clonar el repositorio y abrir la carpeta `Firmware/` como proyecto PlatformIO.
- El archivo `platformio.ini` configura automáticamente el framework ESP-IDF y el board correcto (`az-delivery-devkit-v4`).

**2. Desarrollo de firmware**

- Todo el código del sistema reside en `Firmware/lib/src/main/main.c`.
- Los parámetros de comportamiento (timeouts, pines, ángulos, períodos) se modifican únicamente en la sección de `#define` al inicio del archivo.
- Compilar con el comando de PlatformIO: `Build` o `pio run`.

**3. Programación del dispositivo**

- Conectar el ESP32 por USB.
- Usar `Upload` en PlatformIO (`pio run --target upload`) para flashear el firmware.
- Verificar el arranque correcto con el monitor serial a **115200 bps** (`pio device monitor`).

**4. Pruebas**

- Las pruebas funcionales se ejecutan enviando comandos por el monitor serial (`DISP1`, `STATUS`, `CFG,1,8,30,1`, etc.).
- Los casos de prueba definidos están documentados en `Docs/Embedded_firmware_design_document/Casos_de_prueba`.

**5. Control de versiones**

- Los archivos de build (`.pio/`, `build/`) están excluidos por `.gitignore`.
- Cada cambio funcional verificado se consolida en un commit descriptivo antes de subir al repositorio.
- La documentación en `Docs/` se actualiza en paralelo al firmware cuando hay cambios de comportamiento, interfaces o parámetros.
