# SmartDose: Sistema Automatizado de Dispensación de Medicamentos Programados

## Introducción
#### Contexto del problema:
A medida que la población colombiana tiende al envejecimiento, y el aumento sostenido de enfermedades crónicas, tales como diabetes o hipertensión, ha generado un escenario en el que millones de personas dependen de instrucciones farmacológicas complejas y específicas para conservar su calidad de vida. Sin embargo, gestionar mentalmente qué se debe tomar y a qué hora, representa una carga cognitiva considerable, principalmente entre los adultos mayores que no tienen un cuidador constante. Dicho esto, el olvido de tomar los medicamentos no es un evento irrelevante, puede desencadenar crisis de salud, empeoramiento del cuadro clínico que deriva en hospitalizaciones de emergencia o el deterioro acelerado de las condiciones a tratar, todas consecuencias que, con la administración correcta de los fármacos, se pueden controlar.

#### Importancia y por qué vale la pena resolverlo:
La mala adherencia terapéutica no resulta en consecuencias únicamente para el paciente: representa una sobrecarga al ya presionado sistema de salud pública y a las familias que deben asumir el esfuerzo económico y de tiempo para acompañar a su ser querido y garantizar su bienestar. Si se logra automatizar la dispensación de medicamentos, se elimina el eslabón más débil de la cadena, la memoria humana y la capacidad de seguir indicaciones; más allá del beneficio directo que obtiene el paciente al seguir con su tratamiento como está estipulado, esta solución también les devuelve libertad a los pacientes y reduce la carga del entorno familiar. Resolver este problema con un sistema embebido de bajo costo convierte la solución en una replicable, accesible y sobre todo útil para la sociedad.

#### Área de aplicación:
SmartDose se encuentra justo en la frontera donde, los sistemas embebidos, la ingeniería aplicada a la salud y automatización y control se encuentran. No es un dispositivo médico en el sentido legal o regulatorio, es una solución de ingeniería orientada a un problema humano concreto, dándole visibilidad a las dificultades que día a día pueden enfrentar los enfermos crónicos para su tratamiento. Este proyecto aspira a demostrar como la electrónica, sensores de bajo costo y microcontroladores tienen la capacidad de construir herramientas que impacten realmente a la sociedad.


## Descripción del Problema

La adherencia terapéutica, entendida como el cumplimiento correcto de un tratamiento farmacológico en dosis, frecuencia y horario, representa uno de los desafíos más críticos en el cuidado de pacientes con enfermedades crónicas. Según la Organización Mundial de la Salud, cerca del 50% de los pacientes con enfermedades crónicas no toman sus medicamentos de la forma prescrita, lo que deriva en complicaciones clínicas evitables, hospitalizaciones innecesarias y un deterioro progresivo de la calidad de vida. Este problema se agudiza significativamente en adultos mayores, quienes con frecuencia deben gestionar múltiples medicamentos con horarios distintos, presentan dificultades cognitivas asociadas al envejecimiento y en muchos casos no cuentan con acompañamiento permanente de un cuidador.

La situación actual en hogares y entornos de cuidado domiciliario depende casi exclusivamente de la memoria del paciente o de recordatorios manuales por parte de los familiares o cuidadores. Los pastilleros convencionales, aunque organizan visualmente la medicación por días y tomas, no generan alertas, no verifican si la pastilla fue tomada efectivamente y no registran el historial de dispensaciones. Las alarmas de celular, si bien son una solución parcial, requieren que el paciente identifique correctamente cuál medicamento tomar, en qué dosis y en qué momento, lo que en pacientes con deterioro cognitivo representa una fuente frecuente de errores. 

Se identifica que lo que falta es un sistema autónomo, embebido y de bajo costo que gestione de forma automática los horarios de medicación, dispense físicamente el medicamento en el momento correcto, alerte al paciente de forma sonora para que no ignore la toma, detecte si el compartimiento está vacío antes de intentar dispensar y registre con trazabilidad temporal cada evento relevante del proceso. Un sistema así eliminaría la dependencia de la memoria del paciente, reduciría los errores de medicación y proporcionaría al cuidador o familiar un historial consultable del comportamiento del sistema.

Un caso aplicado puede ser el de la abuela de uno de nuestros integrantes, que debido a la diabetes y su avanzada edad debe tomar durante el día diferentes cantidades de medicamentos a horas muy exactas. Debido a su condición de vejez es propensa a que se le olvide tomarlos a las horas precisas y adecuadas, como aún vive sola, para su familia tiende a ser un poco complejo supervisar o consultar todo el historial de toma de las pastillas de forma adecuada, la invención de este sistema sería una alternativa o solución bastante factible para su condición y de esta forma podría sentirse mas independiente y a su familia le daría la tranquilidad de que cumple de forma precisa con la toma de sus medicamentos. 


## Alcance del Proyecto
#### ¿Qué incluye el proyecto?
-Programación de hasta 4 horarios de dispensación diarios configurables desde la GUI (Graphical User Interface).

-Dispensación automática de n dosis (pastilla) por evento programado.

-Detección de si la pastilla fue dispensada correctamente mediante sensor de presencia.

-Alerta sonora y visual (buzzer + led) si el usuario no recoge la pastilla después de un tiempo.

-Alerta si el compartimento de medicamentos está vacío.

-Sistema de logging con timestamp, niveles INFO/WARN/ERROR y códigos estructurados. Se utiliza comunicación UART.

-Interfaz gráfica ILI9341([LCD-3.2-TOUCH-ILI9341] Pantalla táctil LCD de 3.2’’. ILI9341), el protocolo que usa es el SPI.

-RTS DS3231 para manejo del tiempo, se comunica por vía I2C.

-Firmware estructurado, con manejo de errores y documentación.

#### Prototipo esperado
-Tarjeta universal ensamblada dentro de una carcasa impresa en 3D o construida en acrílico/madera MDF.

-Fuente de alimentación externa.

#### Límites técnicos
-Microcontrolador ESP32.

-Protocolos usados: I2C (RTC DS3231), UART (logging) y SPI (interfaz).

-El sistema no distingue el tipo de medicamento.

-Capacidad de dosis a dispensar (por determinar (depende del diseño físico)).

-No es un dispositivo medico certificado.

-No incluye conectividad WI-FI o bluetooth.

-No controla temperatura de almacenamiento.

## Objetivo General

Desarrollar un sistema embebido de bajo costo para la dispensación automática de medicamentos en horarios programados, con configuración mediante interfaz gráfica, generación de alertas al usuario, detección de condiciones de fallo y registro trazable de eventos con marca temporal.

## Objetivos Específicos

#### #1
Implementar la programación de hasta cuatro horarios diarios configurables desde la interfaz gráfica del sistema.

#### #2
Automatizar la dispensación física de la dosis programada en cada evento de medicación.

#### #3
Incorporar mecanismos de alerta sonora y visual, incluyendo notificación por no recolección y compartimiento vacío.

#### #4
Registrar los eventos relevantes del sistema mediante un esquema de logging estructurado con marca temporal y niveles de severidad.

## Asignación de Roles
#### Technical Lead:
Elisa Calle Escobar

#### Firmware Engineer:
David Aristizábal

#### Hardware Integration Engineer:
Gabriel García 

#### Verification & Testing Engineer:
Santiago Soto
