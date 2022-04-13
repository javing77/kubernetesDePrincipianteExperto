# Kubernetes, de principiante a experto

_Apuntes del curso UDEMY [Kubernetes, de principiante a experto](https://www.udemy.com/course/kubernetes-de-principiante-a-experto/)  desde hace 2 años no recibe actualización el material. :(  vamos a ver como nos va_

### Fallas
1. Módulo 5 video No. 24 no se puede crear un pod como explica en el video.
2. No se puede conectar a los puertos del pod creado solo por la linea de comando, los puertos no quedan expuestos incluso dentro del mismo host. Para esto se debe exponer un servicio en el cluster
(un ingress para hacer que se vea por fuera del cluster )

## Módulo 1 
## Módulo 2
## Módulo 3
## Módulo 4

## Módulo 5

Validar que minikube esté correindo

```
minikube status
```

Validar que el kubectl esté corriendo
```
kubectl get pods
```

Ver la versión del kubectl
```
kubectl version
```

La configuración de kubectl se almacena en ~/.kube/config o tambien se puede el siguente comanado
```
kubectl confing view
``` 
Para información del cluster y el control plane
```
kubectl cluster-info
```

### DashBoard Minikube - Acceso Remoto

El dashboar des una forma sencilla y administar, Para acceder al dashboard de minikube de forma remota (Desde un Mac/Linux). Para ver las caracteristicas habilitadas para el uso podemos usar

```
minikube addons list
```

Para habilitar las metricas en el servicio

```
minikube addons enable metrics-server
```


En la maquina virtual ejecutar
1. Iniciar el dashboad
```
minikube dashboard &
```

2. Activar el acceso al dashboar y quitqar las reglas
```
kubectl proxy --address='0.0.0.0' --disable-filter=true & 
```

Desde la mac
3. Para acceder remotamente usando ssh se mapea de la siguiente fomra
```
ssh -L 12345:localhost:8001 javing77@192.168.0.22
```
Esto hace un puente entre los puertos remoto asiando unos puertos temporales en este caso 12345, Para acceder al sitio hay usar 

```
http://localhost:12345/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/pod/default/doscont?namespace=default
```



### Crear un pod
_Nota: El instructor usa kubectl run --generator la cual fue depreciado y eliminado, en este caso lo que se debe usar es run la forma mas sencilla de crear un pod sin usar un archivo yaml_


el nombre del pod debe ser en minuscula de lo contrario da error
```
# Crea un pod de una imagen de nginx con el nombre nginx
kubectl run Prueba --image=nginx --restart=Never

# Ver el estado del pod:
kubectl get pods
# kubectl get pods -o wide

# Ejemplo del resultado
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          2m8s

```

### Crear un DEPLOYMENT -> Según fuente de Linux Fundation
```
kubectl create deployment mynginx --image=nginx:1.15-alpine
```

Validar 
```
kubectl get deploy,rs,po -l app=mynginx
```

Incrementar el número de replicas
```
kubectl scale deploy mynginx --replicas=3
```


### Describir un pod
En caso que se desee ver los eventos que han pasado en un pod, lo cual ayuda identicar errores del porque un pod no se creó

```
kubectl describe pod nginx
```

### Obtener información detallada de un pod
Solo se debe agregar el parámetro -o yaml al get pod. Con esto se obtiene el manifiesto

```
kubectl get pod nginx -o yaml
```

Para ver el listado de operaciones que podemos hacer con kubectl podemos ejecutar
```
kubectl api-resources
```

### Eliminando un Pod
Se puede eliminar varios pods al mismo tiempo, solo se debe agregar los nombres
```
kubectl delete pod nginxx
```
### DOCKER y los PODS
Como se está usando docker como motor podemos ver los contenedores/pods creados por kubectl.
Pero estos contenedores no se deben administrar por docker si no por el kubectl
```
docker ps 
```

### Ingresar aun pod
Aunque pocas veces se ingresa directamente a un pod. Para salir usar el comando exit
```
kubectl exec -ti nginxpruebas -- bash
```

### Ver los logs de pod
```
kubectl logs nginxpruebas
```

### Manifiestos
Es un archivo en yaml que describe lo que debe contener el pod a crear. 
apiVersion : Podemos usar kubectl api-versions para ver las versiones que podemos usar. 

```
Kind: kubectl api-resources | grep Pod
metada: name: es el nombre del contenedor
spec: Se indican los contenedores que se quieren crear y sus diferentes caracteristicas
```
cuando se usan --- es para separar los recursos 

Aplicando un manifiesto

```
kubectl apply -f pythonserver.yaml
```


### Eliminando pods usando manifiestos
Cuando se usa el archivo yaml para borrar se borran todos los recursos los que estan establecidos en dicho archivo.
```
kubectl delete -f nginx01.yaml
```

### Creando dos contenedores por POD
Ver archivo modulo5/pod/pythonServerDosContainers.yaml

Para ver el log de un pod que tiene dos o mas contenedores  se le agrega el parámetro -c al logs

```
kubectl logs doscont -c cont1
```

### Labels
Es metada que se le aplica a los pods, ayuda a indenticar , estois van dentro de la metadata



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

