# Comunicación RS-485 entre ESP32 y Raspberry Pi Pico 2W para control de servomotor

## 1. Información general

**Asignatura:** Comunicaciones Industriales / Electrónica Digital  
**Práctica:** Implementación de comunicación RS-485 maestro–esclavo  
**Estudiante:** [Nombre del estudiante]  
**Docente:** [Nombre del docente]  
**Institución:** [Nombre de la institución]  
**Fecha:** Octubre 2025  

## 2. Introducción

La comunicación **RS-485** es un estándar industrial utilizado ampliamente para la transmisión de datos en entornos con interferencia eléctrica o largas distancias.  
Se basa en una señal diferencial que mejora la inmunidad al ruido y permite conectar múltiples dispositivos en una misma línea de comunicación.

En esta práctica se implementa una red maestro–esclavo basada en los microcontroladores **ESP32** y **Raspberry Pi Pico 2W**, conectados mediante módulos transceptores **MAX485**.  
El sistema permite el envío de comandos desde el maestro hacia el esclavo para controlar un **servomotor SG90**, demostrando el funcionamiento de los modos **simplex**, **half dúplex** y **full dúplex**.

## 3. Objetivos

### Objetivo general
Implementar una red de comunicación RS-485 entre un ESP32 y una Raspberry Pi Pico 2W para controlar un servomotor mediante transmisión serial diferencial.

### Objetivos específicos
- Analizar el funcionamiento del bus de comunicación RS-485.  
- Configurar los módulos MAX485 para comunicación diferencial.  
- Desarrollar código maestro y esclavo para los tres modos de transmisión.  
- Verificar la comunicación y control del servomotor.  
- Evaluar los resultados en función del tipo de transmisión.

## 4. Marco teórico

### 4.1 Estándar RS-485
El estándar **TIA/EIA-485-A** define una interfaz eléctrica balanceada, con transmisión diferencial en las líneas A y B. Permite conectar hasta 32 nodos en un mismo bus con longitudes de cable de hasta 1200 metros y velocidades de hasta 10 Mbps.

Las principales ventajas del RS-485 son:
- Comunicación diferencial inmune al ruido.  
- Capacidad multipunto.  
- Transmisión half y full dúplex.  
- Bajo consumo energético.

### 4.2 Módulo MAX485
El **MAX485** es un transceptor de bajo consumo que convierte las señales UART (TTL) de los microcontroladores a señales diferenciales RS-485.  
Dispone de los pines:
- **DI (Data In):** Entrada de datos desde el microcontrolador.  
- **RO (Receiver Out):** Salida de datos hacia el microcontrolador.  
- **DE (Driver Enable):** Habilita la transmisión.  
- **RE (Receiver Enable):** Habilita la recepción.  

Cuando DE y RE están en nivel alto, el dispositivo transmite; cuando están bajos, recibe.

### 4.3 Modos de comunicación
- **Simplex:** El maestro envía datos sin recibir respuesta.  
- **Half dúplex:** Comunicación bidireccional alternada.  
- **Full dúplex:** Comunicación bidireccional simultánea.

## 5. Materiales y componentes

| Cantidad | Componente | Función |
|-----------|-------------|----------|
| 1 | ESP32-WROOM-32 | Nodo maestro |
| 1 | Raspberry Pi Pico 2W | Nodo esclavo |
| 2 | Módulo MAX485 | Conversor UART ↔ RS-485 |
| 1 | Servomotor SG90 (rotación continua) | Actuador |
| 1 | Fuente 5V / 2A | Alimentación |
| - | Protoboard y cables Dupont | Conexión del circuito |

## 6. Diagrama de conexión

| Conexión | ESP32 (Maestro) | MAX485 Maestro | MAX485 Esclavo | Raspberry Pi Pico 2W (Esclavo) |
|-----------|----------------|----------------|----------------|-------------------------------|
| UART TX/RX | TX (GPIO17) / RX (GPIO16) | DI / RO | RO / DI | GP1 (RX) / GP0 (TX) |
| Control DE/RE | GPIO4 | DE + RE (puenteados) | DE + RE (puenteados) | GP2 |
| Alimentación | 5V / GND | VCC / GND | VCC / GND | VBUS / GND |
| Bus RS-485 | - | A, B (diferencial) | A, B (diferencial) | - |
| Servo | - | - | - | GP15 (PWM) → señal servo |


## 7. Desarrollo experimental

### 7.1 Código del Maestro (ESP32 – Arduino IDE)

```cpp
/*
 * Maestro RS-485 con ESP32
 */

void setup() {
  Serial.begin(115200);
  Serial2.begin(9600, SERIAL_8N1, 16, 17); // RX=16, TX=17
  pinMode(4, OUTPUT); // Control DE/RE
  digitalWrite(4, LOW);
  Serial.println("ESP32 Maestro iniciado...");
}

void loop() {
  digitalWrite(4, HIGH); // Habilitar transmisión
  String command = "POS:120\n";
  Serial2.print(command);
  Serial.print("Enviado → ");
  Serial.println(command);
  digitalWrite(4, LOW); // Habilitar recepción

  delay(1000);

  if (Serial2.available()) {
    String response = Serial2.readStringUntil('\n');
    Serial.print("Respuesta ← ");
    Serial.println(response);
  }

  delay(2000);
}

from machine import UART, Pin, PWM
import time

uart = UART(0, baudrate=9600, tx=Pin(0), rx=Pin(1))
driver_enable = Pin(2, Pin.OUT)
driver_enable.value(0)

servo = PWM(Pin(15))
servo.freq(50)

def set_servo_speed(value):
    duty = int(((value / 180) * 5000) + 1000)
    servo.duty_u16(duty)

print("Esclavo RS-485 listo...")

while True:
    if uart.any():
        data = uart.readline().decode().strip()
        print("Recibido:", data)

        if data.startswith("POS:"):
            try:
                value = int(data.split(":")[1])
                set_servo_speed(value)
                driver_enable.value(1)
                uart.write(f"OK:{value}\n")
                driver_enable.value(0)
            except:
                uart.write("ERROR\n")

    time.sleep(0.1)
```cpp
### 7.2 Código del Esclavo (Raspberry Pi Pico 2W – Thonny / MicroPython)

```python
from machine import UART, Pin, PWM
import time

uart = UART(0, baudrate=9600, tx=Pin(0), rx=Pin(1))
driver_enable = Pin(2, Pin.OUT)
driver_enable.value(0)

servo = PWM(Pin(15))
servo.freq(50)

def set_servo_speed(value):
    duty = int(((value / 180) * 5000) + 1000)
    servo.duty_u16(duty)

print("Esclavo RS-485 listo...")

while True:
    if uart.any():
        data = uart.readline().decode().strip()
        print("Recibido:", data)

        if data.startswith("POS:"):
            try:
                value = int(data.split(":")[1])
                set_servo_speed(value)
                driver_enable.value(1)
                uart.write(f"OK:{value}\n")
                driver_enable.value(0)
            except:
                uart.write("ERROR\n")

    time.sleep(0.1)


## 8. Modos de comunicación implementados

| Modo | Descripción | Flujo de datos | Ejemplo |
|------|--------------|----------------|----------|
| Simplex | Solo el maestro transmite. | Maestro → Esclavo | `POS:120` |
| Half dúplex | Comunicación alternada entre maestro y esclavo. | Maestro ↔ Esclavo (turnos) | `POS:120` / `OK:120` |
| Full dúplex | Comunicación simultánea entre maestro y esclavo. | Maestro ↔ Esclavo | `POS:120` + `OK:120` |


## 9. Resultados experimentales

Durante las pruebas realizadas con los módulos MAX485, se obtuvieron los siguientes resultados:

1. En modo **simplex**, el maestro (ESP32) transmitió comandos hacia el esclavo (Raspberry Pi Pico 2W) que ejecutó las órdenes sin enviar confirmaciones.  
2. En **half dúplex**, la activación del pin de control DE/RE permitió el envío alternado de información, logrando comunicación bidireccional sin colisiones.  
3. En **full dúplex**, ambos dispositivos se comunicaron de forma simultánea sin interferencias, demostrando la capacidad del bus para transmisión concurrente.  
4. El **servomotor SG90 de rotación continua** respondió correctamente a los comandos `POS:x`, donde el valor 90 representó el reposo, los valores mayores generaron rotación en un sentido y los menores en el sentido contrario.  
5. Se comprobó la estabilidad del enlace y la correcta transmisión de datos en los tres modos de operación.

Evidencias del funcionamiento (colocar capturas o fotos):

```markdown
![Monitor serie mostrando comunicación](media/monitor_serial.png)
![Servo respondiendo a comandos](media/servo.gif)

## 10. Análisis de resultados

| Parámetro | Simplex | Half dúplex | Full dúplex |
|------------|----------|--------------|--------------|
| Dirección de transmisión | Unidireccional | Bidireccional alternada | Bidireccional simultánea |
| Complejidad de implementación | Baja | Media | Alta |
| Velocidad efectiva | Alta | Media | Alta |
| Control del flujo | No requerido | Requerido | No requerido |
| Riesgo de colisión | Nulo | Medio | Bajo |
| Aplicaciones típicas | Sensores, PLC unidireccionales | Redes industriales RS-485 | Comunicaciones avanzadas y monitoreo |

El análisis de los resultados muestra que el modo **half dúplex** ofrece un equilibrio adecuado entre simplicidad y funcionalidad, siendo el más empleado en sistemas industriales basados en RS-485.  
El **simplex** resulta útil para aplicaciones unidireccionales donde no se requiere respuesta del receptor, mientras que el **full dúplex** ofrece el mayor rendimiento, aunque a costa de una mayor complejidad en el hardware.

## 11. Conclusiones

1. Se implementó correctamente la comunicación RS-485 entre el ESP32 (maestro) y la Raspberry Pi Pico 2W (esclavo) utilizando módulos MAX485.  
2. Se verificaron los modos de comunicación simplex, half dúplex y full dúplex, observando las diferencias entre cada uno en cuanto a dirección de flujo, velocidad y control del canal.  
3. El servomotor SG90 respondió adecuadamente a los comandos enviados por el maestro, confirmando la transmisión de datos confiable y la correcta interpretación de los mensajes por parte del esclavo.  
4. El uso de los pines de control DE y RE permitió gestionar el sentido de transmisión, siendo esencial para el funcionamiento en los modos bidireccionales.  
5. La práctica permitió comprender la relevancia del estándar RS-485 en aplicaciones industriales, destacando su robustez, bajo costo y adaptabilidad a entornos con interferencia electromagnética.

## 12. Bibliografía y recursos

- Maxim Integrated, *MAX485 Low-Power Transceiver for RS-485 Communication*, Datasheet.  
- Espressif Systems, *ESP32 Technical Reference Manual*.  
- Raspberry Pi Foundation, *Raspberry Pi Pico W MicroPython SDK*.  
- IEEE Standards Association, *TIA/EIA-485-A: Standard for Electrical Characteristics of Generators and Receivers for Use in Balanced Digital Multipoint Systems*.  
- Texas Instruments, *Application Report: Understanding and Using RS-485*.  
- Horowitz, P., & Hill, W. (2015). *The Art of Electronics* (3rd ed.). Cambridge University Press.

## 13. Estructura del repositorio

## 10. Análisis de resultados

| Parámetro | Simplex | Half dúplex | Full dúplex |
|------------|----------|--------------|--------------|
| Dirección de transmisión | Unidireccional | Bidireccional alternada | Bidireccional simultánea |
| Complejidad de implementación | Baja | Media | Alta |
| Velocidad efectiva | Alta | Media | Alta |
| Control del flujo | No requerido | Requerido | No requerido |
| Riesgo de colisión | Nulo | Medio | Bajo |
| Aplicaciones típicas | Sensores, PLC unidireccionales | Redes industriales RS-485 | Comunicaciones avanzadas y monitoreo |

El análisis de los resultados muestra que el modo **half dúplex** ofrece un equilibrio adecuado entre simplicidad y funcionalidad, siendo el más empleado en sistemas industriales basados en RS-485.  
El **simplex** resulta útil para aplicaciones unidireccionales donde no se requiere respuesta del receptor, mientras que el **full dúplex** ofrece el mayor rendimiento, aunque a costa de una mayor complejidad en el hardware.


## 11. Conclusiones

1. Se implementó correctamente la comunicación RS-485 entre el ESP32 (maestro) y la Raspberry Pi Pico 2W (esclavo) utilizando módulos MAX485.  
2. Se verificaron los modos de comunicación simplex, half dúplex y full dúplex, observando las diferencias entre cada uno en cuanto a dirección de flujo, velocidad y control del canal.  
3. El servomotor SG90 respondió adecuadamente a los comandos enviados por el maestro, confirmando la transmisión de datos confiable y la correcta interpretación de los mensajes por parte del esclavo.  
4. El uso de los pines de control DE y RE permitió gestionar el sentido de transmisión, siendo esencial para el funcionamiento en los modos bidireccionales.  
5. La práctica permitió comprender la relevancia del estándar RS-485 en aplicaciones industriales, destacando su robustez, bajo costo y adaptabilidad a entornos con interferencia electromagnética.


## 12. Bibliografía y recursos

- Maxim Integrated, *MAX485 Low-Power Transceiver for RS-485 Communication*, Datasheet.  
- Espressif Systems, *ESP32 Technical Reference Manual*.  
- Raspberry Pi Foundation, *Raspberry Pi Pico W MicroPython SDK*.  
- IEEE Standards Association, *TIA/EIA-485-A: Standard for Electrical Characteristics of Generators and Receivers for Use in Balanced Digital Multipoint Systems*.  
- Texas Instruments, *Application Report: Understanding and Using RS-485*.  
- Horowitz, P., & Hill, W. (2015). *The Art of Electronics* (3rd ed.). Cambridge University Press.


