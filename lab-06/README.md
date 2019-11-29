<p align="center">
    <img src="../resources/header.png">
</p>

# Lab 06 - Escalado y orquestración de microservicios (Reto para los valientes)
<br/>
Llegamos al siguiente nivel... ¡Reto! 
<br/>

## Objetivos y resultados

El objetivo de este laboratorio es montar un clúster con las aplicaciones que se han creado en los laboratorios anteriores, escalarlas y orquestrarlas. 

Ya que hemos aprendido cómo crear contenedores, necesitamos alguna herramienta o plataforma para orquestrar, escalar y planificar esos contenedores, ya que podremos tener algunos que estén corriendo todo el día, otros que sean bajo demanda, otros que tengan mucha disponibilidad...

Para realizar esto, existen varias herramientas: kubernetes, docker swarm...En este laboratorio nos centraremos en Docker Swarm.

Docker Swarm es la solución nativa propia de Docker para clusters de contenedores Docker que tiene la ventaja de estar estrechamente integrada en el ecosistema de Docker y utiliza su propia API. Supervisa la cantidad de contenedores distribuidos en clústeres de servidores y es la forma más conveniente de crear una aplicación acoplada en clúster sin hardware adicional. Proporciona un sistema de orquestación a pequeña escala pero útil para las aplicaciones Dockerizadas.

Pero, **¿qué es un clúster?**

Un clúster es un conjunto de máquinas que pueden trabajar juntas.

Un clúster swarm consta de varios hosts Docker que se ejecutan en modo swarm y actúan como **managers** (para administrar la membresía y la delegación) y **workers** (que ejecutan servicios swarm). Un host Docker determinado puede ser un manager, un worker o desempeñar ambos roles. Cuando creamos un servicio, definimos su estado óptimo (número de réplicas, recursos de red y almacenamiento disponibles, puertos que el servicio expone al mundo exterior y demás). Docker trabaja para mantener ese estado deseado. Por ejemplo, si un nodo de trabajo deja de estar disponible, Docker programa las tareas de ese nodo en otros nodos. Una tarea es un contenedor en ejecución que forma parte de un servicio swarm y es administrado por un swarm manager, en lugar de un contenedor independiente.

Una de las ventajas clave de los servicios swarm sobre los contenedores independientes es que se puede modificar la configuración de un servicio, incluidas las redes y los volúmenes a los que está conectado, sin la necesidad de reiniciarlo manualmente. Docker actualizará la configuración, detendrá las tareas de servicio con la configuración desactualizada y creará nuevas que coincidan con la configuración deseada.


Docker Swarm proporciona alta disponibilidad de nodos managers utilizando un algoritmo Raft, que necesita un número impar de nodos registrados como manager (3, 5, 7 y así sucesivamente). Este algoritmo selecciona un líder entre los distintos nodos manager.
<br/>

## Crear un clúster swarm

> Los siguientes pasos están pensados para hacerlos utilizando el demonio de Docker, instalando en tu sistema operativo de preferencia (Linux/OSX preferentemente). Al final de cada paso se darán opciones para poder reproducirlo dentro del entorno Play with Docker.

### Paso 1. Configurando el clúster

Con esta configuración, el clúster estara formado por 3 nodos managers y 3 nodos workers:

    docker-swarm-manager-1
    docker-swarm-manager-2
    docker-swarm-manager-3
    docker-swarm-worker-1
    docker-swarm-worker-2
    docker-swarm-worker-3

Estas máquinas se desplegarán en su propia red:

    192.168.66.1/24

Siendo la primera ip del pool DHCP:

    192.168.66.100

Para crear cada docker machine en VirtualBox, ejecuta el siguiente comando poniendo el nombre correcto  cada vez:	

    docker-machine create --driver virtualbox --virtualbox-cpu-count 1 --virtualbox-memory 1024 --virtualbox-hostonly-cidr "192.168.66.1/24" <docker-machine-name>

Antes de configuar el clúster, debemos hace que el entorno apunte a la primera docker machine:

    eval $(docker-machine env docker-swarm-manager-1)

En Play with Docker sólo podemos utilizar cinco instancias, por lo que no podremos tener 3 nodos managers y 3 workers. Como el algoritmo Raft necesita un número impar de nodos managers, tendremos 3 managers y 2 workers. 

> **Si partimos de una nueva sesión de Play with Docker, o no tenemos instancias abiertas, podemos crear automáticamente cinco instancias pinchando en el icono de la llave inglesa y seleccionando la opción "3 Managers and 2 Workers". Esto nos llevaría directamente al final del paso 2.**

Si lo queremos hacer manualmente, o utilizar una instancia ya creada, abriremos hasta cinco instancias diferentes utilizando el botón "Add new instance" en la izquierda de la interfaz.
	
### Paso 2. Inicializar el clúster

Para inicializar el clúster, se usa el siguiente comando:

    docker swarm init --advertise-addr <IP-DE-LA-INSTANCIA> 

En nuestra maquina local utilizaríamos la IP 192.168.66.100 de ejemplo. En Play With Docker, usaremos la IP de la instancia donde ejecutemos el comando, podemos ver la IP de cada instancia en la parte izquierda de la pantalla.

Una vez inicializado, el clúster expone dos tokens: Uno para añadir nodos manager y otro para añadir nodos workers
El comando para obtener los tokens es:

    docker swarm join-token manager -q
    docker swarm join-token worker -q

Una vez tengamos los tokens, cambia el entorno para que apunte a cada docker machine, cada nodo manager y cada nodo worker:

    eval $(docker-machine env <docker-machine-name>)

Usamos el comando `swarm join` en cada nodo ya sea manager o worker:

    docker swarm join --token <manager-or-worker-token> 192.168.66.100:2377

En Play with Docker, tendremos que ir instancia a instancia ejecutando el siguiente comando (como la instancia que inicializó el clúster es un manager, el join tendrá que ejecutarse en dos instancias utilizando el token de manager, y en otras dos utilizando el token de worker):

    docker swarm join --token <manager-or-worker-token> <IP de la instancia dónde se inicializó  el swarm>:2377

Se puede obtener este comando resolviendo ya los token y la IP si ejecutamos el comando `docker swarm join-token manager` (para managers) o `docker swarm join-token worker` (para workers) en la instancia que inició el clúster. Copiando la salida del primer comando en dos instancias y la salida del segundo en otras dos, ya tendremos el clúster con los cinco nodos añadidos.

Ejecutando el comando,

    docker node ls

veremos si se ha creado el swarm correctamente, con todos sus nodos asociados, y podremos reconocer el rol de cada nodo dentro del swarm por el contenido de la columna MANAGER STATUS.

<br/>

### Paso 3. Añadir servicios

Para añadir un servicio al swarm partiendo de una imagen, podemos, desde cualquiera de los nodos manager, ejecutar el siguiente comando:

	docker service create <NOMBRE-DE-LA-IMAGEN>

Por ejemplo, para levantar un servidor `nginx`, utilizaríamos la siguiente sintaxis:

```sh
# --name: El nombre por el que nos refererimos al servicio, lo elegimos nosotros. Si no lo especificamos, se le asignará uno de oficio.
# --publish: Opción concreta de la imagen de nginx, para publicar el servidor que nginx sirve en el puerto 80, en el puerto 8000 de nuestros contenedores.
docker service create --name nginx_server --publish 8000:80 nginx
```

Si hacemos un `docker service ls`, que lista todos los servicios dados de alta en el clúster, veremos cómo se ha añadido nuestro nginx_server, basado en la imagen básica de nginx, que está 

Para escalar los servicios en el clúster, desde cualquiera de los nodos manager, se utiliza el comando `scale`. En el indicaremos el número de réplicas del servicio que queremos levantar a lo largo y ancho del swarm:

```sh
# Los dos argumentos obligatorios del comando docker service scale son el nombre o ID del servicio, y el número de réplicas al que se desea escalar
docker service scale nginx_server=10
```
Si todo ha ido bien, nuestro servidor de `nginx` se habrá replicado por todo el clúster, y con una llamada al comando

	docker service ps nginx_server

podremos ver información sobre las réplicas levantadas, su estado, y que nodo que gestiona cada una.

Finalmente, con una llamada al comando,
	
	docker service rm nginx_server
	
eliminaríamos el servicio del clúster, y se dejarían de ejecutar todas las réplicas.

<br/>

### Paso 4. Manejar el clúster	

Una vez esta inicializado el clúster, podemos pararlo con el siguiente comando:	

    docker-machine stop docker-swarm-manager-1 docker-swarm-manager-2 docker-swarm-manager-3 docker-swarm-worker-1 docker-swarm-worker-2 docker-swarm-worker-3

Y para arrancarlo de nuevo, usaremos este comando:

    docker-machine start docker-swarm-manager-1 docker-swarm-manager-2 docker-swarm-manager-3 docker-swarm-worker-1 docker-swarm-worker-2 docker-swarm-worker-3
<br/>

En Play with Docker no tenemos instalada la funcionalidad `docker-machine`, por lo que no podemos parar y arrancar el swarm. Para abandonar el swarm, podemos utilizar el siguiente comando nodo a nodo en todos ellos, pero para volver a arrancarlo tendríamos que crearlo de nuevo:

    docker swarm leave --force

## ¡Reto! 

El reto consiste en crear un clúster con Docker Swarm y orquestar y escalar los servicios que se han creado en los laboratorios 03 y 04. Como las imágenes que utilizamos no están en el repositorio, tendrás que investigar el uso de la imagen `registry`, que te permitirá distribuir las imágenes que hemos creado entre los distintos nodos, el comando `push` de `docker-compose` que te permitirá subir el stack al registro, y el comando de Docker `stack deploy`, que te permitirá despleglarlo en el clúster.


[< Lab 05 - Creando un stack de servicios con docker-compose](../lab-05) | [Lab 07 - Dockerización de la aplicación Event-UI >](../lab-07)

<p align="center">
    <img src="../resources/header.png">
</p>

