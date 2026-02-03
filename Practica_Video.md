# PRÁCTICA 2 - VÍDEO

## ÍNDICE

1. [Introducción](#1-introducción)
2. [Software necesario](#2-software-necesario)
3. [Comprobar la información del vídeo](#3-comprobar-la-información-del-vídeo)
4. [Remuxing: cambio de contenedor. De .mp4 a .mkv](#4-remuxing-cambio-de-contenedor-de-mp4-a-mkv)
5. [Cambio de códecs y comparación](#5-cambio-de-códecs-y-comparación)
    * [5.1 - Creación de un fichero en H.264](#51---creación-de-un-fichero-en-h264-con-un-bitrate-de-2mbps)
    * [5.2 - Creación de un fichero en H.265](#52---creación-de-un-fichero-en-h265-con-un-bitrate-de-2mbps)
6. [Simulación de streaming con diferentes tipos de fichero](#6-simulación-de-streaming-con-diferentes-tipos-de-fichero)
    * [6.1 - Preguntas finales](#61---preguntas-finales)

---

## 1. Introducción

En esta práctica solo usaremos una máquina con ``Ubuntu 22.04.03`` y una herramienta llamada ``FFmpeg``. La usaremos para hacer un analísis de la información que hay por dentro de un vídeo, cambiaremos de contenedor, cambiar de codecs y compararlos.

## 2. Software necesario

- FFmpeg

    ```bash
    sudo apt update # actualizar paquetes
    sudo apt install ffmpeg -y # instalación
    ```

Una vez que lo hemos instalado, tenemos que descarganos el [vídeo](big-buck-bunny.mp4) para empezar con la práctica.

## 3. Comprobar la información del vídeo

Tenemos que hacer un analísis del vídeo descargado anteriormente y ver, que contenedor usa, codec, bitrate, etc.

Para esto, tenemos que ir al directorio donde se encuentra el vídeo y poner el siguiente comando:

```bash
ffprobe -v error -show_streams big-buck-bunny.mp4
```

Al haber puesto ese comando, nos dice toda la información del vídeo, pero la que nos interesa es lo siguiente:

```bash
- codec_name=h264 # Codec de vídeo
- width=1920 # Ancho de vídeo
- height=1080 # Altura de vídeo
- r_frame_rate=24/1 # Frame/segundo 
- avg_frame_rate=24/1 # Media Frame/segundo
- duration=30.000000 # Duración del vídeo
- bits_per_raw_sample=8 # Profundidad de color en bits
```

## 4. Remuxing: cambio de contenedor. De .mp4 a .mkv

Para hacer el cambio usamos este comando:

```bash
ffmpeg -i big-buck-bunny.mp4 -c:v copy -c:a copy big-buck-bunny.mkv
```

Con esto, hemos cambiado el contenedor (formato de forma más coloquial) de mp4 a mkv. En cuanto a consumo de CPU no se nota mucho, es algo inferior el mkv, pero nada del otro mundo. El tamaño no cambia absolutamente nada.

## 5. Cambio de códecs y comparación

### 5.1 - Creación de un fichero en H.264 con un bitrate de 2Mbps

```bash
ffmpeg -i big-buck-bunny.mp4 -c:v libx264 -b:v 2M -c:a copy big-buck-bunny-h264-2mbps.mp4
```
> Consume bastente CPU y pesa bastante menos que los anteriores.

### 5.2 - Creación de un fichero en H.265 con un bitrate de 2Mbps

```bash
ffmpeg -i big-buck-bunny.mp4 -c:v libx265 -b:v 2M -c:a copy big-buck-bunny-h265-2mbps.mp4
```
> Consume aún más CPU y pesa un pelín más que el H264. También tiene menos "artefactos" que el anterior.

## 6. Simulación de streaming con diferentes tipos de fichero

Vamos a hacer simulaciones de streaming. Una a baja calidad y otra en alta.

1. Low (móvil):
    - Resolución: 240p
    - Bitrate: 400k
2. High (fibra):
    - Resolución: 1080p
    - Bitrate: 2Mbps

### 6.1 - Preguntas finales:

1. Almacenamiento: Si tu servidor tiene un disco de 500 GB, ¿cuántas horas de vídeo del
perfil "HD" (2 Mbps) podrías alojar?

    **Espacio disponible:** 500 * 8 = 4.000Gb = 4.000.000Mb 
    > De GB a Gb, es decir de Gigabytes a Gigabits. Hacemos esto porque el espacio del disco está en GB y el vídeo está en Mb

    **Horas que puede alojar:** (4.000.000 / 2) / 3.600 = 555,5 Horas
    > Pasamos primero de Gb a Mb (de 4.000 a 4.000.000) y el resultado lo pasamos de segundos a horas (2.000.000 / 3600)

2. Red: Tienes una línea de 100 Mbps simétricos. ¿Cuántos usuarios podrían ver el perfil
"Móvil" (400 kbps) simultáneamente antes de saturar el 80% de la línea?

    **80% de la línea:** 80Mbps = 80.000kbps
    > Pasamos de Mbps a kbps (80 * 1000)

    **Antes de saturar el 80%:** 80.000 / 400 = 200 Usuarios
    > Calculamos el Ancho de banda / consumo por móvil
