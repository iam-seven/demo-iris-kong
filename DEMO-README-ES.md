
# DEMO IRIS  2019.1 + KONG + KONGA en Docker

### **Joel  Espinoza F.**   
### Sales Engineer Intersystems LAtam

>### **github: https://github.com/iam-seven/  | twitter: @joespinozaf**
>
>### version 1.0


---
## Objetivo

Crear una demo utilizando la plataforma IRIS 2019.1 desplegando servicios Rest (GET y POST para el ejemplo), Kong como API Gateway y Konga como interfaz de administración de Kong. Todo esto ejecutandose en contenedores sobre Docker y orquestados con docker-compose.

>### Convensiones
>Cuando se menciona **SERVIDOR** (con letras mayusculas) corresponde al servidor donde esta corriendo Docker, es decir, el host principal no a los contenedores.
>
>**CONTAINER ID** corresponde al código hash del contenedor que corre un servicio y se obtiene ejecutando `docker ps`.
>


## Contenido del repositorio
El repositorio contiene los siguientes archivos:

    * .md: archivos con información
    * Dockerfile-iris: creación del contenedor para IRIS, incluyendo la instalación.
    * docker-compose.yml: Archivo de creación de contenedores global.

Estructura del repositorio:
```sh
ms
|-- DEMO-README-ES.md
|-- DEMO-README-EN.md
|-- docker-compose.yml
|-- Dockerfile-iris
|-- datairis (mapeado a /var/data dentro del contenedor)
|-- licenses
|   -- IRIS.key (licencia de IRIS) (no incluido)
|-- Objects
|   -- DemoServices.xml
|-- SHARED
    |-- IRIS-2019.1.tar.gz (instalador de IRIS 2019.1) (no incluido)

```
> Se deberá copiar la licencia de IRIS (activa) en el directorio Licenses con el nombre IRIS.key
> Se deberá copiar el instalador de IRIS 2019.1.0SQL en el directorio SHARED

## Software
    Software utilizado para en este demo:
    1. IRIS 2019.1
    2. Docker 18.01 sobre CentOS 7
        2.1 Contenedores:
            * Ubuntu:latest
            * Kong:1.1.0
            * Konga:latest
            * Postgres:9.5
    3. Postman u otro cliente de servicios.
    4. Eclipse con Plugin Atelier

## Detalle de la Demo
    La demo crea cuatro contenedores Dockers: IRIS 2019.1, Kong, Konga y postgres (usado por kong y konga). Y deja todos los servicios disponibles para configurar un ambiente completo.

    Se debe configurar un Namespace, una app web e importar los objetos adjuntos. Esto creará una tabla, los servicios de GET y POST y finalmente el dispatcher para publicar los servicios.
    
    Configurar Kong y Konga y finalmente, lo que permitirá publicar los servicios a traves del api manager.

## Descarga y generar de la demo

    Descarga el repositorio desde Github, y seguir los siguientes pasos:

1. Editar el archivo docker-compose.yml con las preferencias de red y puertos a disponibilizar.

    * La red virtual por omisión es: arkham.
    * Todos los puertos configurados son los default de cada software.
    * Se recomienda no cambiar las versiones de los software.

2. Una vez editado, abrir una ventana de terminal y ejecutar:
```sh
$ docker-compose up
```

Esto creará las imágenes y luego generará los contenedores con los software necesarios.

3. Abrir una nueva terminal y validar el estado de los contenedores.
```sh
$ docker ps
CONTAINER ID        IMAGE
24fe64b35fca        kong:1.1.0
c272b4cfd46e        ms_irisdb
d7ec5a627068        postgres:9.5
a2e83acd76a2        ms_konga
```
ID de los contenedores nos permitirán ejecutar algunas tareas adicionales, en particular sobre IRIS.

---

# Tareas post instalación

## Validar Servicios

Una vez ejecutandose todo, podemos comenzar con la tarea de validación de los servicios. Abriendo las URL a continuación se debe recibir una respuesta como se indica en cada caso.

1. Abrir un browser (Chrome, Firefox, etc.).
2. Validar servicios:
    * Kong Cliente: http://SERVIDOR:8000  (Código JSON)
    * Kong Administración: http://SERVIDOR:8001 (Código JSON)
    * Konga: http://SERVIDOR:1337 (Aplicación Web)
    * IRIS Management Portal: http://SERVIDOR:52773/csp/sys/UtilHome.csp (Aplicación Web)


### Validación Servicio IRIS

Es muy probable que el servicio de IRIS no suba por si solo, para esto ejecutar:
```sh
$ docker exec <CONTAINER ID> bash ./usr/irissys/irisstart
IRIS is already up! 
```
_(si el servicio esta arriba)_

### Conocer la IP de los Contenedores

Es importante conocer la IP "docker" de los contenedores con los servicios, esto porque para configurar los servicios en Konga no se puede usar localhost o el nombre del servidor DOCKER. Lo mismo ocurrirá cuando se configuren los servicios de IRIS en Kong.

Para obtener la IP docker de los contenedores, se debe ejecutar el siguiente comando:
```sh
$ docker inspect <CONTAINER ID> |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.19.0.4"
```
> El `CONTAINER ID` corresponde al código entregado por: `docker ps` para cada contenedor.

---

## Configuración IRIS

### Crear un Namespace o usar uno existente

Se deberá crear un `namespace` en IRIS a través del Management Portal, la base de datos se recomienda que se almacene en `/var/data`para que los archivos queden fuera del contenedor y así no se pierda la información almacenada. Se entiende que el usuario tiene conocimientos de como crear un namespace y una aplicación web en IRIS.

1. Ingresar en http://SERVIDOR:52773/csp/sys/UtilHome.csp siendo SERVIDOR el hostname o ip del servidor de Docker.
2. Ingresar usando los siguientes datos:
> 
>    USUARIO: superuser
>     CLAVE: pss.01
> 
3. Crear el Namespace y las bases de datos, siguiendo la recomendación anterior.
4. Seguir al siguiente paso.

### Importar archivo con Objetos

El archivo `DemoServices.xml` corresponde a un Export de Objetos entre ellos:
    * Productos (tabla)
    * Rest.Index (dispatcher)
    * ServiciosProductos (servicios Rest)
Estos objetos deben ser importados en el Namespace creado en el servidor de IRIS en Docker.

1. Configurar Atelier para conectarse al servidor IRIS en docker
2. Importar archivo .xml al Namespace creado
3. Validar que los objetos hayan sido creados

### Crear webapp para Servicios Rest

Crear una aplicación para publicar los servicios Rest importados

1. En Administración > Seguridad > Aplicaciones Web >`Crear nueva aplicación web`
2. Configurar General:
    > En nombre para el ejemplo: /csp/api/demoserv
    > Namespace: el que esten usando
    > Seleccionar `Activar: Rest` y en "**Enviar Clase**": `DEMO.Rest.Index`
3. Roles de Aplicación, agregar %ALL y Asignar. Esto para efectos de demo, luego en productivo deben usar una configuración ad-hoc a su plataforma.
4. Guardar y probar.

### Probar Servicio Rest

Pueden insertar un registro en la tabla `Productos` solo para probar que el servicio rest funcione.
```Javascript
http://SERVIDOR:52773/csp/api/demoserv/productos
```
Debera responder al menos `[]`, si agregaron un registro, deberá ser algo como:
```Javascript
[
    {
        id: "1",
        pid: "100",
        name: "Cajas",
        cost: 20,
        state: "A"
    }
]
```
--- 

## Configurar Kong + Konga

Al igual que el caso anterior, se debe conocer la IP del contenedor docker donde esta instaldo Kong, para el ejemplo usaremos la `172.90.0.5`.

1. Ingresar en http://SERVIDOR:1337, siendo SERVIDOR el hostname o ip del servidor de Docker.
2. Crear un usuario administrador ingresando:
    * Usuario
    * Correo electrónico
    * Clave
    * Repetición de la clave
3. Crear una nueva conexion (Connections)
4. Ingresar un nombre de la conexión (como referencia)
5. Ingresar la IP del Contenedor y el puerto '8001', incluyendo http:
    http://172.19.0.5:8001
6. Actualizar Conexión y presionar "Activate" en la pantalla de Conexiones.
7. Si todo esta bien, aparecerá el menú `API GATEWAY` y presentará opciones como `Services, Routes, Consumers, Plugins, Upstreams y Certificates`.

---

# Configuración de un Servicio en Kong
Un servicio en Kong corresponde a la publicación de APIs a través de un Gateway, lo que permite tener un unico punto de entrada, así como balanceo de carga, seguridad de acceso, logging entre muchas otras opciones.

Dentro de la interfaz de administración de Kong (Konga) ir a la opción de menú `SERVICES`.

1. Crear un nuevo Servicio (`+ADD Services`)
2. Se deberán ingresar los siguientes valores:
    * Name: Nombre de referencia al servicio
    * Protocol: en nuestro caso `http`
    * Host: IP del contenedor docker donde esta IRIS (quien disponibiliza el servicio Rest)
    * Port: puerto de IRIS `52773`
    * Path: en nuestro caso como ejemplo `/csp/api/` (personalmente prefiero dejar parte de la ruta como identificador del namespace, así no mezclo servicios).
3. Guardamos la configuración
4. Luego, en la parte superior aparecerá un mensaje en verde indicando que esta Ok el servicio, seleccionaremos la Opcion `Routes`, en la parte superior, bajo `Service Detail``
5. Presionamos `+ADD ROUTE`y llenamos los siguientes datos.
    * Name: nombre de referencia, puede ser por protocolo `DemoGET`
    * Methods: en este caso `GET` y presionen `Enter`, deberá marcarse el protocolo en gris, sino, no esta bien registrado.
    * Protocols: Debe salir `http` y `https`, sino agregar y guardar!.

Si todo esta ok, podran entrar en http://SERVIDOR:8000/demoserv/productos y tener el mismo resultado que http://SERVIDOR:52773/csp/api/demoserv/productos , si hay un regiustro algo así como:
```Javascript
[
    {
        id: "1",
        pid: "100",
        name: "Cajas",
        cost: 20,
        state: "A"
    }
]
```

---

# Modificaciones al Script
Si se desea usar una instancia de IRIS ya existente, se deberá tener en cuenta que debe estar en un segmento de red similar al de los contenedores Docker y si es un contenedor generado de forma individual, deben tener la misma red, para el caso del ejemplo `arkham`.
Basta solo con eliminar la seccion `irisdb` desde el espacio de servicios del archivo `docker-compose.yml`.
