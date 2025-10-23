# Comunicación RS-485 entre ESP32 y Raspberry Pi Pico 2W para control de servomotor

## 1. Información general

**Asignatura:** Comunicaciones Industriales / Electrónica Digital  
**Práctica:** Implementación de comunicación RS-485 maestro–esclavo  
**Estudiante:** [Nombre del estudiante]  
**Docente:** [Nombre del docente]  
**Institución:** [Nombre de la institución]  
**Fecha:** Octubre 2025  

---

## 2. Introducción

La comunicación **RS-485** es un estándar industrial utilizado ampliamente para la transmisión de datos en entornos con interferencia eléctrica o largas distancias.  
Se basa en una señal diferencial que mejora la inmunidad al ruido y permite conectar múltiples dispositivos en una misma línea de comunicación.

En esta práctica se implementa una red maestro–esclavo basada en los microcontroladores **ESP32** y **Raspberry Pi Pico 2W**, conectados mediante módulos transceptores **MAX485**.  
El sistema permite el envío de comandos desde el maestro hacia el esclavo para controlar un **servomotor SG90**, demostrando el funcionamiento de los modos **simplex**, **half dúplex** y **full dúplex**.

---

## 3. Objetivos

### Objetivo general
Implementar una red de comunicación RS-485 entre un ESP32 y una Raspberry Pi Pico 2W para controlar un servomotor mediante transmisión serial diferencial.

### Objetivos específicos
- Analizar el funcionamiento del bus de comunicación RS-485.  
- Configurar los módulos MAX485 para comunicación diferencial.  
- Desarrollar código maestro y esclavo para los tres modos de transmisión.  
- Verificar la comunicación y control del servomotor.  
- Evaluar los resultados en función del tipo de transmisión.

---

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

---

## 5. Materiales y componentes

| Cantidad | Componente | Función |
|-----------|-------------|----------|
| 1 | ESP32-WROOM-32 | Nodo maestro |
| 1 | Raspberry Pi Pico 2W | Nodo esclavo |
| 2 | Módulo MAX485 | Conversor UART ↔ RS-485 |
| 1 | Servomotor SG90 (rotación continua) | Actuador |
| 1 | Fuente 5V / 2A | Alimentación |
| - | Protoboard y cables Dupont | Conexión del circuito |

---

## 6. Diagrama de conexión

| Conexión | ESP32 (Maestro) | MAX485 Maestro | MAX485 Esclavo | Raspberry Pi Pico 2W (Esclavo) |
|-----------|----------------|----------------|----------------|-------------------------------|
| UART TX/RX | TX (GPIO17) / RX (GPIO16) | DI / RO | RO / DI | GP1 (RX) / GP0 (TX) |
| Control DE/RE | GPIO4 | DE + RE (puenteados) | DE + RE (puenteados) | GP2 |
| Alimentación | 5V / GND | VCC / GND | VCC / GND | VBUS / GND |
| Bus RS-485 | - | A, B (diferencial) | A, B (diferencial) | - |
| Servo | - | - | - | GP15 (PWM) → señal servo |

Imágenes de referencia del montaje:

```markdown
![Diagrama de conexión](media/diagrama.png)
![Montaje físico](media/montaje.jpg)


