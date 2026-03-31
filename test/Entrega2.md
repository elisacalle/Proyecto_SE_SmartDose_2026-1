# Requisitos funcionales y no funcionales del sistema, plan de testing y pruebas para los requerimientos

## Requerimientos funcionales

### RF-01 — Programar hasta cuatro horarios diarios desde la GUI
El sistema debe permitir programar hasta cuatro horarios de dispensación diarios desde la interfaz gráfica táctil. Este requerimiento es fundamental porque un paciente con enfermedad crónica puede requerir múltiples tomas a lo largo del día en horarios distintos. El límite de cuatro horarios cubre la mayoría de esquemas farmacológicos habituales sin sobrecomplicar la interfaz. Al hacerlo configurable desde la pantalla táctil, se garantiza que el propio paciente o su cuidador pueda ajustar los horarios sin necesidad de herramientas externas ni conocimientos técnicos.

### RF-02 — Dispensar automáticamente en el horario programado
El sistema debe dispensar automáticamente la dosis programada cuando el horario configurado se cumpla. La automatización de la dispensación elimina la dependencia de la memoria del paciente, que es precisamente el eslabón más débil en el cumplimiento de tratamientos crónicos. Al delegar esta responsabilidad al sistema embebido con una referencia temporal precisa, se garantiza que la medicación esté disponible en el momento exacto indicado por el médico, independientemente del estado cognitivo o de la rutina del paciente.

### RF-03 — Detectar si la pastilla fue dispensada correctamente
El sistema debe detectar mediante un sensor si la pastilla fue efectivamente dispensada al exterior del compartimento. Verificar que la dispensación ocurrió realmente es crítico para la seguridad del paciente. Un mecanismo de dispensación puede activarse sin que la pastilla haya salido debido a un atasco, desgaste mecánico o compartimento vacío. Sin esta verificación el sistema no tendría forma de distinguir entre una dispensación exitosa y un fallo, lo que podría llevar al paciente a creer que su dosis fue suministrada cuando en realidad no lo fue.

### RF-04 — Alerta si el usuario no recoge la pastilla
El sistema debe emitir una alerta sonora y visual cuando el usuario no recoja la pastilla dentro de un tiempo límite configurado. Que la pastilla esté disponible no garantiza que el paciente la tome, especialmente en adultos mayores que pueden estar distraídos, dormidos o con dificultades de movilidad. La alerta actúa como un recordatorio activo que persiste hasta que el paciente responda. Combinar señal sonora y visual aumenta la probabilidad de que el paciente perciba la notificación sin importar sus condiciones sensoriales o el entorno en el que se encuentre.

### RF-05 — Detectar compartimento vacío antes de dispensar
El sistema debe detectar si el compartimento de medicamentos está vacío antes de intentar realizar una dispensación. Intentar dispensar sobre un compartimento vacío puede dañar el mecanismo físico del dispositivo y genera un evento de fallo que podría confundirse con una dispensación exitosa. Detectar esta condición antes de actuar permite al sistema tomar una decisión informada: en lugar de ejecutar un ciclo de dispensación inútil, puede notificar el problema de forma inmediata y precisa, protegiendo tanto al dispositivo como al paciente.

### RF-06 — Alertar cuando el compartimento esté vacío
El sistema debe emitir una alerta sonora y visual cuando el compartimento de medicamentos esté vacío. Un compartimento vacío sin notificación significa que el dispositivo seguirá intentando dispensar en cada horario programado sin éxito, y el paciente no recibirá su medicación sin saber por qué. La alerta garantiza que el paciente o su cuidador sean informados oportunamente para recargar el dispensador antes de que se pierda una toma, manteniendo la continuidad del tratamiento.

### RF-07 — Registrar eventos con marca temporal
El sistema debe registrar cada evento relevante como dispensaciones, alertas y errores, mediante un esquema de logging estructurado con marca temporal. El registro de eventos con hora exacta permite reconstruir el comportamiento del sistema en cualquier momento posterior. Esto es valioso tanto para el cuidador que quiere verificar si el paciente tomó su medicación, como para el desarrollador que necesita depurar un comportamiento inesperado. Sin trazabilidad temporal, cualquier análisis del funcionamiento del sistema se vuelve especulativo e inútil en contextos clínicos.

### RF-08 — Clasificar el log con niveles de severidad
El sistema debe clasificar cada entrada del log con un nivel de severidad: INFO, WARN o ERROR. Clasificar los eventos por severidad hace el log consultable y accionable. Permite filtrar rápidamente los eventos que requieren atención inmediata sin tener que revisar toda la historia de operación normal. Un operador o cuidador puede enfocarse en los niveles de mayor severidad para identificar problemas, mientras que los registros informativos documentan el funcionamiento correcto del sistema a lo largo del tiempo.

### RF-09 — Mostrar hora, horarios configurados y estado en pantalla
El sistema debe mostrar en la pantalla táctil la hora actual, los horarios configurados y el estado del compartimento. El paciente necesita tener visibilidad permanente del estado del sistema sin depender de herramientas externas. Ver la hora actual le permite orientarse temporalmente, ver los horarios programados le recuerda cuándo esperar la próxima dosis, y conocer el estado del compartimento le indica si debe recargarlo. Concentrar esta información en la pantalla convierte al dispositivo en una fuente confiable de información de salud para el usuario.

### RF-10 — Modificar o eliminar horarios desde la GUI
El sistema debe permitir modificar o eliminar horarios de dispensación previamente configurados desde la interfaz gráfica. Los tratamientos médicos no son estáticos: los médicos ajustan dosis, frecuencias y horarios a lo largo del tiempo según la evolución del paciente. Un sistema que no permita actualizar la configuración pierde utilidad práctica muy rápidamente y obliga a intervenciones técnicas para cada cambio, lo cual es inaceptable para el perfil de usuario al que está dirigido el dispositivo.

---

## Requerimientos no funcionales

### RNF-01 — Respuesta a entrada táctil en menos de 500 ms
El sistema debe responder a cualquier entrada del usuario en la pantalla táctil en menos de 500 ms. La velocidad de respuesta de la interfaz determina directamente si el usuario percibe el dispositivo como funcional o defectuoso. Una respuesta lenta genera desconfianza, especialmente en adultos mayores que pueden interpretar la demora como un error y repetir la acción, generando interacciones no deseadas. Mantener este umbral de respuesta garantiza una experiencia de uso natural y confiable para el perfil de usuario del dispositivo.

### RNF-02 — Precisión temporal máxima de ±1 minuto
El sistema debe mantener una precisión temporal máxima de ±1 minuto respecto a la hora programada. En tratamientos farmacológicos crónicos el horario de administración puede tener implicaciones clínicas directas, particularmente en medicamentos cuya efectividad depende de mantener niveles estables en el organismo. Una desviación mayor podría comprometer la eficacia del tratamiento o generar efectos no deseados, por lo que contar con una referencia temporal confiable es un aspecto no negociable del sistema.

### RNF-03 — Firmware modular con manejo de errores y documentación
El firmware debe estar estructurado en módulos independientes con manejo explícito de errores y documentación en el código. Un sistema embebido destinado al cuidado de la salud debe ser mantenible, auditable y robusto. La modularidad permite aislar fallos, facilita las pruebas de cada componente y hace posible que cualquier miembro del equipo entienda y modifique una parte del sistema sin afectar las demás. El manejo explícito de errores asegura que ningún fallo quede silencioso ni cause comportamientos impredecibles.

### RNF-04 — Operación continua sin reinicio manual
El sistema debe operar de forma continua sin requerir reinicio manual, tolerando condiciones de fallo sin detener su ciclo de operación normal. El dispositivo opera en el hogar de un paciente que en muchos casos vive solo y no cuenta con asistencia técnica permanente. Un sistema que se bloquea ante un error obliga a una intervención externa que puede no ocurrir a tiempo, resultando en tomas de medicación perdidas. La resiliencia operacional no es una característica deseable sino un requisito de seguridad para este tipo de aplicación.


# Pruebas para esos requerimientos

REQUERIMIENTOS FUNCIONALES
RF-01 Programar hasta 4 horarios
Estrategia
1)	Programar 4 horarios desde la GUI 
2)	Guardar configuración 
3)	Apagar el sistema 
4)	Encender nuevamente 
5)	Verificar: 
•	Los horarios siguen almacenados 
•	Son correctos 
•	El sistema los usa para dispensar

Validación 
1)	El sistema almacena exactamente 4 horarios.
2)	Tras reinicio, los horarios siguen iguales.


RF-02 Dispensación automática 
Estrategia
1)	Configurar horario cercano
2)	Medir tiempo real vs Activación
Validación 
1)	Diferencia + o – 1 min (se usa cronómetro)


RF-03 Detección de dispensación
Estrategia
1)	Simular dispensación normal
2)	Bloquear salida(atasco)
3)	Quitar pastilla (compartimento vacío)
Validación 
1)	Sensor detecta:
OK-----INFO
Falla----ERROR



RF-04 Alerta por no recolección

Estrategia
1)	Dispensar
2)	No retirar pastilla

Validación 
1)	Led y buzzer se activan después de un tiempo


RF-05 Detectar compartimento vacío
Estrategia
1)	Vaciar compartimento
2)	Forzar evento de dispensación

Validación 
1)	No se activa motor 
2)	Se genera alerta

RF-06 Alerta compartimento vacío
Estrategia
1)	Activar condición de vacío manualmente
Validación 
1)	LED + buzzer + log WARN/ERROR

RF-07 Logging con timestamp
Estrategia
1)	Ejecutar eventos
2)	Leer UART
Validación 
1)	Timestamp coincide con RTC

RF-08 Severidad del log
Estrategia
1)	Generar eventos de cada tipo
Validación 
1)	INFO/WARN/ERROR correctamente asignados

RF-09 Información en pantalla
Estrategia
1)	Comparar GUI vs valores internos
Validación 
1)	Datos coinciden con sistema

RF-10 Modificar/Eliminar horarios
Estrategia
1)	Crear horario
2)	Modificarlo
3)	Eliminarlo
4)	Esperar evento
Validación 
1)	Sistema responde al cambio en tiempo real

REQUERIMIENTOS NO FUNCIONALES
RNF-01 (Tiempo de respuesta<500 ms)
Estrategia
1)	Medir con cronómetro o logs
Validación 
1)	Tiempo <=500 ms




RNF-02 Precisión temporal
Estrategia
1)	Comparar RTC vs reloj externo por varios minutos
Validación
1)	Error <= 1min

RNF-03 Firmware modular
Estrategia
1)	Revisión de código
Validación 
1)	Separación en módulos:
-GUI
-RTC
-Dispensador
-Sensores

RNF-04 Operación continua
Estrategia
1)	Simular fallos:
-Sensor desconectado
-Compartimento vacío
2) Ejecutar durante varias horas


Validación 
1)	Sistema no se detiene


ARTEFACTOS A USAR
1)	Monitor serial (UART)
2)	Cronómetro
3)	Multímetro
4)	Cámara
5)	Logs guardados
6)	Scripts


Tipos de test
Pruebas unitarias (Prueba de una sola pieza aislada del sistema)
1)	Leer hora del RTC (DS3231) 
2)	Detectar si el sensor cambia de estado 
3)	Encender/apagar buzzer 
4)	Dibujar algo en la pantalla
Pruebas de integración (Prueba de interacción de módulos juntos)
1)	Configuras horario en GUI 
2)	Se guarda en memoria 
3)	RTC llega a la hora 
4)	Se activa dispensación
Pruebas de sistema (Se prueba como lo ejecutaría el usuario)
1)	Usuario configura horarios 
2)	Espera
3)	Sistema dispensa
4)	Usuario recoge
5)	Se registra en log
Pruebas de fallo (Probar lo que pasa cuando las cosas salen mal)
1)	Compartimento vacío
2)	Pastilla atascada
3)	Usuario no recoge
4)	Sensor desconectado
5)	RTC falla


