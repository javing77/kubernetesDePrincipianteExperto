
# EDX--> Linux Fundation 

## Capitulo 8.
### Kubernetes Object Model
Kubernetes tiene un modelo de objetos muy rico que representa diferentes entidades persistentes en el clúster de Kubernetes. Esas entidades describen

1. Lo que aplicaciones conteinizadas estan corriendo.
2. Los nodos donde las aplicaciones conteinizadas son desplegados.
3. Consumos de  recursos de la aplicación.
4. Políticas adjuntas a las aplicaciones como por ejemplo: Políticas de reinicio/actualización a fallas, etc.

Cada uno de estos objetos son declarados en la sesion _spec_  

Un ejemplo de Objetos en Kubernetes son: Pods, ReplicaSets, Deployments, Namespaces, etc. Como se mencionó anteriormente las configuraciones que tendras estos objetos van el campo **spec**

Ejemplo de la implementación de un manifiesto de configuración de un Objeto en Yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.15.11
        ports:
        - containerPort: 80
```

apiVersion: Este es el primer campo y obligatorio. Especifica el API endpoint en el API Server el cual debe coincidir con la versión existente de ese objeto **kubectl api-versions** .
kind: Segundo campo obligatorio, Especifica el tipo objeto para el ejemplo anterior es un Deployemnt  **kubectl api-resources | grep Pod**
metadata: Tercer campo obligatorio , indica información basica del objeto como _name_ , _labels_, _namespace_.
spec: Es el cuarto campo obligatorio (En el ejemplo de referencia se unan dos spec  y spec.template.spec.) el campo _spec_ indica el inicio del bloque de definión del estado deseado del objeto. En el ejemplo se está indicando 3 replicas de un pod. Estos pod son creados usando un template.  (spec.template.spec). Este pod crea los containers de una imagen de docker de nginx:1.15.11 .

### POD
//TODO: Cargar imagen de notion.
Los pods por definición son efimeros y no tienen la capacidad de _self-heal_ . Esta es la razón por que se usan con controllers que manejan replicacion, tolerancia a fallos, _self-healing_ (ReplicationController, ReplicaSets)

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: nginx
spec:
  containers:
  - name: nginx
    image: nginx:1.15.11
    ports:
    - containerPort: 80
```

### Labels
Son pares de llave-valor (key-value) en los objetos de Kubernetes (e.g. Pods, ReplicaSets, Nodes, Namespaces, Persistent Volumes). Los labels son usados para organizar un subconjunto de objetos. Los labels no son indicadores unicos es decir pueden exister multiples objetos con el mismo label
Los Controllers usan los label para agrupar logicamente difernetes objeton, en vez de usar nombre de objetos o ids. Tambien se pueden usar una Label Selectors para hacer una mejora agrupación

//TODO: Pegar imagen de notion

### ReplicationController
Es un controller no recomendado para su uso --> Averiguiar el porqué

### ReplicaSets 
Es la evolución "next-generation" de un ReplicationController. Implementa replicación y "self-healing". Utilizando ReplicaSets podemos escalar el numero de Pods que  esta corriendo una aplicacion de container imagen específica. 

//TODO: PEGAR IMAGEN

### Deployments
El deploycontroller es parte del master node's controller manager y como contralador tambien se asegura que el estado actual siempre conincida con el que deseado (configurado). Permite realizar actualizaciones y reversiones de aplicaciones sin interrupciones. Administra directamente sus replicaSets  para el escaado de aplicaciones.

//TODO: PEGAR IMAGEN

### Namespaces

Si varios usuarios y equipos usan el mismo clúster de Kubernetes, podemos dividir el clúster en subclústeres virtuales mediante espacios de nombres. Los nombres de los recursos/objetos creados dentro de un espacio de nombres son únicos, pero no entre los espacios de nombres del clúster.

```
kubectl get namespace
# Resultado
default           Active       11h
kube-node-lease   Active       11h
kube-public       Active       11h
kube-system       Active       11h

```

## Modulo 9 - Authentication, Authorization, and Admission Control
Para acceder a los recursos de Kubernetes o a los objetos del cluster, necesitamos acceder aun endpoint del servidor de la API.  Cada petición pasa por los siguientes etapas de control de acceso.


    Authentication
        Logs in a user.
    Authorization
        Authorizes the API requests submitted by the authenticated user.
    Admission Control
        Software modules that validate and/or modify user requests based.

Kubernetes no tiene un objeto especial para los USER. Pero a pesar de eso puede almacenar los usernames para la fase de Autenticación .

### Modulo 10 - Services
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    app: frontend
  ports:
  - protocol: TCP
    port: 80
    targetPort: 5000
```


### Modulo 11 - Deploying a Stan-Alone application 

1. Crear un archivo Yaml con la siguiente contendido: webserver.yaml

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: webserver
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
        ports:
        - containerPort: 80
```

2. Hacer el deployment con kubectl

```
kubectl create -f webserver.yaml
``` 

3. Validar la creacion de los pods y de los replicaSets

```
kubectl get pods,replicasets
``` 

4. Crear un Service  de tipo NodePort el cual abre un puerto estatico para todos los nodos. Si nos conectamos a este puerto desde cualquier nodo,  El proxi nos redireccionara al ClusterIp service.

webserver-svc.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: web-service
  labels:
    run: web-service
spec:
  type: NodePort
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx 
```

5. Crear el servicio usando kubectl y validar la creación

```
kubectl create -f webserver-svc.yaml
kubectl get service 
```

_**Nota**_ : Una forma directa de exponer un deployment es 
```
kubectl expose deployment webserver --name=web-service --type=NodePort
```

6. Acceder a la aplicación.  La aplicación está correindo dentro de la VM de minikube, para cceder a la aplicación primero debemos asignar una ip 
```
minikube ip
```

Finalmente podemos hacer un curl ip:puerto . El puerto es el que sale en el kubectl get services.


### Liveness and Readiness Probes
https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/

## Liveness

Cuando un conteiner dentro de un pod se encuentra en ejecución pero la aplicación que estaba correiendo dentro del conteiner de repente se detiene o deja de responder, el conteiner no es funcional para nosotros. En este caso se recomienda reiniciar el conteiner.
Para esto podemos usar Liveness Probe. El cual valida la salud de las aplicaciones y si la validación falla entonces kubectl reinicia automaticamente la aplicación.

Liveness puede ser configurado por:
    1. Liveness command
    2. liveness Http request
    3. TCP Liveness probe

## Modulo 12 Kubernetes Volume Management 
Tipos de volumnes:
    emptyDir: Está ligado con la vida de un pod, si un pod finaliza el contendido de emptyDir será eliminado completamente.
    hostPath: Podemos compartir un directorio de un host al pod, si el pod finaliza el contendido seguirá en la carpeta del host.
    gcePersistentDisk: Se puede montar un disco de Google Compute Engine.
    awsElasticBlockStore: Podemos montar un disco de AWS EBS Volume
    azuereDisk: Podemos montar un disco de Microsoft Azure File Volume.
    cephfs: Podemos montar este volumen en el pod y cuando el pod finalice este tipo de volumen aun permanecerá.
    nfs: Volumnes tipo nfs
    iscsi: Volumen tipo icesi
    secret: Podemos alamacenar informacion sensible 
    configMap: Almacena información de configuración.

persistentVolume:  (PV) Es una capa de stración 
Container Storage Interface: (CSI)