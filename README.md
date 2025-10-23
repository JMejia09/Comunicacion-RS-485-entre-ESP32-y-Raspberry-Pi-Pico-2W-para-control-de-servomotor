# 🧩 Comunicación RS-485 entre ESP32 y Raspberry Pi Pico 2W para control de servomotor

## 🏫 Informe técnico de laboratorio

**Curso:** Comunicaciones Industriales / Electrónica Digital  
**Práctica:** Implementación de comunicación RS-485 maestro–esclavo  
**Alumno:** [Tu nombre completo]  
**Docente:** [Nombre del profesor]  
**Institución:** [Nombre de la institución educativa]  
**Fecha:** Octubre 2025  

---

## 🧠 1. Introducción

La comunicación industrial **RS-485** es un estándar ampliamente utilizado para el intercambio de datos en entornos ruidosos o de larga distancia, gracias a su modo diferencial y capacidad multipunto.  

En este proyecto se implementa una red RS-485 con un **maestro (ESP32)** y un **esclavo (Raspberry Pi Pico 2W)**, simulando los tres modos de transmisión (simplex, half dúplex y full dúplex) para controlar un **servomotor SG90**.  

El objetivo es comprender los principios de comunicación diferencial, la gestión de turnos de transmisión mediante el pin de habilitación (DE/RE) del transceptor **MAX485**, y la respuesta del sistema actuador frente a los comandos enviados.

---

## 🎯 2. Objetivos

### Objetivo general
Implementar y analizar una comunicación RS-485 entre un microcontrolador maestro y un esclavo, aplicando los modos simplex, half dúplex y full dúplex, con control práctico de un servomotor.

### Objetivos específicos
- Conocer la estructura y funcionamiento del estándar RS-485.  
- Implementar la comunicación UART diferencial usando módulos MAX485.  
- Configurar un protocolo maestro–esclavo entre **ESP32** y **Raspberry Pi Pico 2W**.  
- Controlar un servomotor desde el maestro mediante comandos seriales.  
- Evaluar los resultados de transmisión en cada modo de comunicación.

---

## 🧩 3. Marco teórico

El estándar **RS-485** (TIA/EIA-485) define una interfaz eléctrica balanceada diferencial, permitiendo la comunicación a largas distancias (hasta 1200 m) y velocidades superiores a 10 Mbps.  

A diferencia de RS-232, RS-485 permite conectar múltiples dispositivos en un mismo bus (topología multipunto), siendo común en entornos industriales como **MODBUS RTU** o **Profibus**.

### Características principales:
- Comunicación diferencial (líneas A y B).  
- Hasta 32 nodos por bus.  
- Tolerancia a interferencias electromagnéticas.  
- Permite modos: simplex, half dúplex, full dúplex.

El chip **MAX485** convierte las señales UART (TTL) en señales RS-485 diferenciales y controla el flujo mediante los pines **DE (Driver Enable)** y **RE (Receiver Enable)**.  
Estos pines determinan si el dispositivo está transmitiendo o recibiendo, aspecto esencial para half dúplex.

---

## ⚙️ 4. Materiales y componentes

| Cantidad | Componente | Función |
|-----------|-------------|----------|
| 1 | ESP32-WROOM-32 | Nodo maestro, envía comandos |
| 1 | Raspberry Pi Pico 2W | Nodo esclavo, ejecuta control de servo |
| 2 | Módulos MAX485 | Conversores UART ↔ RS-485 |
| 1 | Servo motor SG90 (rotación continua) | Actuador |
| 1 | Fuente de 5 V / 2 A | Alimentación común |
| — | Protoboard y cables Dupont | Conexión de circuito |

---

## 🔌 5. Diagrama de conexión

### Descripción de conexiones

| Conexión | ESP32 (Maestro) | MAX485 Maestro | MAX485 Esclavo | Raspberry Pi Pico 2W (Esclavo) |
|-----------|----------------|----------------|----------------|-------------------------------|
| UART TX/RX | TX (GPIO17) / RX (GPIO16) | DI / RO | RO / DI | GP1 (RX) / GP0 (TX) |
| Control DE/RE | GPIO4 | DE + RE (puenteados) | DE + RE (puenteados) | GP2 |
| Alimentación | 5V / GND | VCC / GND | VCC / GND | VBUS / GND |
| Bus RS-485 | — | A, B (diferencial) | A, B (diferencial) | — |
| Servo | — | — | — | GP15 (PWM) → señal servo |

📸 Inserta aquí tus fotos y diagramas:

```markdown
![Diagrama de conexión](media/diagrama.png)
![Montaje físico](media/montaje.jpg)

