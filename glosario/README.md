<p align="center">
    <img src="../resources/header.png">
</p>

# Docker Cheat Sheet
<br/>
<p align="center">
<img src="resources/architecture.svg" width="500">
<br/>
Arquitectura Docker
</p>
<br/>

## Glosario

**Capa**: archivo de sólo lectura a disposición del sistema

**Imagen**: conjunto vertical y adyacente de capas. Es la base del contenedor

**Contenedor**: instancia ejecutable de una imagen 

**Registry/Hub**: registro privado o público para descargar imágenes

## Comandos

```sh
# Listar todas las imágenes
docker images -a

# Listar todos los contenedores en ejecución
docker ps

# Listar todos los contenedores
docker ps -a

# Iniciar un contenedor nuevo
docker run <nombre imagen>

# Iniciar un contenedor nuevo en background
docker run -d <nombre imagen>

# Iniciar un contenedor
docker start <nombre contendedor>

# Parar un contenedor
docker stop <nombre contendedor>

# mapear puerto del host a puerto del contenedor
# -p <host port>: <container port>
docker run -p 8080:8080 <nombre imagen>

# Ver el log de un contenedor
docker logs <nombre contenedor>

# Tail salida consola contenedor en ejecución
docker logs -f <nombre contenedor>

# Borrar todos los contenedores parados 
docker rm $(docker ps -a -q)

# Borrar todos los contenedores (parados/ejecución)
docker rm -f $(docker ps -a -q)

# Matar todos los contenedores en ejecución
docker kill $(docker ps -q)

# Borrar imagen
docker rmi <nombre imagen>

# Borrar todas las imágenes
docker rmi $(docker images -q)

# Interactuar dentro del contenedor mediante bash shell
sudo docker exec -it <nombre contenedor> bash

# Crear volumen
docker volume create <nombre volumen>

# Listar volúmenes
docker volume ls

# Inspeccionar volumen
docker volume inspect <nombre volumen>

# Borrar volumen
docker volume rm <nombre volumen>

# Compartir almacenamiento entre host y contenedor
#-v <host path>: <contenedor path>
docker run -v <host path>: <contenedor path> <nombre imagen>

# docker compose construir contenedores
docker-compose build

# docker compose iniciar contenedores
docker-compose up -d

# docker compose parar contenedores
docker-compose stop -t 1

# docker compose borrar contenedores
docker-compose rm -f

# docker compose descargar imágenes
docker-compose pull
```
	
<p align="center">
    <img src="../resources/header.png">
</p>