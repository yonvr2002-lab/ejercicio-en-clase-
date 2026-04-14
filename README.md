# 📡 Ejercicio en Clase - Análisis de Tráfico de Red con YOLOv8

## 🧠 Objetivo

Analizar el tráfico de red generado por el modelo YOLOv8 y comparar el comportamiento de los protocolos **TCP** y **UDP** en dos escenarios:

* Descarga del modelo (TCP)
* Transmisión de video en tiempo real (UDP)

---

## ⚙️ Preparación del entorno (Google Colab)

1. Abrir Google Colab
2. Crear un nuevo notebook
3. Activar GPU:

   * Ir a `Entorno de ejecución > Cambiar tipo de entorno de ejecución`
   * Seleccionar **GPU**

---

## 🧪 Fase 1: Análisis de descarga (TCP)

### Paso 1: Instalar herramientas

```bash
!pip install ultralytics
!apt-get install tcpdump
```

### Paso 2: Iniciar captura de paquetes

```bash
!tcpdump -i eth0 -s 65535 -w descarga_tcp.pcap &
```

### Paso 3: Generar tráfico (descargar modelo YOLOv8)

```python
from ultralytics import YOLO
model = YOLO("yolov8n.pt")
```

### Paso 4: Detener captura

```bash
!killall tcpdump
```
EJECUTADO

<img width="1732" height="790" alt="image" src="https://github.com/user-attachments/assets/9a442fdf-14d4-48ee-86ef-171cb78b99fc" />
<img width="900" height="395" alt="image" src="https://github.com/user-attachments/assets/6449b533-71c6-4197-8249-9d96c0ee2a6b" />


---

## 📹 Fase 2: Transmisión de video (UDP)

### Paso 1: Iniciar captura UDP

```bash
!tcpdump -i eth0 udp -w video_udp.pcap &
```

### Paso 2: Simular envío de video

```python
import socket
import time

sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
server_address = ('localhost', 12345)

for i in range(50):
    message = f'Frame {i}'
    sock.sendto(message.encode(), server_address)
    time.sleep(0.1)
```

### Paso 3: Detener captura

```bash
!killall tcpdump
```

---
EJECUTADO

<img width="1117" height="665" alt="image" src="https://github.com/user-attachments/assets/950dd803-b921-4927-8582-205d984a3c82" />

## 🔍 Fase 3: Análisis con Wireshark

### Archivo 1: descarga_tcp.pcap

<img width="1761" height="750" alt="image" src="https://github.com/user-attachments/assets/011659f2-e62a-4dfb-bcea-560ca7bd6ec6" />


Filtros útiles:

* `tcp.flags.syn == 1`
* `tcp.analysis.retransmission`
* `tcp.port == 443`

### Archivo 2: video_udp.pcap

<img width="1053" height="509" alt="image" src="https://github.com/user-attachments/assets/b653e422-e847-40de-9308-1b9361ab1674" />


Filtrar:

* `udp`
<img width="1365" height="728" alt="image" src="https://github.com/user-attachments/assets/e3a673eb-fca4-42f1-a778-0aa1f1fbf563" />


---

## ❓ Respuestas

### 1. ¿Qué es YOLO?

YOLO (You Only Look Once) es un modelo de visión por computadora que permite detectar objetos en imágenes y video en tiempo real. Se caracteriza por ser rápido, preciso y eficiente.

Su arquitectura se basa en redes neuronales convolucionales (CNN) que procesan la imagen completa en una sola pasada.

---

### 2. TCP vs UDP

La descarga del modelo usa **TCP** porque:

* Es confiable
* Garantiza entrega de datos
* Reenvía paquetes perdidos

La transmisión de video usa **UDP** porque:

* Es más rápido
* Tiene menor latencia
* No necesita confirmación de paquetes

---

### 3. Fiabilidad vs Velocidad

Si aparece `tcp.analysis.retransmission`, significa que:

* Se perdieron paquetes
* TCP los volvió a enviar

Esto es importante en descargas porque asegura que el archivo llegue completo.

En video en vivo sería malo porque:

* Genera retraso (lag)
* Afecta la fluidez

---

### 4. Identificar el origen del tráfico

Se pueden usar filtros como:

* `ip.src == X.X.X.X`
* `ip.dst == X.X.X.X`

Esto permite identificar el servidor que envía el modelo (por ejemplo, servidores de Ultralytics o Google Cloud).

---

## 📌 Conclusión

TCP es ideal para tareas donde la integridad de los datos es crítica (descargas), mientras que UDP es mejor para aplicaciones en tiempo real donde la velocidad es más importante que la precisión (video en vivo).

---
