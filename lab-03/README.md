<p align="center">
    <img src="../resources/header.png">
</p>

# Lab - 03 Una pequeña práctica, un "Hola Mundo" por supuesto.

## Introducción

Este laboratorio está dividido en dos secciones:

- [**Dockerizar una aplicación web**](./frontend): Tomando como base una pequeña aplicación *HelloWorld* realizada con Angular, crearemos una imagen Docker que incluya un servidor web **Nginx** que publique la aplicación. La aplicación web intentará conectarse a un servidor para obtener un saludo personalizado. Si no es posible obtener el saludo desde el servidor mostrará un saludo por defecto.  
- [**Dockerizar una aplicación Spring Boot**](./backend): Partiendo de una aplicación Spring Boot, crearemos una imagen que al arrancar publicará una API Rest que podrá ser consumida por la aplicación Web. La API devuelve saludos personalizados en distintos idiomas. Los distintos saludos los obtiene conectándose a una base de datos alojada en otro servidor. Si no es posible acceder a la base de datos devuelve un saludo por defecto.


### Objetivos y resultados
El principal objetivo y resultado esperado es dockerizar por un lado el frontend y por otro el backend. En estos laboratorios no estableceremos la conexión entre ambas y dejaremos eso para otro laboratorio posterior donde se integrarán junto al servidor de base de datos.

< [Lab 02. Dokerfiles](../lab-02) | [Lab 03. Frontend](./frontend)>

<p align="center">
    <img src="../resources/header.png">
</p>
