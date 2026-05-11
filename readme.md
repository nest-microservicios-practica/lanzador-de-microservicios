# Dev

1. Clonar el repositorio
2. Crear un .env basado en el .env.template
3. Ejecutar el comando `git submodule update --init --recursive` para reconstruir los submodulos
4. Ejecutar el comando `docker compose up --build`


## PASOS PARA GENERAR PRODUCCION DE TODOS LOS MICROSERVICIOS

1. Ejecutar el comando ```docker compose --env-file .env.prod  -f docker-compose.prod.yml build```
el anterior comando crea las imagenes para produccion y al mismo tiempo corre la migracion para el microservicio de pedidos, para los demas no se hizo, para auth-miservice no era necesario para el ejemplo actual, pero, el dockerfile.prod de pedido microservice tiene un ejemplo de como correr las migraciones con prisma al estar creando la imagen de produccion
2. para probar localmente puedo levantar los contenedore con las imagenes de produccion ```docker compose --env-file .env.prod  -f docker-compose.prod.yml up```

## PASOS SUBIR UNA IMAGEN DE UN MICROSERVICIO A hub.docker.com/

1. abrir la terminal y movernos hasta la carpeta del microservicio
2. tener iniciada la sesion desde el navegador web en [hub.docker.com/](https://hub.docker.com/)
3. tener docker corriendo en la maquina donde estoy trabajando
4. tener en la raiz del microservicio archivo `dockerfile.prod`
5. para este ejemplo utilizamos el nombre de `cliente-gateway` para el microservicio que queremos subir
6. ejecutar en la terminal el comando `docker build -f .\dockerfile.prod -t josueperezf/cliente-gateway .`
7. luego de generar la imagen localmente, la subimos ejecutando en la terminal el comando: `docker push josueperezf/cliente-gateway`

## PASOS PARA INSTALAR Y PROBAR KUBERNETES

1. tener docker desktop corriendo
2. en docker desktop, configuracion, kubernetes, habilitarlo si no está
3. instalar chocolatey, si no lo tenemos, lo instalamos, la instalacion se hace mediante un comando que se ejecuta en la powershell como administrador
4. luego de instalar chocolatey, debemos instalar `https://helm.sh/` esto se hace colocando en la terminal el comando `choco install kubernetes-helm`
5. en la terminal de docker, colocar el comando `kubectl version`, si da respuesta estamos bien

## Pasos para crear los Git Submodules

1. Crear un nuevo repositorio en GitHub
2. Clonar el repositorio en la máquina local
3. Añadir el submodule, donde `repository_url` es la url del repositorio y `directory_name` es el nombre de la carpeta donde quieres que se guarde el sub-módulo (no debe de existir en el proyecto)
```
git submodule add <repository_url> <directory_name>

EJEMPLO:
git submodule add https://github.com/nest-microservicios-practica/cliente-gateway.git cliente-gateway
git submodule add  https://github.com/nest-microservicios-practica/productos-microservicio.git   productos-microservice
git submodule add  https://github.com/nest-microservicios-practica/pedidos-microservice.git   pedidos-microservice 


```
4. Añadir los cambios al repositorio (git add, git commit, git push)
Ej:
```
git add .
git commit -m "Add submodule"
git push
```
5. Inicializar y actualizar Sub-módulos, cuando alguien clona el repositorio por primera vez, debe de ejecutar el siguiente comando para inicializar y actualizar los sub-módulos
```
git submodule update --init --recursive
```
6. Para actualizar las referencias de los sub-módulos
```
git submodule update --remote
```

### NOTAS PARA PROGRAMAR CON MICROSERVICIOS POR REFERENCIA COMO ESTA ESTE

si estamos editando ejemplo el gateway o cualquier otro microservicio, debemos:

1. crear el commit en la seccion de ese microservicio, hacer el push en ese microservicio
2. luego debemos actualizar la referencia en nuestro proyecto lanzador de microservicios con un commit y ejecutar el push desde el repo padre o lanzador.


## Importante
Si se trabaja en el repositorio que tiene los sub-módulos, **primero actualizar y hacer push** en el sub-módulo y **después** en el repositorio principal. 

Si se hace al revés, se perderán las referencias de los sub-módulos en el repositorio principal y tendremos que resolver conflictos.


