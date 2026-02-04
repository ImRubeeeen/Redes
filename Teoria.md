# Streaming 2º ASIX/ASIR

## Índice

1. [Descarga directa vs Streaming](#descarga-directa-vs-streaming)
2. [Topología de red](#topología-de-red)
3. [Capa de transporte: TCP vs UDP](#capa-de-transporte-tcp-vs-udp)
4. [QoS: Jitter y Buffer](#qos-jitter-y-buffer)
5. [Protocolos de Streaming](#protocolos-de-streaming)
6. [Icecast 2](#icecast-2)
7. [Mixxx](#mixxx)
8. [Códecs](#códecs)
9. [Cálculo de peso (audio)](#cálculo-de-peso-audio)
10. [Vídeo](#vídeo)
11. [Conversiones](#conversiones)

---

## Descarga directa vs Streaming

**Descarga directa**
- El usuario demanda un fichero de 100 MB y 10 minutos de duración.
- Comienza la descarga, se almacena en buffer y empieza la reproducción.
- Si el usuario termina a los 2 minutos, el servidor ya ha entregado los 100 MB.

**Streaming**
- Los datos se envían en flujo constante.
- No hay almacenamiento local permanente.
- Solo se consume el ancho de banda realmente utilizado (2 minutos en el ejemplo).

---

## Topología de red

### Unicast

- Conexión 1 a 1 (estándar de Internet).
- Si hay 100 oyentes, el servidor abre 100 sockets TCP y envía la información 100 veces.
- **Ancho de banda total**:  
  `BW_total = BW_stream × N_usuarios`
- **Desventaja**: poco escalable.

### Multicast
- El servidor envía la información a una dirección multicast (224.0.0.0 – 239.255.255.255).
- Los routers replican el paquete solo si hay suscriptores.
- **Desventaja**: los routers suelen bloquear multicast. Solo viable en redes internas.

---

## Capa de transporte: TCP vs UDP

- TCP:
  - Orientado a conexión
  - Garantiza entrega y orden de los datos
  - Retransmite paquetes perdidos
  - Más fiable pero más lento (mayor latencia)
  - Usado en: web (HTTP/HTTPS), streaming tipo Netflix/Spotify, correo

- UDP:
  - No orientado a conexión
  - No garantiza entrega ni orden
  - No hay retransmisiones
  - Muy rápido (baja latencia) pero menos fiable
  - Usado en: videollamadas, juegos online, streaming en tiempo real

En resumen, TCP prioriza fiabilidad, UDP prioriza velocidad.

---

## QoS: Jitter y Buffer

### Jitter (fluctuación)

- Variación en el tiempo de llegada de los paquetes.
- Ejemplo: paquete 1 → 20 ms, paquete 2 → 150 ms, paquete 3 → 20 ms.
- Si el jitter supera el tamaño del buffer → **buffer underrun** (cortes de audio).

### Buffer (amortiguador)

- Memoria temporal en cliente y servidor.
- Función: acumular segundos de audio para absorber el jitter.
- A mayor buffer:
  - Más estabilidad
  - Más latencia

### Burst-on-Connect

- Característica de servidores como Icecast.
- Problema: Al conectarse un oyente, el servidor envía datos iniciales a máxima velocidad.
- Solución:  El servidor envía los datos iniciales (ej. 64KB) a la
máxima velocidad posible que permita la red (ej. 10x), llenando el
buffer del cliente casi instantáneamente para que el audio empiece a
sonar de inmediato (Time-to-first-byte reducido).

---

## Protocolos de Streaming

### 1. Capa de transporte

**TCP**
- Si se pierde un paquete, se retransmite (ACK/NACK).
- Ventajas: calidad, compatibilidad con firewalls, NAT y proxies.
- Desventaja: mayor latencia.

**UDP**
- No hay retransmisión.
- Menor latencia, se sacrifica calidad.

---

### 2. Capa de aplicación

#### HTTP Legacy (Icecast)
- Protocolo: ICY
- Conexión TCP continua.
- Puertos: 80, 443, 8000
- Formatos: MP3, OGG, AAC

#### HTTP Adaptativo
- Protocolos: HLS (HTTP Live Streaming de Apple), MPEG-DASH
- No es un flujo continuo. El contenido se divide en *chunks* de 2–10 segundos.
- Pro: Permite calidad adaptativa mediante un *manifest*, que da la opción a descargar un chunk de mejor o peor calidad.

#### Real-Time
- **RTMP**: TCP, obsoleto para usuario final, usado para enviar vídeo al servidor (por ejemplo de OBS a YT/Twitch).
- **RTSP**: Usado en cámaras de seguridad (CCTV) y sistemas domóticos. Generalmente usa UDP para datos y TCP para control.
- **WebRTC**: Para videoconferencia. P2P, cifrado, UDP, funciona en
navegador sin plugins (Google Meet, Discord). 

---

## Cuadro resumen de protocolos

| Protocolo | Base | Latencia | Uso principal | Firewall | CDN |
|---------|------|----------|--------------|----------|-----|
| Icecast | TCP/HTTP | 10–30 s | Radio online | Muy fácil | Difícil |
| HLS/DASH | TCP/HTTP | 15–45 s | Streaming vídeo | Muy fácil | Excelente |
| RTMP | TCP | 2–5 s | Ingesta | Medio | No |
| WebRTC | UDP/TCP | <0.5 s | Videoconferencia | Complejo | No |
| RTSP | UDP+TCP | <1 s | Cámaras IP | Problemas NAT | No |

Como hemos visto, la industria utiliza mucho más TCP que UDP.

- Netflix, HBO, Disney+, etc utilizan HTTP adaptativo. Al ver una película estás descargando pequeños chunks, trozos del vídeo, secuencialmente vía TCP.

- Spotify y Apple Music también usa TCP. ¿Has escuchado una canción alguna vez en Spotify en el que pierdas un fragmento? Quizás se pare pero no escucharás algo raro como en una videollamada. 

- Twitch (del lado del receptor): usa TCP. Por eso hay un delay. 

- La radio online también es TCP (y vamos a ponerlo en práctica con Icecast2).

- Cuando se necesita interactuar con la otra parte el delay no es admisible y por tanto se utiliza UDP.
---

## Icecast 2

Icecast2 es un software de servidor de streaming de código abierto.

- Actúa como una antena de radio virtual.
- Distribuye audio, no genera contenido.

**Características**

- Formatos: MP3, OGG
- Gestión de oyentes
- Puntos de montaje (ej. `/radio-asir`)

## Mixxx

Mixxx es un software de DJ y emisión de audio de código abierto.

- Actúa como la fuente de sonido de una radio online.
- Genera y envía el audio al servidor de streaming (Icecast, por ejemplo).

**Características**

- Reproducción y mezcla de música (DJ)
- Emisión en directo hacia Icecast
- Gestión de listas de reproducción
- Soporte de micrófono
- Controladores MIDI
- Formatos: MP3, OGG, WAV, FLAC

---

## Códecs

Son algoritmos que permiten la compresión de los ficheros de
audio/vídeo. También para la descompresión. Se usa para reducir el trasiego de información sin perder calidad. 

### Audio
- MP3, AAC, Vorbis, WAV

### Vídeo
- H.264, H.265, AV1

---

### Conceptos de audio

- Frecuencia de muestreo: El audio es una onda analógica. Para
digitalizarla hay que muestrearla, algo así como hacerle fotos cada X tiempo. 
  - Estándar: 44,1 kHz
- Profundidad de bits: Es la profundidad es la calidad de la foto. Se trata de la cantidad de bits que se transmiten por segundo: a
mayor cantidad, más calidad. 
  - Estándar: 16 bits (calidad CD)
- Canales: Es el úmero de audios independientes que viajan en el mismo stream.
  - Estándar: mono, estéreo

---

## Códecs con perdida / sin perdida

- Códecs con pérdida: reducen peso sacrificando información que
puede ser imperceptible por el oído humano. Al comprimirlo dicha información se pierde y es irrecuperable.
  - Ejemplo: MP3
- Códecs sin pérdida: comprime el fichero como lo haría un .zip
pero sin eliminar información. Al descomprimir el flujo de bits es
idéntico al original. El factor de compresión es menor a los códecs con pérdida.
  - Ejemplo: FLAC, WAV.

---

## Cálculo de peso (audio)

Peso = Frecuencia x Bits x Canales x Segundos

Sin compresión WAV: 3 minutos de canción en estéreo con
frecuencia de muestreo de 44,1kHz y profundidad de 16 bits.

```
44.100 × 16 × 2 × 3 × 60 / 8 ≈ 31,75 MB
```

### Ejercicios

1. Calcula el peso aproximado de un archivo de audio sin compresión
(WAV) de 5 minutos, con una frecuencia de muestreo de 44,1 kHz, 16
bits y en estéreo.

```
1. Convertir el tiempo a segundos (lo necesitaremos)

5 min x 60 segundos = 300 segundos

2. Calcular el total de bits

44.100 × 16 × 2 × 300 = 423.3600.000 bits

3. Convertir bits a Bytes

423.360.000 / 8 = 52.920.000

4. Convertir Bytes a MegaBytes (MB)

52.920.000 / 1.000.000 = 52,92MB
```

2. Si emitimos un streaming en MP3 a un bitrate constante (CBR) de
128 kbps, ¿cuánto ancho de banda total consumirá el servidor si tiene 25 oyentes simultáneos?

```
1. Calcular el consumo total en kbps

128 kbps x 25 = 3.200 kbps

2. Convertir kbps a Mbps (Megabits por segundo)

3.200/1.000 = 3,2 Mbps
```

3. Calcula el bitrate de un flujo de audio que utiliza una frecuencia de 48kHz, 24 bits de profundidad y un solo canal (mono).

```
1. Calcular bits por segundo (Bitrate bruto)

48.000 x 24 x 1 = 1.152.000 bps (bits/s)

2. Convertir bps a kilobits por segundo (kbps)

1.152.000/1000 = 1.152 kbps
```

4. Tienes un servidor con un límite de subida de 10 Mbps. ¿Cuántos
oyentes a 192 kbps puede soportar teóricamente antes de saturar la
red?

```
1. Igualar unidades (Convertir Mbps a kbps), ya que lo necesitaremos

10 Mbps x 1.000 = 10.000 kbps

2. Dividir capacidad total entre consumo unitario

10.000/192 = 52,0833 oyentes (son 52 realmente)
```
---

## Vídeo

- Contenedores: es un formato de fichero que incluye lo siguiente
  - Pistas de vídeo
  - Pistas de audio
  - Subtítulos
  - Metadatos
- Ejemplo: MP4, MKV, MOV, OGG

### Ejercicios

```
Peso = (Ancho x Alto) x Profundidad de color x FPS x Tiempo
```

- Resolución: 
  - 1080p -> 1920 x 1080
  - 4K -> 4096 x 2160
  - 8K -> 7680 x 4320

- Profundidad de color: bits usados para definir el color de cada píxel (24 habitualmente bits: 8+8+8)

- FPS: Frames, fotos, por segundo.

Al utilizar un códec, comprimo el vídeo y ya no se envía el vídeo píxel a píxel puesto que el códec ha decidido qué píxeles ha mantenido y cuáles ha eliminado. El bitrate es el dato que nos interesa en el caso de ficheros comprimidos.

```
Peso = Bitrate x Tiempo
```

El bitrate es la cantidad de información que puede enviarse por
segundo.

| Resolución | Calidad | Bitrate Mínimo | Bitrate Recomendado |
| :--- | :--- | :--- | :--- |
| 4K (2160p) | Ultra HD | 15 Mbps | 25 - 45 Mbps |
| 1080p (Full HD)| Alta | 4 Mbps | 6 - 9 Mbps |
| 720p (HD) | Media | 1.5 Mbps | 3 - 4 Mbps |
| 480p (SD) | Estándar | 500 kbps | 1 Mbps |
| 360p | Baja | 400 kbps | 700 kbps |

1. La pesadilla del almacenamiento

Un estudio de cine graba en RAW (sin comprimir) con una cámara 4K (3840x2160 píxeles), a 60 fps y una profundidad de color de 30 bits (HDR).

- Calcula el bitrate en Gbps (Gigabits por segundo).

```
1. Píxeles por cuadro: 

3.840 x 2.160 = 8.294.400 píxeles.

2. Bits por cuadro: 

8.294.400 píxeles x 30 bits = 248.832.000 bits.

3. Bits por segundo (bps):

248.832.000 bits x 60 fps = 14.929.920.000 bps

4. Pasar a Gbps: Dividimos por $1.000.000.000$ (para pasar de bits a Gigabits):

14.929.920.000 / 1.000 .000 .000 = 14,93 Gbps
```

- ¿Cuánto espacio de disco ocupará una toma de solo 10 segundos?

```
Pasamos de la "velocidad de transmisión" al "peso del archivo".

1. Total de bits en 10 segundos:

14.929.920.000 bps x 10 = 149.299.200.000 bits

2. Convertir bits a Bytes: Dividimos entre 8.

149.299.200.000 / 8 = 18.662.400.000 Bytes

3. Convertir Bytes a Gigabytes (GB): Dividimos entre 1.000.000.000.

18.662.400.000 / 1.000.000 .000 = 18,66 GB
```

- Si tienes un disco duro de 1 TB, ¿Cuántos minutos de vídeo podrías guardar?

```
1. Igualar unidades: 

Sabemos que 1 TB = 1.000 GB

2. Calcular cuántos "bloques de 10 segundos" caben:

Dividimos la capacidad total entre lo que pesan 10 segundos: 

1.000 GB / 18,66 GB = 53,59 (estos son grupos de 10 segundos)

3. Calcular el total de segundos:

53,59 x 10 segundos = 535,9 segundos

4. Convertir segundos a minutos: Dividimos entre 60

535,9 / 60 = 8.93 min
```

2. Quieres retransmitir la graduación por YouTube. Tienes una conexión de fibra con 20 Mbps de subida (upload). Quieres emitir en 1080p usando el códec H.264.

- Si configuras un bitrate de 6 Mbps, ¿qué porcentaje de tu línea de subida estás consumiendo?

```
(6 / 20) x 100 = 30%
```

- Si de repente otros 3 alumnos empiezan a emitir sus propias directos a la misma calidad (6 Mbps cada uno), ¿qué pasará con la emisión? Justifica la respuesta técnica (buffering, saturación de red, latencia).

```
4 alumnos x 6 Mbps cada uno = 24 Mbps

Comparación con la capacidad:

- Consumo necesario: 24 Mbps
- Capacidad disponible: 20 Mbps
- Déficit: 4 Mbps
```

- ¿Qué solución técnica aplicarías para que los 4 alumnos puedan emitir simultáneamente sin ampliar la línea de fibra?

```
La solución consiste en reducir el bitrate individual para que la suma de los 4 no supere el límite de seguridad (el 80% de la línea que mencionamos anteriormente es lo ideal).

- Cálculo del bitrate máximo total recomendado (80%):

20 Mbps x 0,80 = 16 Mbps totales

- Reparto entre los 4 alumnos:

16 Mbps / 4 = 4 Mbps por alumno.

Solución propuesta: Ajustar la configuración de cada encoder (OBS, por ejemplo) a un bitrate de 4 Mbps.

Compromiso técnico: Se sacrifica un poco de calidad de imagen (se verá algo más comprimido o "pixelado" en escenas de mucho movimiento), pero se garantiza que las 4 señales lleguen de forma fluida y sin cortes a YouTube.
```
---

## 11. Conversiones

**Conversiones básicas**
Bits y Bytes
1 byte = 8 bits  
bits = bytes * 8  
bytes = bits / 8  

**Unidades de tamaño (almacenamiento)**
1 KB = 1000 bytes  
1 MB = 1000 KB  
1 GB = 1000 MB  
1 TB = 1000 GB  

bytes a KB = bytes / 1000  
KB a MB = KB / 1000  
MB a GB = MB / 1000  
GB a TB = GB / 1000  

**Unidades de velocidad (red)**
1 kbps = 1000 bps  
1 Mbps = 1000 kbps  
1 Gbps = 1000 Mbps  

bps a kbps = bps / 1000  
kbps a Mbps = kbps / 1000  
Mbps a Gbps = Mbps / 1000  

**Tiempo**
segundos = minutos * 60  
minutos = segundos / 60  

**Conversión tamaño desde bits**
bytes = bits / 8  
KB = bytes / 1000  
MB = KB / 1000  
GB = MB / 1000  

**Conversión tamaño hacia bits**
bits = bytes * 8  
bits = KB * 1000 * 8  
bits = MB * 1000 * 1000 * 8  
bits = GB * 1000 * 1000 * 1000 * 8
