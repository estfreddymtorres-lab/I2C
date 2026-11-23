# I2C
---

## 📌 Proyecto 4: Reconocimiento y Decodificación de Tramas I2C para Control de Pantalla OLED

Este proyecto consiste en la captura, inspección temporal y decodificación hexadecimal del protocolo de comunicación síncrono **I2C** utilizando una **Raspberry Pi Pico** como dispositivo maestro y una pantalla gráfica **OLED SSD1306** como esclavo, analizando las transacciones en el bus mediante un analizador lógico con **Logic 2**[cite: 14, 15, 16].

### 📝 Descripción
El objetivo es identificar visual y experimentalmente la anatomía de una trama I2C en líneas *Open-Drain* (`SDA` y `SCL`), abarcando las condiciones de **Start**, direccionamiento de 7 bits + bit `R/W`, respuestas de reconocimiento (**ACK/NACK**) en el noveno pulso de reloj y condición de **Stop**[cite: 14, 15, 16]. Además, se valida el escaneo automático del bus (`i2c.scan()`) para localizar el periférico en `0x3C` y se decodifican las tramas de configuración y escritura en memoria gráfica (**GDDRAM**) según el *datasheet* del controlador SSD1306[cite: 14, 16, 18].

### 🛠️ Tecnologías y Hardware
* **Microcontrolador Maestro:** Raspberry Pi Pico W programada en **MicroPython** (entorno Thonny), usando el bus I2C en `GP14` (`SDA`) y `GP15` (`SCL`) a una frecuencia configurada de **100 kHz** (frecuencia real medida ~83–84 kHz)[cite: 16, 17].
* **Periférico Esclavo:** Pantalla OLED SSD1306 (0.96", $128 \times 64$ píxeles) con dirección física **`0x3C`**[cite: 14, 16, 19].
* **Instrumentación:** Analizador lógico configurado a **1 MS/s** ($\ge 10\times$ frecuencia del bus) y software **Saleae Logic 2**[cite: 16].

### 🚀 Metodología y Pruebas Realizadas
1. **Validación ACK vs. NACK:** Envío de tramas de escritura hacia la dirección válida `0x3C` (donde el esclavo fuerza `SDA` a `0` en el 9.º pulso generando **ACK**) y hacia `0x3D` (donde las resistencias *pull-up* mantienen `SDA` en `1` generando **NACK**)[cite: 16, 17].
2. **Escaneo Automático del Bus:** Ejecución de `i2c.scan()` barriendo direcciones de `0x08` a `0x77` para verificar la detección física de `0x3C`[cite: 16, 18].
3. **Decodificación de Comandos y Datos OLED:** Captura y contraste contra el *datasheet* de instrucciones precedidas por el *Control Byte* de comando (`0x80`) —encendido (`0xAF`), apagado (`0xAE`), contraste de doble byte (`0x81 + 0xC8`), modo normal (`0xA6`) e invertido (`0xA7`)— y escritura masiva de píxeles en GDDRAM precedida por el *Control Byte* de datos (`0x40`)[cite: 16, 18, 19].

### 📊 Conclusiones Principales
* **Comportamiento del Noveno Pulso (ACK/NACK):** Se verificó que un **NACK** no representa una falla o bloqueo del bus I2C, sino el estado lógico alto esperado por acción de las resistencias *pull-up* cuando ningún dispositivo reclama la dirección consultada, permitiendo mapear topologías mediante escaneo[cite: 17, 19].
* **Abstracción entre Comandos y Píxeles:** El protocolo I2C transporta octetos sin distinguir su propósito; es el **Control Byte** del controlador SSD1306 (`0x80` frente a `0x40`) el que conmuta internamente entre registros de configuración y escritura en memoria de video (GDDRAM)[cite: 16, 18, 19].
