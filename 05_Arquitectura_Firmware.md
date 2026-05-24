# Arquitectura de Firmware

El firmware del sistema SmartDose está diseñado bajo una arquitectura modular basada en FreeRTOS, permitiendo la ejecución concurrente de tareas como monitoreo de sensores, control de actuadores, interfaz de usuario y gestión de tiempo.

## Arquitectura modular propuesta

Aunque actualmente el firmware se encuentra centralizado en `main.c`, la arquitectura propuesta divide el sistema en módulos independientes para mejorar mantenibilidad y escalabilidad.

<img width="650" height="300" alt="image" src="https://github.com/user-attachments/assets/d20bbc90-6d76-4ce7-83e0-5b383e4f2daa" />
