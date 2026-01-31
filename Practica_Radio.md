# PRÁCTICA 1 - RADIO

## ÍNDICE

## 1. Introducción

Antes de empezar, esta práctica se va a hacer con 2 máquina virtuales con Ubuntu 22.04.03 y adaptador puente en ambas. Una se usará como servidor de streaming y la otra será el DJ.

Tenemos que asignarle una IP estática a cada una para evitar problemas. Para ello, vamos a usar la que viene
al entrar a la máquina que se nos asigna por DHCP.

```bash
ip a
```
> Con la IP que nos da, le asignamos esa misma IP, con su gateway y máscara de subred.

## 2. Software necesario

### 2.1 - Instalación y explicación de cada uno

Recomiendo estar en **su** para no tener que usar todo el rato el "sudo" para instalar las cosas.

1. Icecast 2

    ```bash
    apt update # actualizar paquetes
    apt install icecast2 -y # instalar el programa y decirle que si a todo para que no te pregunte
    ```
   > Máquina que se encangará del servidor de streaming 

- Cuando ejecutamos el comando de instalación nos pregunta si queremos configurar la contraseña, le tenemos que dar a que sí, ya que es importante para que pueda funcionar.

- También nos pregunta por el nombre del servidor, en mi caso, puse **localhost**. Después de este paso, ya viene la parte de poner las contraseñas para distintas cosas (del repetidor, del administrador...).

2. Mixxx

    ```bash
    apt update
    apt install mixxx -y
    ```
    > Máquina que se encargará de hacer de DJ

## 3. Configuración y/o uso de éstos

### 3.1 - Icecast 2

Después de que se haya instalado correctamente, tenemos que acceder desde el navegador web de la siguiente forma:

```
http://localhost:8000 // Icecast2 usa el puerto 8000 para poder acceder por defecto
```
> Si queremos administrar las emisiones, tenemos que ir al panel de administrador y nos pregunta por usuario y contraseña. El usuario es **admin** y la contraseña la que hayamos puesto en la instalación.

### 3.2 - Mixxx

Cuando se haya instalado, podemos acceder como cualquier otra aplicación desde la interfaz gráfica. Al abrirlo, nos pregunta por la carpeta donde se almacenará la música.

- Configuración: Ahora tenemos que empezar con los ajustes que tenemos que cambiar. Para eso, tenemos que hacer lo siguiente: ``Menú de arriba`` > ``Opciones`` > ``Preferencias`` > ``Emisión en vivo``.

- Parámetros que tenemos que modificar:

    | Tipo      | Montar                    | Servidor                     | Puerto                                   | Identificación | Contraseña                          |
    |-----------|---------------------------|------------------------------|------------------------------------------|----------------|-------------------------------------|
    | Icecast 2 | /ruben (usuario de cada uno)| 172.30.17.89 (VM streaming) | 8000 (modificable)                       | source         | La asignada al instalar Icecast 2   |

    Después de hacer estos cambios, aplicamos la configuración y desde el Mixxx le damos a ``ON AIR`` (una vez que hayamos puesto en la carpeta donde busca las canciones cualquier fichero en .mp3). 

## 4. Comprobación de su funcionamiento

Tenemos que acceder desde el navegador: En la máquina servidor accedemos de la misma forma pero poniendo /ruben (usuario que hemos configurado aquí) y en cliente (máquina donde está el Mixxx) controlamos la música.

Adjunto vídeo donde se ve el funcionamiento: 


