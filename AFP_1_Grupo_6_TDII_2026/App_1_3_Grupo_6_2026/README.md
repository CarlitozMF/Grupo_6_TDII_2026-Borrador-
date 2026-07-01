# App 1.3: Selector de Secuencias (MEF Jerárquica)

## Título y Objetivos
**App 1.3: Sistema Multisecuencia de LEDs**

Implementar un selector de cuatro secuencias de iluminación distintas, alternadas mediante el pulsador de usuario. El objetivo es consolidar el uso de **Máquinas de Estados Finitos (MEF) jerárquicas** y la gestión de eventos no bloqueantes para la transición de modos de operación.

## Especificaciones del Circuito
* **Placa:** Nucleo-STM32F439ZI.
* **Entrada:** Pulsador (USER_BTN) para conmutación cíclica (Modo 1 -> 2 -> 3 -> 4 -> 1).
* **Salidas:** 3 LEDs onboard con lógicas independientes por cada secuencia.

## Teoría de Operación
El sistema opera bajo una **MEF Principal** que define el modo de ejecución actual. Al presionar el botón, el sistema realiza un **reset de seguridad** (limpieza de registros y variables) para evitar colisiones entre secuencias. 
1. **Secuencia 1:** Alternancia secuencial (150ms).
2. **Secuencia 2:** Parpadeo simultáneo (300ms).
3. **Secuencia 3:** Parpadeo asíncrono con frecuencias independientes (100ms, 300ms, 600ms).
4. **Secuencia 4:** Lógica complementaria (LED2 opuesto a LED1/3, 150ms).

## Arquitectura del Software
* **Capa 1:** Mapeo de hardware mediante `leds_array`.
* **Capa 2:** Drivers de escritura y toggle de pines.
* **Capa 3:** MEF de Selección (Switch principal) con sub-máquinas de estados para la lógica de tiempo.


## Detalles de Robustez
* **Reset de Seguridad:** Ante cada cambio de secuencia, todas las variables de control (`contador`, `divisor`, `ultimo_disparo`, `modo`) se reinician, garantizando que cada secuencia inicie desde un estado conocido.
* **Manejo de estados inválidos:** Se incluyó un bloque `default` para reencauzar la ejecución hacia la `Secuencia_1` en caso de errores de memoria.

## Mapeo de Hardware
| LED | Puerto | Pin | Función |
| :--- | :--- | :--- | :--- |
| LD1 | GPIOB | LD1_Pin | LED Verde |
| LD2 | GPIOB | LD2_Pin | LED Azul |
| LD3 | GPIOB | LD3_Pin | LED Rojo |

## Conclusión
La implementación demuestra la escalabilidad de la arquitectura propuesta. Al encapsular cada secuencia dentro de un `case` de la MEF, se logra un sistema altamente modular que permite añadir nuevas secuencias con un impacto nulo sobre las ya existentes.