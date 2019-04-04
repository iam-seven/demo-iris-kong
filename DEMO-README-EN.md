
# DEMO IRIS  2019.1 + KONG + KONGA on Docker

### Joel Espinoza
Sales Engineer Intersystems LAtam

**github**: https://github.com/iam-seven/  | **twitter**: @joespinozaf
###### version 1.0

---
## Scope

Create a demo using the IRIS 2019.1 platform deploying Rest services (GET and POST for the example), Kong as the API Gateway and Konga as the Kong administration interface. All this running in containers on Docker and orchestrated with docker-compose.

>### Notes:
> When ** SERVER ** is mentioned (with upper case letters) it corresponds to the server where Docker is running, that is, the main host does not contain the containers.
>
>**CONTAINER ID** is the hash code of the container that runs a service and is obtained by executing `docker ps`.



## Content of the repository
The repository contains the following files:

     * .md: files with information
     * Dockerfile-iris: creation of the container for IRIS, including installation.
     * docker-compose.yml: Global container creation file.

Structure of the repository:
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
> The IRIS license (active) must be copied to the Licenses directory with the name IRIS.key
> The installer of IRIS 2019.1.0SQL should be copied to the SHARED directory

## Software
    Software used for this demo:
    1. IRIS 2019.1
    2. Docker 18.01 sobre CentOS 7
        2.1 Contenedores:
            * Ubuntu:latest
            * Kong:1.1.0
            * Konga:latest
            * Postgres:9.5
    3. Postman u otro cliente de servicios.
    4. Eclipse con Plugin Atelier

## Detail of the Demo
     The demo creates four Dockers containers: IRIS 2019.1, Kong, Konga and postgres (used by kong and konga). And it leaves all the services available to configure a complete environment.

     You must configure a Namespace, a web app and import the attached objects. This will create a table, the GET and POST services and finally the dispatcher to publish the services.
    
     Configure Kong and Konga and finally, what will allow to publish the services through the api manager.

## 
442/5000
Download and generate the demo

     Download the repository from Github, and follow the following steps:

1. Edit the file docker-compose.yml with the network preferences and ports to be made available.

     * The default virtual network is: arkham.
     * All ports configured are the default of each software.
     * It is recommended not to change the versions of the software.

2. Once edited, open a terminal window and execute:
```sh
$ docker-compose up
```

This will create the images and then generate the containers with the necessary software.

3. Open a new terminal and validate the status of the containers.
```sh
$ docker ps
CONTAINER ID        IMAGE
24fe64b35fca        kong:1.1.0
c272b4cfd46e        ms_irisdb
d7ec5a627068        postgres:9.5
a2e83acd76a2        ms_konga
```
ID of the containers will allow us to execute some additional tasks, in particular on IRIS.

---

# Post installation tasks

## Validate Services

Once everything is done, we can start with the task of validating the services. By opening the URL below a response should be received as indicated in each case.

1. Open a browser (Chrome, Firefox, etc.).
2. Validate services:
    * Kong Cliente: http://SERVER:8000  (JSON response)
    * Kong Administración: http://SERVER:8001 (JSON response)
    * Konga: http://SERVER:1337 (Web app)
    * IRIS Management Portal: http://SERVER:52773/csp/sys/UtilHome.csp (Web app)


### IRIS Service validation

It is very likely that the IRIS service does not running by itself, for this to run:
```sh
$ docker exec <CONTAINER ID> bash ./usr/irissys/irisstart
IRIS is already up! 
```
_(if the service is up and running)_

### Obtain the IP of the Containers

It is important to know the IP "docker" of the containers with the services, this because to configure the services in Konga you can not use localhost or the name of the DOCKER server. The same will happen when configuring the IRIS services in Kong.

To obtain the IP docker of the containers, the following command must be executed:
```sh
$ docker inspect <CONTAINER ID> |grep IPAddress
            "SecondaryIPAddresses": null,
            "IPAddress": "",
                    "IPAddress": "172.19.0.4"
```
> the `CONTAINER ID` is the hash code showed by: `docker ps` for each cointainer.

---

## IRIS configuration

### Create a Namespace or use an existing one

You must create a `namespace` in IRIS through the Management Portal, the database is recommended to be stored in` / var / data` so that the files are out of the container and thus the stored information is not lost. It is understood that the user has knowledge of how to create a namespace and a web application in IRIS.

1. Open http://SERVER:52773/csp/sys/UtilHome.csp  SERVER is hostname or ip address of Docker server.
2. Sign in using:
> 
>     User: superuser
> Password: pss.01
> 
3. Create the Namespace and the databases, following the previous recommendation.
4. Continue to the next step.

### Import file with Objects

The `DemoServices.xml` file corresponds to an Export of Objects between them:
    * Products (table)
    * Rest.Index (dispatcher)
    * ServicesProducts (Rest services)
These objects must be imported into the Namespace created on the IRIS server in Docker.

1. Configure Atelier to connect to the IRIS server in docker
2. Import .xml file to the created Namespace
3. Validate that the objects have been created

### Create webapp for Rest Services

Create an application to publish the imported Rest services

1. In Administration> Security> Web Applications> `Create new web application`
2. Configure General:
    > In name for the example: / csp / api / demoserv
    > Namespace: the one you are using
    > Select `Activate: Rest` and in" ** Send Class ** ":` DEMO.Rest.Index`
3. Application Roles, add% ALL and Assign. This for demo purposes, then in productive they must use an ad-hoc configuration to their platform.
4. Save and test.

### Try Service Rest

You can insert a record in the `Products` table only to prove that the rest service works.
```Javascript
http://SERVER:52773/csp/api/demoserv/products
```
You should answer at least `[]`, if you added a record, it should be something like:
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

## Configure Kong + Konga

As in the previous case, you should know the IP of the docker container where Kong is installed, for the example we will use `172.90.0.5`.

1. Enter in http: // SERVER: 1337, where SERVER is the hostname or ip of the Docker server.
2. Create an admin user by entering:
     * User
     * Email
     * Key
     * Repetition of the key
3. Create a new connection (Connections)
4. Enter a name of the connection (for reference)
5. Enter the IP of the Container and the port '8001', including http:
     http://172.19.0.5:8001
6. Update Connection and press "Activate" on the Connections screen.
7. If all is well, the `API GATEWAY` menu will appear and present options such as `Services, Routes, Consumers, Plugins, Upstreams and Certificates`.

---

# Configuration of a Service in Kong
A service in Kong corresponds to the publication of APIs through a Gateway, which allows having a single point of entry, as well as load balancing, access security, logging among many other options.

Within the Kong administration interface (Konga) go to the menu option `SERVICES`.

1. Create a new Service (`+ ADD Services`)
2. The following values ​​must be entered:
    * Name: Service reference name
    * Protocol: in our case `http`
    * Host: IP of the docker container where IRIS (who makes the Rest service)
    * Port: port of IRIS `52773`
    * Path: in our case as an example `/csp/api/` (I personally prefer to leave part of the route as identifier of the namespace, so I do not mix services).
3. We save the configuration
4. Then, in the upper part a message will appear in green indicating that the service is Ok, we will select the option `Routes`, in the upper part, under `Service Detail`
5. Press `+ ADD ROUTE` and fill in the following information.
    * Name: reference name, can be by protocol `DemoGET`
    * Methods: in this case `GET` and press `Enter`, the protocol should be marked in gray, otherwise, it is not well registered.
    * Protocols: You must exit `http` and `https`, but add and save !.

If everything is ok, you can enter http://SERVER:8000/demoserv/products and have the same result as http://SERVER:52773/csp/api/demoserv/products, if there is a register something like:
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

# Script Modifications
If you want to use an existing IRIS instance, you must take into account that it must be in a network segment similar to that of Docker containers and if it is an individually generated container, they must have the same network, for the case of the example `arkham`.
It suffices just to remove the `irisdb` section from the service space of the `docker-compose.yml` file.