# ![][image1]
# UNIVERSIDAD DE BUENOS AIRES
# Facultad de Ingeniería
# TA134 – Sistemas Embebidos
# Curso 1 – Grupo 3
# Memoria del Trabajo Final: Ascensor Embebido de 3 Pisos con Control de Acceso RFID
| Autor | Padrón | Mail |
| ----- | ----- | ----- |
| Colodro, Felipe\] | 106.433 | fcolodro@fi.uba.ar |
| dos Reis, Mariana\] | 111106 | utoscan@fi.uba.ar |
| Toscan, Uma\] | 111106 | utoscan@fi.uba.ar |

2026 | 1er Cuatrimestre 

Taller de Sistemas Embebidos (TA134) 

Universidad de Buenos Aires | Facultad de Ingeniería 

# Resumen
En el presente trabajo se desarrolló una maqueta funcional de un ascensor de 3 paradas (Planta Baja, Piso 1 y Piso 2\) basada en una placa STM32 Nucleo-F446RE, programada en lenguaje C bajo el paradigma Bare Metal (sin sistema operativo). El sistema gestiona el llamado de piso, el desplazamiento del motorreductor, la apertura/cierre de puerta, la detección de sobrecarga mediante celda de carga, y el control de acceso mediante tarjetas RFID.

El diseño de software se estructura mediante un **Ejecutor Cíclico (Super-Loop)** con base de tiempo de 1ms (SysTick → Callback), organizado en las etapas **Escrutar → Procesar → Actuar**, donde la etapa de Procesar implementa una **Máquina de Estados Finitos (FSM)** que gobierna el comportamiento del ascensor. La comunicación entre etapas se resuelve mediante una **cola de eventos** (array de estructuras), lo que permite que fuentes de entrada heterogéneas (botones físicos, comandos por Bluetooth, sensores) se traten de forma unificada.

El sistema cuenta además con persistencia de configuración en Flash interna, control de velocidad del motor por PWM, indicación visual (LEDs) y sonora (buzzer), pantalla LCD 16x2 por I2C, y un modo de configuración (SET\_UP) con menú interactivo.

# Registro de versiones
| Revisión | Cambios realizados | Fecha |
| ----- | ----- | ----- |
| 1.0 | Creación del esqueleto y estructura base del documento. | 08/07/2026 |
| 1.1 | Redacción detallada, desarrollo y completado de las secciones del informe. | 11/08/2026 |

# Índice General
* Capítulo 1: Introducción general  
  * 1.1. Análisis de necesidad y objetivo

  * 1.2. Productos comparables

  * 1.3. Alcance y limitaciones

* Capítulo 2: Introducción específica  
  * 2.1. Requisitos del proyecto

  * 2.2. Casos de uso

  * 2.3. Elementos de hardware

* Capítulo 3: Diseño e implementación  
  * 3.1. Esquema eléctrico y conexionado

  * 3.2. Descripción del comportamiento (Máquina de Estados)

  * 3.3. Arquitectura del firmware  
    * 3.3.1. Módulo Tick

    * 3.3.2. Módulo Eventos

    * 3.3.3. Módulo Escrutar

    * 3.3.4. Módulo FSM Ascensor

    * 3.3.5. Módulo Actuadores

    * 3.3.6. Módulo Configuración (Flash interna)

    * 3.3.7. Módulo RFID (RC522)

    * 3.3.8. Módulo Bluetooth (HM-10)

* Capítulo 4: Ensayos y resultados  
  * 4.1. Prueba de integración (video)

  * 4.2. Salida de consola y Build Analyzer

  * 4.3. Medición y análisis de tiempos de ejecución (WCET)

  * 4.4. Cálculo del Factor de Uso (U) de la CPU

  * 4.5. Medición y análisis de consumo

  * 4.6. Cumplimiento de requisitos

* Capítulo 5: Conclusiones  
  * 5.1. Resultados obtenidos

  * 5.2. Próximos pasos

* Capítulo 6: Uso de herramientas de IA

* Capítulo 7: Bibliografía y referencias

# Capítulo 1: Introducción general
## 1.1. Análisis de necesidad y objetivo
Los edificios de baja altura con pocos pisos suelen requerir sistemas de ascensor simples, confiables y de bajo costo. Este tipo de sistema debe gestionar el llamado de piso, el desplazamiento seguro de la cabina, la apertura y cierre de puerta, y mecanismos básicos de seguridad como la detección de sobrecarga y la parada de emergencia.   
El objetivo del proyecto es diseñar e implementar el firmware y el hardware de una maqueta de ascensor de 3 paradas, aplicando los contenidos fundamentales del Taller de Sistemas Embebidos: programación Bare Metal orientada a eventos, máquinas de estado, manejo de periféricos por polling/interrupciones/DMA, buses I2C y SPI, persistencia de configuración, y una interfaz de usuario mediante menú interactivo.

## 1.2. Productos comparables
A diferencia de los kits educativos de ascensores en miniatura disponibles comercialmente (que suelen ser de código cerrado y sin posibilidad de personalización), este proyecto propone un desarrollo abierto donde cada aspecto —desde el control de acceso por RFID hasta el bajo consumo— es diseñado e implementado por el equipo.

**1. Fundino elevador**

   Este sistema de formación para ascensores de cuatro plantas proporciona una plataforma que permite a alumnos y estudiantes llevar a cabo una amplia gama de tareas de programación de PLC utilizando el entorno de desarrollo Arduino sobre la base de una simulación realista de ascensor. Las aplicaciones eléctricas y mecánicas están estrechamente relacionadas y ofrecen un alto nivel de oportunidades de aprendizaje.

![][image2]  
*Figura 1.1: Fundino elevador.*
**2. Ascensor Encoder**

Los encoders son dispositivos que pueden convertir la posición o el movimiento de un eje a señales digitales que pueden ser leídas por un controlador lógico programable (PLC). En el caso de este proyecto, el encoder ayuda a determinar la posición exacta de la cabina entre los pisos y asegurar paradas precisas y suaves. Proyectos como este no solo son educativos, sino que también pueden ser muy gratificantes, ya que se ve una idea convertirse en una realidad funcional.  
   ![][image3]

*Figura 1.2: Ascensor Encoder.*

La Tabla 1.1 contrasta las prestaciones de los dos productos comerciales de referencia contra el prototipo desarrollado en este trabajo.

| Aspecto | Funduino Elevator (4 niveles) | Ascensor Ecopech (3 pisos con Encoder) | Prototipo desarrollado (Ascensor Inteligente) |
| ----- | ----- | ----- | ----- |
| **Paradas** | 4 niveles | 3 pisos | 3 (Planta Baja, Piso 1 y Piso 2\) |
| **Capacidad** | Maqueta educativa | Maqueta educativa | Detección de sobrecarga por celda de carga (umbral configurable) |
| **Máquina de tracción** | Motor DC con reductor y puente en H L293D | Motor DC | Motor DC con driver L298N y control por **PWM** |
| **Comandos** | Lógica 5V (Funduino MEGA 2560 R3) | Lógica 5V (Arduino Nano) | Lógica 3,3-5V (**STM32F103 / F446RE**) |
| **Interfaz de usuario** | Pantalla OLED, botoneras de piso (subida/bajada) y panel interior completo | Sensor Encoder y Buzzer (sin pantalla incluida en el kit base) | Pantalla **LCD 16x2 (I2C)** y botoneras interior/exterior |
| **Seguridad** | Barreras de luz por nivel y botón de alarma | Encoder para paradas precisas y suaves | Celda de carga (**HX711**) y botón de emergencia con **prioridad absoluta** |
| **Registro / Telemetría** | Conexión opcional para Bluetooth (módulo no incluido) | No especificado | Telemetría en tiempo real mediante **Bluetooth (HM-10)** |
| **Persistencia** | Basada en la memoria interna del microcontrolador | No especificado | Parámetros configurables persistidos en **EEPROM externa / Flash interna** |
| **Infraestructura** | Bastidor de perfil de aluminio 20x20 | Estructura en MDF y piezas 3D | Maqueta experimental de laboratorio, sin obra civil |
| **Precio** | **184,90 €** (aprox. $190.000 ARS) | **S/ 190.00** (aprox. $50.000 ARS) | Prototipo de laboratorio (Costo estimado: **\~$55.000 ARS**) |

*Tabla 1.1: Comparación de prestaciones entre productos comerciales y el prototipo desarrollado.*
En resumen, el mercado ofrece desde kits educativos básicos basados en Arduino con lógica simplificada, hasta instalaciones industriales certificadas de alto costo y código cerrado. Ninguna de estas soluciones combina la flexibilidad de un desarrollo Bare Metal en STM32 con funciones de seguridad avanzada (celda de carga) y telemetría activa por Bluetooth en un formato de aprendizaje abierto. Esto justifica el desarrollo de un sistema propio que integre control de precisión y auditoría remota con hardware accesible.
## 1.3. Alcance y limitaciones
   Alcance implementado:

* Electrónica de control: Gestión de 3 paradas con lógica automática y control de motor por PWM.

* Interfaz de usuario: Botoneras (internas/externas), display LCD 16x2 y telemetría Bluetooth.

* Seguridad: Detección de sobrecarga (HX711) y botón de emergencia con prioridad absoluta.

* Configuración: Modo SET\_UP con persistencia de parámetros en EEPROM externa.

  Fuera de alcance actual:

* Diseño mecánico de infraestructura civil o edificio real (maqueta demostrativa).

* Escalabilidad a más de 3 paradas por restricciones de tiempo y presupuesto.  
# Capítulo 2: Introducción específica
## 2.1. Requisitos del proyecto
En la Tabla 2.1 se detallan los principales requisitos funcionales del sistema:

| Grupo | ID | Descripción |
| ----- | ----- | ----- |
| Movimiento | 1.1 | El sistema desplazará la cabina entre 3 paradas: Planta Baja, Piso 1 y Piso 2\. |
|  | 1.2 | El sistema detectará la llegada a cada piso mediante un sensor reed switch dedicado. |
|  | 1.3 | El sistema controlará la velocidad del motor mediante PWM. |
|  | 1.4 | Si el viaje excede un tiempo máximo sin detectar llegada, el sistema pasará a estado de FALLA. |
| Llamado de piso | 2.1 | El sistema permitirá solicitar un piso mediante botonera externa (en cada piso) e interna (en la cabina). |
|  | 2.2 | El sistema encolará pedidos pendientes si se solicitan mientras el ascensor está en movimiento. |
| Puerta | 3.1 | El sistema abrirá la puerta automáticamente al llegar a un piso. |
|  | 3.2 | El sistema cerrará la puerta automáticamente tras un tiempo configurable de espera. |
|  | 3.3 | El sistema verificará el cierre de puerta mediante sensor reed switch antes de iniciar un viaje. |
| Seguridad | 4.1 | El sistema detendrá el ascensor y abrirá la puerta ante la activación de la llave de emergencia. |
|  | 4.2 | El sistema detectará sobrecarga mediante celda de carga y bloqueará nuevos viajes mientras esté activa. |
|  | 4.3 | El sistema emitirá señales sonoras distintivas para llegada, tecla, sobrecarga, emergencia y falla. |
| Control de acceso | 5.1 | El sistema leerá el UID de tarjetas RFID mediante el módulo RC522. |
|  | 5.2 | El sistema podrá restringir el llamado de piso a tarjetas autorizadas (configurable). |
| Interfaz de usuario | 6.1 | El sistema mostrará el piso actual y el estado del ascensor en un display LCD 16x2 por I2C. |
|  | 6.2 | El sistema indicará mediante LEDs el estado de movimiento, puerta, SET\_UP, falla y emergencia. |
|  | 6.3 | El sistema permitirá control remoto (llamado de piso) mediante Bluetooth (HM-10). |
| Configuración (SET\_UP) | 7.1 | El sistema contará con un modo SET\_UP con menú interactivo por LCD y botones. |
|  | 7.2 | El sistema permitirá configurar y persistir: tiempo de puerta abierta, umbral de sobrecarga y velocidad del motor. |
|  | 7.3 | La configuración persistirá en Flash interna del microcontrolador entre reinicios. |
| Arquitectura de software | 8.1 | El sistema se implementará Bare Metal, sin sistema operativo, bajo el paradigma Event-Triggered. |
|  | 8.2 | El sistema utilizará un Ejecutor Cíclico (Super-Loop) con una vuelta completa menor a 1ms. |
|  | 8.3 | El sistema utilizará una base de tiempo de 1ms (SysTick → Callback) para todas las temporizaciones. |
|  | 8.4 | Todas las tareas serán no bloqueantes (temporizadas o no temporizadas), sin uso de HAL\_Delay() en la lógica de aplicación. |

*Tabla 2.1: Requisitos del proyecto.*

## 2.2. Casos de uso
### 2.2.1. Caso de uso 1: Un usuario llama al ascensor desde un piso
| Elemento | Definición |
| ----- | ----- |
| Disparador | Un usuario presiona el botón externo de llamado en Planta Baja, Piso 1 o Piso 2\. |
| Precondiciones | El sistema está en modo NORMAL, en estado IDLE (o con otro pedido pendiente), sin sobrecarga activa. |
| Flujo principal | El botón genera un evento de pedido de piso, que se encola en la lista de pedidos pendientes. Si el ascensor está en IDLE, cierra la puerta (si estaba abierta), viaja al piso solicitado controlando el motor por PWM, detiene el motor al detectar el reed switch del piso destino, y abre la puerta automáticamente. |

*Tabla 2.2: Caso de uso 1\.*

### 2.2.2. Caso de uso 2: Se activa la llave de emergencia durante un viaje
| Elemento | Definición |
| ----- | ----- |
| Disparador | El usuario acciona la llave de emergencia mientras el ascensor está en movimiento. |
| Precondiciones | El sistema está en cualquier estado que no sea ya EMERGENCIA. |
| Flujo principal | El sistema detiene el motor de inmediato, abre la puerta, enciende el LED de emergencia, activa el buzzer en patrón continuo y muestra el aviso en el LCD, independientemente del estado en el que se encontraba. El sistema permanece en este estado hasta que la llave vuelve a su posición normal. |

*Tabla 2.3: Caso de uso 2\.*

### 2.2.3. Caso de uso 3: Un usuario configura los parámetros del sistema (modo SET\_UP)
| Elemento | Definición |
| ----- | ----- |
| Disparador | El usuario mantiene presionado el botón interno de Planta Baja durante el encendido del sistema. |
| Precondiciones | El sistema está apagado o acaba de reiniciarse. |
| Flujo principal | El sistema arranca en modo SET\_UP en lugar de NORMAL. El usuario navega entre los parámetros configurables (tiempo de puerta, umbral de sobrecarga, velocidad de motor) usando el botón de Planta Baja como "navegar" y el de Piso 2 como "incrementar valor". Al llegar a la opción "Guardar y salir" y confirmar (llave de emergencia), el sistema persiste los valores en Flash interna y puede reiniciarse en modo NORMAL. |

*Tabla 2.4: Caso de uso 3\.*

### 2.2.4. Caso de uso 4: Control de acceso mediante tarjeta RFID
| Elemento | Definición |
| ----- | ----- |
| Disparador | El usuario acerca una tarjeta/llavero RFID al módulo RC522. |
| Precondiciones | El sistema tiene habilitado el control de acceso (ACCESO\_RFID\_REQUERIDO \= 1). |
| Flujo principal | El sistema lee el UID de la tarjeta mediante el protocolo REQA \+ anticolisión sobre SPI. Si el UID coincide con la lista de tarjetas autorizadas, se habilita el llamado de piso; caso contrario, se rechaza el pedido y se indica mediante LCD/buzzer. |

*Tabla 2.5: Caso de uso 4\.*

## 2.3. Elementos de hardware
### 2.3.1. Placa de desarrollo
| Como unidad central de procesamiento se utilizó la placa STM32 Nucleo-F446RE, compatible con el ecosistema HAL/CubeMX. Gestiona toda la lógica del ascensor mediante una máquina de estados, el manejo de tiempos críticos mediante SysTick y TIM2 (PWM), y la comunicación con los periféricos mediante I2C1, SPI1 y USART3. | ![][image4]<br>*Figura 2.1: Nucleo-F446RE utilizada.*  |
| :---- | :---: |
### 2.3.2. Motor y driver L298N
| Se utilizó un motorreductor DC de 12V acoplado a una polea para el sistema de tracción de la cabina, controlado a través de un driver L298N (puente H). La dirección de giro se controla mediante los pines IN1/IN2 (GPIO), y la velocidad mediante PWM sobre el pin ENA (TIM2\_CH1). | ![][image5]<br>*Figura 2.2: Motorreductor y driver L298N.*  |
| :---- | :---: |
### 2.3.3. Celda de carga \+ HX711
| Para la detección de sobrecarga se utilizó una celda de carga tipo barra recta de 3kg, junto al amplificador HX711, que entrega el dato mediante un protocolo propio de 2 hilos (DT/SCK). | ![][image6]<br>*Figura 2.3: Celda de carga y HX711.*  |
| :---- | :---: |
### 2.3.4. Display LCD 16x2 (I2C)
| Se utilizó un display LCD 16x2 con backpack I2C (PCF8574), que muestra el piso actual, el estado del ascensor y el menú de configuración (SET\_UP). | ![][image7]<br>*Figura 2.4: LCD 16x2 utilizado.*  |
| :---- | :---: |
### 2.3.5. Módulo RFID RC522 (SPI)
| Se integró un módulo lector RFID RC522 por SPI1, utilizado para el control de acceso opcional mediante tarjetas/llaveros. | ![][image8]<br>*Figura 2.5: Módulo RC522.*  |
| :---- | :---: |
### 2.3.6. Módulo Bluetooth HM-10 (UART)
| Se utilizó un módulo HM-10 (Bluetooth Low Energy) sobre USART3 para permitir el llamado de piso de forma remota desde una aplicación de celular. | ![][image9]<br>*Figura 2.6: Módulo HM-10.*  |
| :---- | :---: |
### 2.3.7. Botones, reed switches y llave de emergencia
| Se utilizaron 6 pulsadores (3 botoneras externas de piso \+ 3 botoneras de cabina), 3 sensores reed switch (uno por piso, para detección de llegada) y una llave de emergencia tipo switch mantenido. | \[COMPLETAR: foto de botones/reed switches\] <br>*Figura 2.7: Botones y sensores utilizados.*  |
| :---- | :---: |
### 2.3.8. LEDs y buzzer
| Se utilizaron 3 LEDs indicadores de piso y un buzzer activo para señalización sonora de eventos (llegada, tecla, sobrecarga, emergencia, falla). | \[COMPLETAR: foto de LEDs/buzzer\] <br>*Figura 2.8: LEDs y buzzer.*  |
| :---- | :---: |

# Capítulo 3: Diseño e implementación
## 3.1. Esquema eléctrico y conexionado
Para la integración física del sistema se utilizó una placa experimental soldada (sin protoboard ni cables Dupont), con interconexión de componentes mediante cables soldados. El circuito se centra en la placa Nucleo-F446RE, que gestiona los periféricos mediante las siguientes interfaces:

* **GPIO (entradas):** botones de llamado externo/interno, reed switches de piso, llave de emergencia.

* **GPIO (salidas):** LEDs indicadores, buzzer, direccionamiento del motor (IN1/IN2).

* **PWM (TIM2\_CH1):** control de velocidad del motor a través de ENA del L298N.

* **I2C1 (PB8=SCL, PB9=SDA):** display LCD.

* **SPI1 (PA5=SCK, PA6=MISO, PA7=MOSI):** módulo RFID RC522.

* **USART3 (PB10=TX, PB11=RX):** módulo Bluetooth HM-10.

* **2 hilos bit-banged (PC4=DT, PC5=SCK):** amplificador de celda de carga HX711.

En la Tabla 3.1 se detalla la asignación completa de pines:

| Bloque | Señal | Pin STM32 | Configuración |
| ----- | ----- | ----- | ----- |
| Reed switches | PB / P1 / P2 | PC6 / PC7 / PC8 | GPIO\_Input |
| Botones externos | PB / P1 / P2 | PC9 / PC10 / PC11 | GPIO\_Input |
| Botonera de cabina | PB / P1 / P2 | PC12 / PB1 / PB2 | GPIO\_Input |
| Llave de emergencia | — | PB12 | GPIO\_Input |
| LEDs de piso | PB / P1 / P2 | PB3 / PB4 / PB5 | GPIO\_Output |
| Buzzer | — | PB13 | GPIO\_Output |
| Motor L298N | IN1 / IN2 | PC0 / PC1 | GPIO\_Output |
| Motor L298N | ENA | PA0 | TIM2\_CH1 (PWM) |
| LCD I2C1 | SCL / SDA | PB8 / PB9 | I2C1 |
| RC522 SPI1 | SCK / MISO / MOSI | PA5 / PA6 / PA7 | SPI1 |
| RC522 | RST / CS | PA9 / PB6 | GPIO\_Output |
| HX711 | DT / SCK | PC4 / PC5 | GPIO\_Input / GPIO\_Output |
| HM-10 USART3 | TX / RX | PB10 / PB11 | USART3 |

*Tabla 3.1: Asignación de pines del sistema.*

\[COMPLETAR: capturas del .ioc de CubeMX, fotos de la placa soldada (frente y dorso), y foto de la maqueta mecánica completa\]

## 3.2. Descripción del comportamiento (Máquina de Estados)
El comportamiento del ascensor se modela mediante una máquina de estados finitos con los siguientes estados: IDLE, PUERTA\_ABRIENDO, PUERTA\_ABIERTA, PUERTA\_CERRANDO, VIAJANDO, EMERGENCIA y FALLA.

Desde IDLE, ante un pedido de piso pendiente, el sistema transiciona a PUERTA\_CERRANDO (asegurando el cierre antes de cualquier viaje). Al confirmarse el cierre por el reed switch de puerta, se pasa a VIAJANDO, activando el motor en la dirección correspondiente (subiendo o bajando según el piso destino) y armando un temporizador de viaje (watchdog). Al detectar la llegada al piso destino mediante el reed switch correspondiente, se detiene el motor y se transiciona a PUERTA\_ABRIENDO y luego a PUERTA\_ABIERTA, donde permanece durante un tiempo configurable antes de volver a cerrar.

Los estados EMERGENCIA y FALLA tienen prioridad absoluta: cualquier evento de activación de la llave de emergencia interrumpe el estado actual (sin importar cuál sea) y lleva al sistema a EMERGENCIA. El estado FALLA se alcanza si el temporizador de viaje vence sin detectar llegada (posible reed switch desalineado o motor trabado).

![][image10]

*Figura 3.1: Máquina de estados del sistema.*

## 3.3. Arquitectura del firmware
El firmware se estructura en las etapas **Escrutar → Procesar → Actuar**, comunicadas mediante una cola de eventos, sobre un Ejecutor Cíclico con tick de 1ms. Ningún módulo utiliza HAL\_Delay() en su lógica de operación regular.

![][image11]

*Figura 3.2: Orden de despacho de las tareas dentro de una vuelta del ejecutivo cíclico.*

### 3.3.1. Módulo Tick
Provee la base de tiempo de 1ms mediante el callback de SysTick (HAL\_SYSTICK\_Callback), y una utilidad de temporizador no bloqueante (temporizador\_t) usada por todos los demás módulos para medir tiempos sin bloquear el super-loop.

### 3.3.2. Módulo Eventos
Implementa una cola FIFO circular (array de estructuras) que desacopla la etapa de Escrutar de la etapa de Procesar. Cualquier fuente de entrada (botón físico, comando Bluetooth, sensor) empuja el mismo tipo de evento, permitiendo que la máquina de estados no distinga el origen.

### 3.3.3. Módulo Escrutar
Realiza el polling con antirrebote por software de los 6 botones, 3 reed switches y la llave de emergencia, y el polling de la celda de carga (HX711), generando eventos ante cada cambio de estado relevante.

### 3.3.4. Módulo FSM Ascensor
Implementa la máquina de estados descrita en la sección 3.2, incluyendo la lista de pedidos pendientes por piso y la lógica de selección del próximo destino.

### 3.3.5. Módulo Actuadores
Conjunto de drivers de bajo nivel para motor (PWM \+ dirección), LEDs, buzzer (con patrones no bloqueantes) y LCD (I2C).

### 3.3.6. Módulo Configuración (Flash interna)
Persiste los parámetros configurables (tiempo de puerta, umbral de sobrecarga, velocidad de motor) en un sector de Flash interna del STM32F446RE, con verificación por checksum ante lecturas de sectores no inicializados.

### 3.3.7. Módulo RFID (RC522)
Implementa la inicialización del chip MFRC522 y la lectura de UID mediante REQA \+ anticolisión de nivel 1 sobre SPI1, comparando contra una lista de UIDs autorizados.

### 3.3.8. Módulo Bluetooth (HM-10)
Recepción de comandos por USART3 mediante interrupción (no polling bloqueante), permitiendo el llamado de piso remoto desde una aplicación de celular.

\[COMPLETAR: fragmentos de código relevantes (capturas) de los módulos que consideren más representativos, similar a la Figura 3.6 del ejemplo de referencia\]

# Capítulo 4: Ensayos y resultados
## 4.1. Prueba de integración (video)
\[COMPLETAR: link al video breve mostrando el ascensor funcionando — llamado de piso, viaje, apertura/cierre de puerta, emergencia, sobrecarga\]

## 4.2. Salida de consola y Build Analyzer
\[COMPLETAR: capturas de la salida de compilación (Console) y del Build Analyzer, con el detalle de tamaño de secciones (text, data, bss en bytes) y regiones (RAM, FLASH en bytes y %)\]

## 4.3. Medición y análisis de tiempos de ejecución (WCET)
\[COMPLETAR: metodología de medición (por ejemplo, DWT Cycle Counter) y tabla con el WCET medido de cada tarea del super-loop: Escrutar\_Actualizar, BLE\_Actualizar, FsmAscensor\_Tick, Buzzer\_Actualizar, etc.\]

| Tarea | WCET medido (µs) |
| ----- | ----- |
| Escrutar\_Actualizar | \[COMPLETAR\] |
| BLE\_Actualizar | \[COMPLETAR\] |
| FsmAscensor\_Tick | \[COMPLETAR\] |
| Buzzer\_Actualizar | \[COMPLETAR\] |

*Tabla 4.1: WCET por tarea.*

## 4.4. Cálculo del Factor de Uso (U) de la CPU
\[COMPLETAR: U \= (suma de WCET de todas las tareas) / (período del tick, 1ms), expresado en porcentaje, con el análisis correspondiente sobre el margen disponible\]

## 4.5. Medición y análisis de consumo
\[COMPLETAR: mediciones de corriente en los pines de 3,3V y 5V, con miliamperímetro y osciloscopio, en al menos dos condiciones: consumo normal y consumo en bajo consumo/reposo, siguiendo el formato de tabla de la Tabla 4.5 del ejemplo de referencia\]

## 4.6. Cumplimiento de requisitos
| Estado | Descripción |
| :---: | ----- |
| 🟢 | Implementado |
| 🟡 | Parcialmente implementado |
| 🔴 | No implementado |

*Tabla 4.2: Descripción de los íconos de estado.*

| Grupo | ID | Descripción | Estado |
| ----- | ----- | ----- | ----- |
| Movimiento | 1.1 | 3 paradas (PB, P1, P2) | \[COMPLETAR\] |
|  | 1.2 | Detección de llegada por reed switch | \[COMPLETAR\] |
|  | 1.3 | Control de velocidad por PWM | \[COMPLETAR\] |
|  | 1.4 | Watchdog de viaje (timeout → FALLA) | \[COMPLETAR\] |
| Llamado de piso | 2.1 | Botonera externa e interna | \[COMPLETAR\] |
|  | 2.2 | Cola de pedidos pendientes | \[COMPLETAR\] |
| Puerta | 3.1 \- 3.3 | Apertura/cierre automático con sensor | \[COMPLETAR\] |
| Seguridad | 4.1 \- 4.3 | Emergencia, sobrecarga, buzzer | \[COMPLETAR\] |
| Control de acceso | 5.1 \- 5.2 | Lectura y validación de UID (RC522) | \[COMPLETAR\] |
| Interfaz de usuario | 6.1 \- 6.3 | LCD, LEDs, Bluetooth | \[COMPLETAR\] |
| Configuración | 7.1 \- 7.3 | Menú SET\_UP \+ persistencia en Flash | \[COMPLETAR\] |
| Arquitectura | 8.1 \- 8.4 | Bare metal, super-loop \<1ms, tick 1ms, no bloqueante | \[COMPLETAR\] |

*Tabla 4.3: Cumplimiento de requisitos.*

# Conclusiones
## 5.1. Resultados obtenidos
\[COMPLETAR: reflexión honesta sobre qué se logró, qué dificultades surgieron (por ejemplo, temporización del HX711, protocolo del RC522 sin librería, calibración mecánica de la cabina) y cómo se resolvieron\]

## 5.2. Próximos pasos
\[COMPLETAR: por ejemplo, implementar modo de bajo consumo real con HAL\_PWR\_EnterSLEEPMode(), agregar más pisos, mejorar la app de Bluetooth, o sumar autenticación de sectores en el RC522 en vez de solo UID\]

# Capítulo 6: Uso de herramientas de IA
Se documenta a continuación el uso de herramientas de Inteligencia Artificial durante el desarrollo del proyecto.

Se utilizó IA como apoyo en las siguientes tareas:

* **Corrección de errores en los informes:** revisión y corrección de redacción, ortografía y estructura de la memoria técnica.  
* **Programación STM32:** apoyo en la estructura y funcionamiento del firmware (módulos, máquina de estados y ajustes puntuales de código).  
* **Depuración y resolución de problemas:** consultas sobre errores de compilación, comportamientos inesperados del microcontrolador y posibles inconsistencias no detectadas a simple vista en el código.

El uso de estas herramientas permitió resolver de forma ágil dudas del día a día, concentrando el esfuerzo del equipo en la lógica del sistema y la integración del prototipo.
# Capítulo 7: Bibliografía
\[1\] STM32F446RE Reference Manual. \[Online\]. Available: [https://www.st.com/resource/en/reference\_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf](https://www.st.com/resource/en/reference_manual/rm0390-stm32f446xx-advanced-armbased-32bit-mcus-stmicroelectronics.pdf)

\[2\] L298N Datasheet. \[Online\]. Available: [https://www.st.com/resource/en/datasheet/l298.pdf](https://www.st.com/resource/en/datasheet/l298.pdf)

\[3\] MFRC522 Datasheet. \[Online\]. Available: \[COMPLETAR\]

\[4\] HX711 Datasheet. \[Online\]. Available: \[COMPLETAR\]

\[5\] A Beginner's Guide to Designing Embedded System Applications on Arm Cortex-M Microcontroller. \[Online\]. Available: [https://www.arm.com/resources/education/books/designing-embedded-systems](https://www.arm.com/resources/education/books/designing-embedded-systems)

\[6\] Campus Grado FIUBA \- TA134. \[Online\]. Available: [https://campusgrado.fi.uba.ar/course/view.php?id=1217](https://campusgrado.fi.uba.ar/course/view.php?id=1217)

\[7\] Repositorio del proyecto: [https://github.com/Mardosreis/tdse-tf\_1erC\_1-03](https://github.com/Mardosreis/tdse-tf_1erC_1-03)

\[8\] Fundino elevador: [https://funduinoshop.com/es/educacion/funduino/ascensor-funduino/funduino-elevator-ascensor/elevador-para-arduino?srsltid=AfmBOopy4-ADLDJnJeEbezHpD-sw17suN0-LHIAc42m35cEGP\_zn\_UFD](https://funduinoshop.com/es/educacion/funduino/ascensor-funduino/funduino-elevator-ascensor/elevador-para-arduino?srsltid=AfmBOopy4-ADLDJnJeEbezHpD-sw17suN0-LHIAc42m35cEGP_zn_UFD)

\[9\] Ascensor Encoder: [https://ecopechperu.com/producto/ascensor-03-pisos-con-encoder/](https://ecopechperu.com/producto/ascensor-03-pisos-con-encoder/)

[image1]: ./images/image1.png

[image2]: ./images/image2.png

[image3]: ./images/image3.png

[image4]: ./images/image4.png

[image5]: ./images/image5.png

[image6]: ./images/image6.png

[image7]: ./images/image7.png

[image8]: ./images/image8.png

[image9]: ./images/image9.png

[image10]: ./images/image10.png

[image11]: ./images/image11.png