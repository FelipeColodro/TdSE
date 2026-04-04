Estados del LED
OFF: LED apagado
ON: LED encendido
BLINK: LED titilando periódicamente
PULSE: LED encendido por un tiempo determinado
N_PULSES: LED emite una cantidad finita de pulsos


Eventos del LED
EV_LED_ON: encender el LED
EV_LED_OFF: apagar el LED
EV_LED_BLINK_START: iniciar titilado
EV_LED_BLINK_STOP: detener titilado
EV_LED_PULSE: generar un pulso
EV_LED_N_PULSES: generar N pulsos

Acciones del LED
LED_Set(ON): pone el LED en estado encendido
LED_Set(OFF): pone el LED en estado apagado
Inicialización/modificación de variables de control:
timer: contador de tiempo en ms
period: período de titilado
pulse_count: cantidad de pulsos restantes

Uso de Timer (tick = 1 ms)
En cada ciclo (1 ms):
timer++

Comportamiento (modelo lógico):
1. Estado ON
Acción: LED_Set(ON)
Se mantiene hasta recibir EV_LED_OFF
2. Estado OFF
Acción: LED_Set(OFF)
Estado inicial del sistema
3. Estado BLINK
Si timer >= period:
Toggle del LED (ON ↔ OFF)
Reset de timer
4. Estado PULSE
Al recibir EV_LED_PULSE:
LED ON
timer = 0
Si timer >= T_pulse:
LED OFF
Fin del estado
5. Estado N_PULSES
Al recibir EV_LED_N_PULSES:
Inicializar pulse_count = N
Mientras pulse_count > 0:
Generar pulsos usando timer
Decrementar pulse_count en cada ciclo completo
Cuando pulse_count == 0:
LED OFF


Tabla de Estados y Excitaciones del modelo Actuator
(Actuator Statechart - State Transition Table)

| Current State | Event | Guard | Next State | Actions |
|--------------|------|-------|------------|---------|
| OFF | EV_LED_ON | - | ON | LED_Set(ON) |
| OFF | EV_LED_BLINK_START | - | BLINK | timer = 0 |
| OFF | EV_LED_PULSE | - | PULSE | LED_Set(ON), timer = 0 |
| OFF | EV_LED_N_PULSES | - | N_PULSES | pulse_count = N, timer = 0 |
| ON | EV_LED_OFF | - | OFF | LED_Set(OFF) |
| ON | EV_LED_BLINK_START | - | BLINK | timer = 0 |
| ON | EV_LED_PULSE | - | PULSE | timer = 0 |
| BLINK | EV_LED_BLINK_STOP | - | OFF | LED_Set(OFF) |
| BLINK | TICK | timer >= period | BLINK | Toggle LED, timer = 0 |
| PULSE | TICK | timer >= T_pulse | OFF | LED_Set(OFF) |
| PULSE | TICK | timer < T_pulse | PULSE | timer++ |
| N_PULSES | TICK | timer >= period | N_PULSES | Toggle LED, timer = 0, pulse_count-- |
| N_PULSES | TICK | pulse_count == 0 | OFF | LED_Set(OFF) |
