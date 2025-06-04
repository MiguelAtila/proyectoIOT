# Proyecto IoT con MQTT y MTU

Este proyecto simula un entorno IoT que conecta sensores físicos o simulados (Arduino o GUI en Python) con un servidor MQTT Mosquitto, mediante un módulo MTU escrito en Node.js. Incluye un sistema de respaldo en caso de desconexión y un subscriber en Python que permite seleccionar topics específicos por sede y piso.

---

## Estructura del Proyecto

```plaintext
PROYECTODEMODAY/
├── arduinoSens/
│   └── sensores_MTU.ino                 # Código Arduino (físico)
├── nodeMQTT/
│   ├── index.js                         # Código principal del MTU en Node.js
│   ├── .env                             # Configuración del entorno (SEDE, PISO, SERIAL_PORT)
│   ├── logs/                            # Carpeta donde se almacenan logs offline
│   ├── package.json                     # Dependencias y metadata
│   └── package-lock.json
├── pythonMTU/
│   ├── subscriber.py                    # Subscriber con selección de topic
│   ├── subscriberGrl.py                # Subscriber general (todos los topics)
│   ├── publisherPruebas.py              # Publisher de prueba (modo local/simulador)
│   └── logs/                            # Logs para respaldo del simulador
├── simuladorArduino/
│   └── simuladorGUI.py                  # Simulador gráfico en Tkinter (envía datos por serial)
├── procedimiento.txt                    # Guía paso a paso (formato de texto plano)
└── README.md                            # Actual archivo
```

---

## Requisitos

- Node.js y `npm`
- Python 3.10+
- `mosquitto` (servidor MQTT)
- Dependencias:
  - Node.js: `npm install mqtt serialport dotenv`
  - Python: `pip install paho-mqtt pyserial`

---

## Configuración de `.env`

Ubicado en `nodeMQTT/.env`:

```env
SEDE=amerikeCDMX
PISO=P1
SERIAL_PORT=/dev/pts/3
```

- **SEDE**: amerikeCDMX o amerikeGDJ
- **PISO**: PB, P1 o P2
- **SERIAL_PORT**: ruta del puerto virtual creado por `socat`

---

## Ejecución Paso a Paso

### 1. Crear puertos virtuales con `socat`

```bash
sudo socat -d -d pty,raw,echo=0 pty,raw,echo=0
# Anota los /dev/pts/X y /dev/pts/Y que se muestran
```

### 2. Modificar `.env` con la ruta del puerto

```env
SERIAL_PORT=/dev/pts/X  # Cambia X por el valor mostrado por socat
```

### 3. Verificar IP del servidor (en cada archivo):

- `nodeMQTT/index.js`
- `pythonMTU/subscriber.py`
- `pythonMTU/publisherPruebas.py` (si lo usas)

Busca líneas como:

```js
host: '192.168.3.52'  // Cambia por la IP de tu servidor MQTT
```

### 4. Ejecutar servidor Mosquitto

En la máquina servidor:

```bash
mosquitto -c /etc/mosquitto/mosquitto.conf
```

Asegúrate de que tenga autenticación habilitada.

---

## Ejecución por módulo

### Opción A: Simulación con GUI (sin Arduino físico)

1. Ejecuta el simulador:

```bash
python3 simuladorArduino/simuladorGUI.py
```

2. Ejecuta el MTU (lector del puerto serial):

```bash
cd nodeMQTT
node index.js
```

3. Ejecuta el subscriber (selección por topic):

```bash
cd pythonMTU
python3 subscriber.py
```

---

## Lista de Topics por Sede y Piso

| Topic MQTT                         | Descripción                             |
|-----------------------------------|-----------------------------------------|
| amerikeCDMX/PB/temp               | Temperatura sede CDMX PB                |
| amerikeCDMX/PB/hum                | Humedad sede CDMX PB                    |
| amerikeCDMX/PB/rfid               | RFID autorizado CDMX PB                 |
| amerikeCDMX/PB/rfid/denegado     | RFID denegado CDMX PB                   |
| amerikeCDMX/PB/otros             | Otros sensores sede CDMX PB             |
| ...                               | (Misma estructura para P1, P2, GDJ...)  |

---

## Checklist Validable por Componente

| Categoría               | Verificación                                               | Observaciones Docentes                    |
|------------------------|------------------------------------------------------------|--------------------------------------------|
| MQTT Server            | Mosquitto activo, con auth y accesible desde red           |                                            |
| MTU (Node.js)          | Lee datos del serial, publica en topic correcto            |                                            |
| .env                   | SEDE, PISO y SERIAL_PORT definidos correctamente            |                                            |
| Publisher (Python)     | Publica datos simulados si se ejecuta directamente          |                                            |
| Subscriber (Python)    | Se suscribe a topic correcto según menú                     |                                            |
| Offline logs           | Se genera archivo si MTU pierde conexión                    |                                            |
| Datos enviados         | Formato válido `TEMP:`, `HUM:`, `RFID:` o `otros`          |                                            |
| Simulador GUI          | Envia datos correctos por serial al MTU                     |                                            |
| `socat`                | Virtualiza puertos serial correctamente                     |                                            |
| Prueba de recuperación | Al reconectar MQTT, se reenvían nuevos datos sin error      |                                            |


---

## Recomendaciones Finales

- Ejecuta siempre primero `socat` y el simulador GUI si no usas Arduino.
- No cierres la terminal de `socat`, déjala corriendo en segundo plano.
- Usa `subscriberGrl.py` si deseas visualizar todos los topics sin filtro.
- Asegúrate de que Mosquitto permita conexiones remotas si trabajas en red.

---

## Autor

Ciberseguridad 6o Semestre, Uiversidad Amerike  
Proyecto para entorno de ciberseguridad e IoT académico – 2025