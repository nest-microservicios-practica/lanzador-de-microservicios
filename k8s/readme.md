# PASOS PARA AGREGAR Y LEVANTAR UN MICROSERVICIO PARA KUBERNETES

lo divido en dos secciones, una configuracion inicial si no la tengo, y la dos es la de crear y configurar microservicios y mas

## configuracion inicial - creacion inicial de la carpeta k8s para KUBERNETES

1. la carpeta 'k8s' se creo en blanco si no existe
2. luego si la carpeta `k8s` esta vacia, nos movimos a la terminal y desde alli ejeuctamos el comando `helm create tienda`, si no tengo instalado el helm puedo ir al readme inicial del proyecto, alli explico como instalarlo con chocolatey
3. si la carpeta `k8s` estaba vacia y esto creando todo de cero, borro el contenido de la carpeta template se borro y se inicio de cero.

## crear y configurar microservicios en k8s para KUBERNETES

1. ahora nos movemos a la carpeta de tienda, creamos una carpeta para el microservicio que vamos a trabajar, ejemplo 'cliente-gateway'
2. estando en la carpeta del microservicio que vamos a iniciar a trabajar `cliente-gateway`, en la terminal ejecutamos el comando `kubectl create deployment cliente-gateway --image=josueperezf/cliente-gateway:latest    --dry-run=client -o yaml > deployment.yml` donde `cliente-gateway` es el nombre de nuestro microservicio, y `--imagen` a punta a donde tenemos la imagen, para este ejemplo la busca en docker hub. no le pasamos credenciales porque no estamos pagando por el docker hub. importante, debemos validar que el archivo generado sea `UTF8` para que todo ande bien. si queremos crear el de nats seria algo asi `kubectl create deployment nats --image=nats  --dry-run=client -o yaml > deployment.yml`
3. ahora debemos movernos a la carpeta raiz de `k8s/tienda` y alli, si no lo hemos ejecutado nunca, ejecutamos `helm install tienda .` en caso contrario, ejecutamos `helm upgrade tienda .`. igual si se ejecuta el install y ya se habia ejecutado, no pasa nada.
4. cada vez que cambiemos algo y queramos probar, por obligacion debemos ejecutar el comando `helm upgrade tienda .`para que tome los cambios
5. ahora si queremos que el microservicio se pueda acceder desde fuera de la red interna, ejemplo el cliente-gateway, debemos crear un servicio de tipo `nodeport`, en caso contario de tipo `clusterip` alli debemos decir y hace el que corresponda:
   1. paso para crear un servicio de tipo `nodeport`
      1. ir a la carpeta donde estamos trabajando para nuestro caso es `k8s/tienda/templates/cliente-gateway/`
      2. alli ejecutar el comando `kubectl create service nodeport cliente-gateway --tcp=3000 --dry-run=client -o yaml > service.yml`. ESTO creará un archivo llamado `service.yml` 'DEBEMOS VALIDAR QUE EL ARCHIVO CREADO ES UTF8' con la configuracion de los puerto que expondrá. recordemos que cada cambio que hacemos debemos refrescarlo, para ello tenemos que en la terminal de la raiz `k8s/tienda/` debemos ejecutar el comando `helm upgrade tienda .` para que tome los nuevos cambios
      3. para ver si realmente esta corriendo, ejecutamos en la terminal `kubectl get services` alli podemos ver algo como:
           ```
           NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
           cliente-gateway   NodePort    10.111.122.247   <none>        3000:31399/TCP   4m4s
           kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          6d17h
           ```
           ese `31399` es el puerto que nos expone al exterior, es decir si en local queremos probar ese microservicio, tendriamos que llamar algo como: `http://localhost:31399/` y ya deberia estar andando sin problemas. 3000 es el puerto interno para ese microservicio
   2. paso para crear un servicio de tipo `clusterip` de acceso solo internamente no desde fuera de la red
      1. debemos ubicarnos en la carpeta del microservicio o servicio que vamos a trabajar ejemplo `k8s/tienda/templates/nats/`,
      2. ya en la carpeta, debemos ejecutar el comando: `kubectl create service clusterip nats --tcp=4222 --dry-run=client -o yaml > service.yml`, esto nos generara un archivo de llamado `service.yml`. recordemos que `nats` es el nombre del servicio, y que `4222` es el puerto que necesitamos, todo depende del puerto que necesitemos. IMPORTANTE debemos validar que el archivo generado sea `UTF8` para que todo ande bien
      3. ahora debemos ir a la carpeta `k8s/tienda/` y alli ejecutar el comando `helm upgrade tienda .` para que actualice los cambios
      4. para ver si realmente esta corriendo, ejecutamos en la terminal `kubectl get services` alli podemos ver algo como:
           ```
            NAME              TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
            cliente-gateway   NodePort    10.111.122.247   <none>        3000:31399/TCP   47m
            kubernetes        ClusterIP   10.96.0.1        <none>        443/TCP          6d18h
            nats              ClusterIP   10.110.85.156    <none>        4222/TCP         44s
           ```
           NATS es el que estamos creando de ejemplo, podemos acceder a el solo internamente entre servicios. como lo sabemos? viendo en que type `ClusterIP`, y que en la columna ports dice que internamente maneja el puerto que le indicamos, pero que desde fuera solo tiene acceso mediante `TCP`
6. si un microservicio necesita variables de entorno, creando un `secret`, si queremos ver cuales secret tenemos creados, podemos listarlos con el comando `kubectl get secrets` lo podemos hacer yendo a la carpeta `k8s/tienda/templates/mi-microservicio/` y alli ejecutamos el co mando. `kubectl create secret generic <nombre> --from-literal=key=value` sirve para crear variables de entorno o secret en kubernetes. esto permite ejemplo agrupar variables de entorno por microservicio o algo asi, ejemplo el `nombre` seria `pedidos-microservice-secret` y key es el nombre de esa variable y value es el valor. un secret puede tener varias key y values. EJEMPLO!! `kubectl create secret generic auth-microservice-secrets --from-literal=jwt_secret="mi_clave_secreta_para_jwt_PRODUCCION"  --from-literal=database_url="mongodb+srv://mi-usuario:mi-password@auth-microservice-curso.q4tshyd.mongodb.net/auth_microservice_db?retryWrites=true&w=majority"`. si quiero listar los secrets `kubectl get secrets`
   ```
   auth-microservice-secrets       Opaque               2      2m4s
   pedidos-microservice-secrets    Opaque               1      42h
   sh.helm.release.v1.tienda.v22   helm.sh/release.v1   1      41h
   sh.helm.release.v1.tienda.v23   helm.sh/release.v1   1      41h
   ```
   si quiero ver el contenido de uno en especifico:  `kubectl get secrets auth-microservice-secrets -o yaml`. algunos o todos los valores salen convertidos a base64

## comandos basicos

1. `kubectl get pods`:  para listar los pods que existen, alli podemos obtener el nombre del pod especifico que queremos revisar
2. `kubectl get services` listar todos los servicios que se pueden acceder desde fuera del contenedor mediante http
3. `kubectl get secrets` lista todos los secrets, los creados y los internos que maneja automaticamente kubernetes
4. `kubectl get secrets auth-microservice-secrets -o yaml` muestra todo lo que tenga un secret por dentro, cada variable y contenido. 'auth-microservice-secrets' es nombre de ejemplo
5. lista el contenido de un secret o variables de entorno en particular MUY UTIL.
6. `kubectl logs cliente-gateway-6d46b958bb-shwcw --previous` SIRVE PARA VER LOS LOGS DE UN POD o microservicio especifico, sirve para ver errores de logica de programacion o lo que tengamos de logs en muestra programacion de netsjs
7. `kubectl describe pods nombre-de-mi-pod`: SIRVE PARA SABER SI TENEMOS UN ERROR EN UN POD O ALGO ASI en un POD. ejemplo de nombre de pod `cliente-gateway-6d46b958bb-shwcw` este nombre varia cada vez que cuando la herramienta lo desea.
8. `kubectl create secret generic <nombre> --from-literal=key=value  --from-literal=key2=value2` para crear variables de entorno o secret en kubernetes. esto permite ejemplo agrupar variables de entorno por microservicio o algo asi, ejemplo el nombre seria `pedidos-microservice-secret` y key es el nombre de esa variable y value es el valor. un secret puede tener varias key y values. ejemplo real de crear varias key al mismo tiempo para un microservive `kubectl create secret generic auth-microservice-secrets --from-literal=jwt_secret="mi_clave_secreta_para_jwt_PRODUCCION"  --from-literal=database_url="mongodb+srv://mi-usuario:mi-password@auth-microservice-curso.q4tshyd.mongodb.net/auth_microservice_db?retryWrites=true&w=majority"` tambien se puede eliminar si se desea, escribiendo `kubectl delete secret auth-microservice-secrets`. en la raiz del proyecto hay un archivo llamado `K8s.README.md` con mas documentacion sobre todo este tema

## comando utilizados para crear los deploy de cada microservice

dentro de la carpeta `k8s/tienda/templates/` creo una carpeta con el nombre del microservicio y alli ejecuto los comandos y, debemos verificar que el archivo generado sea `UTF8` en vs code se ve si es o no, tambien lo podemos convertir a `UTF8` si lo necesita. comandos:

1. cliente-gateway: `kubectl create service nodeport cliente-gateway --tcp=3000 --dry-run=client -o yaml > service.yml`
2. nats: `kubectl create deployment nats --image=nats  --dry-run=client -o yaml > deployment.yml`
3. productos: `kubectl create deployment productos-microservice --image=josueperezf/productos-microservice:latest    --dry-run=client -o yaml > deployment.yml`
4. pedidos: `kubectl create deployment pedidos-microservice --image=josueperezf/pedidos-microservice:latest    --dry-run=client -o yaml > deployment.yml`
5. pagos: `kubectl create deployment pagos-microservice --image=josueperezf/pagos-microservice:latest    --dry-run=client -o yaml > deployment.yml`

## ejemplo de crear secrets

comando para crear un secret para el microservicio de pagos, debemos movernos a la carpeta correspondiente `k8s/tienda/templates/mi-microservicio/`, y alli ejecutar el con los valores reales claramente.
`kubectl create secret generic pagos-microservice-secrets --from-literal=stripe_secret_key=abc --from-literal=stripe_endpoint_secret=123`

## ejmplo de crear un servicio

`kubectl create service nodeport pagos-microservice --tcp=3003 --dry-run=client -o yaml > service.yml`
este es para permitir que nuestro microservicio pueda escuchar peticiones desde fuera del cluster, es decir, con su propia url y puerto

actualmente solo dos microservicios tienen servicios externos o de tipo `nodeport` el gateway y el de pagos. gateway porque es el que recibe todas las peticiones desde el frontend, y el pagos por que es hibridos y recibe peticiones de stripe que es el que le informa si el cliente a pago. cuando lo pruebe puede que no ande o algo asi, pero es porque expira lo de stripe o necesita una url externa productiva para probar, y como este proyecto es solo academico, no podemos probarlo al 100%

PARA MAS INFORMACION ABRIR EL ARCHIVO QUE ESTA EN LA RAIZ DEL PROYECTO, LLAMADO <K8s.README.md>