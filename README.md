# ESP32 Hardware Timer Interrupt Example

Este proyecto muestra cómo utilizar un **timer hardware del ESP32** para generar interrupciones periódicas utilizando Arduino.

El programa configura un **temporizador que genera una interrupción cada segundo** y cuenta cuántas interrupciones han ocurrido.

---

## Funcionamiento

1. Se crea un **timer hardware** usando `timerBegin()`.

2. Se asocia una **rutina de interrupción (ISR)** al timer con `timerAttachInterrupt()`.

3. Se configura una **alarma del timer** para que se active cada **1 segundo (1,000,000 µs)** usando `timerAlarmWrite()`.

4. Cada vez que ocurre la interrupción:
   - Se ejecuta la función `onTimer()`
   - Se incrementa la variable `interruptCounter`

5. En el `loop()`:
   - Se detecta si ha ocurrido una interrupción
   - Se incrementa el contador total
   - Se muestra el resultado por el **Monitor Serie**

---

## Código

```cpp
#include <Arduino.h>

volatile int interruptCounter;
int totalInterruptCounter;

hw_timer_t * timer = NULL;
portMUX_TYPE timerMux = portMUX_INITIALIZER_UNLOCKED;

void IRAM_ATTR onTimer() {
 portENTER_CRITICAL_ISR(&timerMux);
 interruptCounter++;
 portEXIT_CRITICAL_ISR(&timerMux);
}

void setup() {
 Serial.begin(115200);

 timer = timerBegin(0, 80, true);
 timerAttachInterrupt(timer, &onTimer, true);
 timerAlarmWrite(timer, 1000000, true);
 timerAlarmEnable(timer);
}

void loop() {
 if (interruptCounter > 0) {

  portENTER_CRITICAL(&timerMux);
  interruptCounter--;
  portEXIT_CRITICAL(&timerMux);

  totalInterruptCounter++;

  Serial.print("An interrupt as occurred. Total number: ");
  Serial.println(totalInterruptCounter);
 }
}
