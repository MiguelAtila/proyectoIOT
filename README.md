# Proyecto IoT - Monitoreo con MQTT y Serial

Este repositorio contiene un sistema de monitoreo basado en sensores conectados vía Arduino (o simulador), usando MQTT como medio de transporte de datos y un MTU en Node.js para gestionar la publicación. Un subscriber en Python permite monitorear en tiempo real por sedes, pisos y sensores.

---

## 1. Introducción General

- **Objetivo**: Transmitir datos desde sensores (reales o simulados) a través de un servidor MQTT, con fallback local ante desconexiones.
- **Componentes**:
  - Arduino o simulador (SimulIDE o Python)
  - MTU (Node.js) con conexión serial y publicación MQTT
  - Broker Mosquitto con autenticación
  - Subscriber (Python) para monitoreo interactivo

---

## 2. Instalación y Configuración Inicial

### 2.1 Requisitos

- Node.js y npm
- Python 3.x (`paho-mqtt`)
- Mosquitto MQTT
- `socat` (para puertos virtuales)
- Dependencias:
  ```bash
  npm install serialport mqtt dotenv
  pip install paho-mqtt
  ```

### 2.2 Creación de Puertos Virtuales

```bash
socat -d -d PTY,link=/dev/ttyVirtual1,raw,echo=0 PTY,link=/dev/ttyVirtual2,raw,echo=0
```

Verifica las rutas con:
```bash
ls -l /dev/ttyVirtual*
```

### 2.3 Configuración del archivo `.env`

```dotenv
SEDE=amerikeCDMX
PISO=P1
SERIAL_PORT=/dev/ttyVirtual2
MQTT_HOST=192.168.X.X
MQTT_PORT=1883
MQTT_USER=mtuuser
MQTT_PASS=amerike
```

---

## 3. MTU - index.js

- Lee datos del puerto serial
- Extrae tipo de sensor
- Construye topic dinámicamente: `SEDE/PISO/sensor`
- Publica a MQTT
- Si se pierde conexión, guarda localmente en `logs/`

---

## 4. Broker Mosquitto

- Asegura configuración con autenticación:
  - `mosquitto_passwd -c /etc/mosquitto/passwd mtuuser`
  - Modifica `/etc/mosquitto/mosquitto.conf`:
    ```
    allow_anonymous false
    password_file /etc/mosquitto/passwd
    listener 1883
    ```

---

## 5. Subscriber (Python)

- Ejecuta con:
  ```bash
  python3 subscriber.py
  ```
- Menú para seleccionar sede, piso y sensor
- Se conecta al topic correspondiente y muestra los mensajes

---

## 6. Checklist de Validación Técnica

| Ítem | Cumple | Observaciones |
|------|--------|----------------|
| `.env` configurado correctamente | ✅ / ❌ | ... |
| Mosquitto funcionando con autenticación | ✅ / ❌ | ... |
| Comunicación Serial en MTU activa | ✅ / ❌ | ... |
| Publicación correcta de topics MQTT | ✅ / ❌ | ... |
| Guardado en archivo cuando no hay red | ✅ / ❌ | ... |
| Subscriber recibe datos al seleccionar topic correcto | ✅ / ❌ | ... |
| Diferenciación de sede y piso funciona | ✅ / ❌ | ... |

---

## 7. Conclusiones

Este sistema demuestra cómo se puede integrar hardware y software con protocolos de comunicación modernos para crear una arquitectura IoT robusta. Su diseño permite extenderlo fácilmente hacia bases de datos, dashboards o alertas.
