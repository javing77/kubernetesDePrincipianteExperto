# Kubernetes, de principiante a experto

_Apuntes del curso UDEMY [Kubernetes, de principiante a experto](https://www.udemy.com/course/kubernetes-de-principiante-a-experto/)  desde hace 2 años no recibe actualización el material. :(  vamos a ver como nos va_

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
Ver archivo Módulo5/pod/pythonServerDosContainers.yaml

Para ver el log de un pod que tiene dos o mas contenedores  se le agrega el parámetro -c al logs

```
kubectl logs doscont -c cont1
```

### Labels
Es metada que se le aplica a los pods, ayuda a indenticar , estois van dentro de la metadata


## Módulo 6 - ReplicaSets


### Que es un ReplicaSet
Un ReplicaSet se define con campos, incluyendo un selector que indica cómo identificar a los Pods que puede adquirir, un número de réplicas indicando cuántos Pods debería gestionar, y una plantilla pod especificando los datos de los nuevos Pods que debería crear para conseguir el número de réplicas esperado. Un ReplicaSet alcanza entonces su propósito mediante la creación y eliminación de los Pods que sea necesario para alcanzar el número esperado. Cuando un ReplicaSet necesita crear nuevos Pods, utiliza su plantilla Pod.

El enlace que un ReplicaSet tiene hacia sus Pods es a través del campo del Pod denominado metadata.ownerReferences, el cual indica qué recurso es el propietario del objeto actual. Todos los Pods adquiridos por un ReplicaSet tienen su propia información de identificación del ReplicaSet en su campo ownerReferences. Y es a través de este enlace cómo el ReplicaSet conoce el estado de los Pods que está gestionando y actúa en consecuencia.

Un ReplicaSet garantiza que un número específico de réplicas de un pod se está ejecutando en todo momento. Sin embargo, un Deployment es un concepto de más alto nivel que gestiona ReplicaSets y proporciona actualizaciones de forma declarativa de los Pods junto con muchas otras características útiles. Por lo tanto, se recomienda el uso de Deployments en vez del uso directo de ReplicaSets, a no ser que se necesite una orquestración personalizada de actualización o no se necesite las actualizaciones en absoluto.

En realidad, esto quiere decir que puede que nunca necesites manipular los objetos ReplicaSet: en vez de ello, usa un Deployment, y define tu aplicación en la sección spec.

![RS](./KubernetesDePrincipianteAExperto/modulo6-rs/RS.png)

[Más información de ReplicaSets](https://kubernetes.io/es/docs/concepts/workloads/controllers/replicaset/)

[Ejemplo-RS](KubernetesDePrincipianteAExperto/modulo6-rs/rs-01.yaml)

```
kubectl apply -f modulo6-rs/rs-01.yaml
kubectl describe rs/rs-frontend
kubectl get po -l tier=frontend
```

Se puede actulizar el archivo y volver aplicarlo y va a realizar esa configuración sobre los pods estabalecidos

Para saber quien es el dueño (owner) de un pod, podemos generar información del pod con los parámetros -o yaml

```
kubectl get po rs-frontend-xt7jr -o yaml > modulo6-rs/informacionPod1.yaml
```
[modulo6/informacionPod1.yaml](KubernetesDePrincipianteAExperto/modulo6-rs/informacionPod1.yaml)


a consultar el archivo generado podemos encontrar en la parte de la metada la si

```
metadata:
  creationTimestamp: "2022-04-21T22:29:02Z"
  generateName: rs-frontend-
  labels:
    tier: frontend
  name: rs-frontend-xt7jr
  namespace: default
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: rs-frontend
    uid: 60b36c25-41c4-462f-b213-392671eee269

```
El campo uid especifica quien es el owner, esto se puede validar haciendo un get rs rs-frontend -o yaml

_Nota: En caso que un pod no tenga de tenga ownerReference pero tenga el mismo label de un RS, el RS los hereda, no importa que sean del mismo tipo_

### Escalar manualmente un ReplicaSet

  
  ```
  kubectl scale  rs rs-frontend --replicas=3
  ```

### Asignar un label a un pod existente.

```
kubectl label pod pruebapod1 tier=frontend
kubectl get po --show-labels pruebapod1
```

### Validar la nota anterior 
1. creamos una archivo yaml [modulo6-rs/rs-02.yaml](KubernetesDePrincipianteAExperto/modulo6-rs/rs-02.yaml) este manifiesto es de nuevo RS
2. Creamos un pod manualmente.
    ```
    kubectl run pruebapod1 --image=nginx:alpine
    kubectl label pod pruebapod1 tier=frontend
    ```
3. Validamos con un get pruebapod1 -o yaml que este no tenta metada.ownerReferences

4. Aplicamos el manifiesto 
```
kubectl apply -f modulo6-rs/rs-02.yaml
```

5. Ahora volvemos hacer un get prubeapod -o yaml y veremos que se agregó metada.ownerReferences

```
  ownerReferences:
  - apiVersion: apps/v1
    blockOwnerDeletion: true
    controller: true
    kind: ReplicaSet
    name: rs-frontend-2
    uid: a3dcf3ca-576a-4ccd-bac0-e8a43fda71a2
```


**NOTA: Por este motivo no se deberia crear pods de forma manual **


### Problemas de ReplicaSets.
En caso de realizar un cambio del manifiesto en la parte de los template ejemplo se cambia la versión de la imagen y aplicamos el manifiesto no va a ocurrir nada. Esto porque el RS solo mira que se cumpla la cantidad de pods solicitados. Excepto a que se force eliminando uno de los pods.

## Módulo 7 - Deployments

## Deployment

Un controlador de Deployment proporciona actualizaciones declarativas para los Pods y los ReplicaSets.

Cuando describes el estado deseado en un objeto Deployment, el controlador del Deployment se encarga de cambiar el estado actual al estado deseado de forma controlada. Puedes definir Deployments para crear nuevos ReplicaSets, o eliminar Deployments existentes y adoptar todos sus recursos con nuevos Deployments.

[Más información de Deployments](https://kubernetes.io/es/docs/concepts/workloads/controllers/deployment/)

### Creando un deployment

Un archivo de manimiento sus promera lineas deben ser apiVersion: apps/v1 kind: Deployment.

[Ejemplo-Deployment](KubernetesDePrincipianteAExperto/modulo7-deploy/deploy01.yaml)

1. Realizar el deploy del Deployent
```
kubectl apply -f modulo7-deploy/deploy01.yaml
kubectl get deploy
```

2. Validar los labels que tenga este deployment
```
kubectl get deployment --show-labels nginx-deployment
```

3. Ver el estados del deployment
```
kubectl rollout status deployment nginx-deployment
```

4. Validar  la existencia de los pods con --show-labels
```
kubectl get po --show-labels
```

5. A diferencia cuando se trabajó solo con el RS este no tenia un metada.ownerReferences pero como este RS fue creado con un Deployment este si cuenta con un metada.ownerReferences

```
kubectl get rs nginx-deployment-b9d465bf -o yaml
```

### Actualizar un Deployment para que este actualice sus PODs

Para este caso se actualizara la version del nginx pasandola a ser alpine
[modulo7-deploy/deploy02.yaml](KubernetesDePrincipianteAExperto/modulo7-deploy/deploy02.yaml)

```
kubectl apply -f modulo7-deploy/deploy02.yaml
```

Al aplicar este nuevo manifiesto, causará que los pod inicien a actualizarce uno a uno.
Para ver en tiempo real lo que está pasando 

```
kubectl rollout status deployment nginx-deployment
```

Obtener mas información del proceso del deployement
```
kubectl describe deployment nginx-deployment
```

![RollOut](./KubernetesDePrincipianteAExperto/modulo6-rs/RS.png)
[Más Información de RollOut](https://kubernetes.io/docs/tutorials/kubernetes-basics/update/update-intro/)

### Ver los rollout hechos a un deployment
Estos son básicamento nos dá información de los RS  que se han creado
```
kubectl rollout history deployment nginx-deployment
```

Esto se hace para poder regresar aun estado anterior. Por defecto se puede mantener hasta 10 de historico. Esto se puede cambiar en archivo de manifiesto con el parámentro __spec. revisionHistoryLimit: 5__

Es practico usar Change-Cause para que actualice los history quede el caso de actualizacion hay 3 formas

1. Usando el parámetro --record  el cuarda guarda cual fue el comando que se ejecutó **NOTA: ESTA OPCION ESTA MARCADA COMO DEPRECATED**
```
kubectl apply -f modulo7-deploy/deploy02.yaml --record
kubectl rollout history deployment nginx-deployment
``` 

2. Usnado metadata.annotations.kubernetes.io/change-cause: Ejemplo

[modulo7-deploy/deploy03.yaml](KubernetesDePrincipianteAExperto/modulo7-deploy/deploy03.yaml)

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: front
  annotations:
    kubernetes.io/change-cause: "Actualizar el puerto"
```

Aplicando el manifiesto y viendo el history del rollout

```
kubectl apply -f modulo7-deploy/deploy03.yaml
kubectl rollout history deployment nginx-deployment
```

3. Tambie se podría aplicar la anotación directamente al deployment. __kubectl annotate deployment/nginx-deployment kubernetes.io/change-cause="image updated to 1.16.1"__

### Ver los cambios hechos en un rollout

```
kubectl rollout history deployment nginx-deployment --revision=3
```

### HACIENDO UN ROLLBACK

Un rollback teoriacamente se puede hacer es porque la aplicación no está ejecutando como esperabmos que se hiciera o bien sea inestable .

Para este ejercicio se apicará una actlización al deploy pero colocando una imagen que no exista.
así generando error al momento de crear el pod

[modulo7-deploy/deploy04.yaml](KubernetesDePrincipianteAExperto/modulo7-deploy/deploy04.yaml)

```
kubectl apply -f modulo7-deploy/deploy04.yaml
kubectl get pods
```

    NAME                                READY   STATUS         RESTARTS   AGE
    nginx-deployment-68fd89c797-d2wx2   1/1     Running        0          13m
    nginx-deployment-68fd89c797-sv46c   1/1     Running        0          13m
    nginx-deployment-68fd89c797-wtw5h   1/1     Running        0          13m
    nginx-deployment-7d448bf6c8-ff4x4   0/1     ErrImagePull   0          3m14s

Para volver a un estado anterior
```
kubectl rollout undo deployment nginx-deployment --to-revision=3
```

# Módulo 8 Servicios
es el objeto de la API de Kubernetes que describe cómo se accede a las aplicaciones, tal como un conjunto de Pods, y que puede describir puertos y balanceadores de carga.

Con Kubernetes no necesitas modificar tu aplicación para que utilice un mecanismo de descubrimiento de servicios desconocido. Kubernetes le otorga a sus Pods su propia dirección IP y un nombre DNS para un conjunto de Pods, y puede balancear la carga entre ellos.
Motivación
[Más información] (https://kubernetes.io/docs/concepts/services-networking/service/)


Crear un archivo que contenga un deployment y simultaneamente el Servicie 

[modulo8-svc/svc-01.yaml](./KubernetesDePrincipianteAExperto/modulo8-svc/svc-01.yaml)

Aplicar el manifiesto:

```
kubectl apply -f modulo8-svc/svc-01.yaml
kubectl get services
kubectl get endpoints
```

### Ejecutando un pod temporar
```
kubectl run --rm -ti podtest3 --image=nginx:alpine -- sh
```

### Tipo de Servicios : ClusterIp
Es una ip virtual que kubernetes asingna al servicio y kubernetes se encarga que sea fija,
esta ip es interna. (solo absesible dentro del cluster).

Para definir el tipo de servicio se hace dentro del spec  "spec.type: CluserIp"

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
  labels:
    app: front
spec:
  type: CluserIp
  selector:
    app: front-nginx
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 80
```

### Tipo de Servicios : NodePort
Permite exponer el servicio por fuera del Nodo, una vez creado un servicio NodePort.
User  -----> Nodeport ------>   ClusterIp    ----> PODs

[Mas Información](https://kubernetes.io/docs/concepts/services-networking/service/)

[modulo8-svc/svc-np-03.yaml](KubernetesDePrincipianteAExperto/modulo8-svc/svc-np-03.yaml)

Aplicar el manifiesto:

```
kubectl apply -f modulo8-svc/svc-np-03.yaml
```

Para accder al recurso expuesto se hace a través de la ip del node de minikube esto funciona solo dentro de la maquina virtual.
```
kubectl get nodes -o wide
```

Para exponer el servicio por fuera de la máquina virtual
```
kubectl port-forward --address 0.0.0.0 service/my-service-backed 30227:8080
```

### Tipo de Servicios : LoadBalancer
Crea un servicio de Balanceador externo usando plataformas como AWS, AZUERE, GCP

# Módulo 9 - Goland

1. Crear el achivo main.go el cual va a ser un webservice rest básico en golang
[main.go](./KubernetesDePrincipianteAExperto/modulo9/backend/main.go)

2. Descargar la imagen de golang desde docker
```
docker pull golang
```

3. Crear un container que  va a disponibilizar el servicio : Esto es para la fase de desarrollo de la aplicación.

```
docker run --rm -dti -p 9090:9090 -v $PWD/modulo9/backend/:/go  --name gonlangsvc  golang bash
```

4. Accder al contenedor de docker
```
docker ps 
docker exec -ti id_contenedor bash
```

5. Crear una RESTAPI con GO
[main.go](./KubernetesDePrincipianteAExperto/modulo9/backend/main.go)

6. Crear un Dockerfile que construya una imagen para correr el main.go
[Dockerfile](./KubernetesDePrincipianteAExperto/modulo9/backend/Dockerfile)

7. Construir la imagen
```
docker build -t javing77/k8s-hands-on-golang .
docker push javing77/k8s-hands-on-golang
```

9. Correr la imagen previamente create
```
docker run -d -p 9090:9090 --name golang-hands-on k8s-hands-on-golang
```

10. Validar que el servicio esté ejecutandose correctamente.
```
curl localhost:9090
```

11. Eliminar el contenedor creado
```
docker rm -f golang-hands-on
``` 

12. Crear el manifiesto para la imagen 

# Modulo 10 Namespace
 ![Namespaces](KubernetesDePrincipianteAExperto/modulo10/namespaces.png)


Brinda un scope 'Limite' de objectos (Deploy, SVC, PO, RS) entre otros objetos, Ejemplo un ambiente DEV, QAD dentro un mismo cluster pero separción logica.

Namespaces are a way to divide cluster resources between multiple users


  [Mas Informacion](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces/)


Obtener namespace  

```
kubectl get namespaces
```

Obtener pods por namespaces

```
kubectl get  pods --namespace default
```

Mostrar los objetos que tiene un namespace 
```
kubectl get namespaces
kubectl get all -n lfs158
```

## Creando un namespace
1. Linea de comando 

```
kubect create namespace prueba1
```

2. Por yaml

[namespace.yaml](./KubernetesDePrincipianteAExperto/modulo10/namespace.yaml)

```
kubectl apply -f modulo10/namespace.yaml
kubectl get namespaces --show-labels
```

3. Creando un pod dentro del namespace
```
kubectl run  pruebanamespace --image=nginx:alpine --namespace kubernetes-de-principiante-a-experto
kubectl get po --namespace kubernetes-de-principiante-a-experto
```

## Creando namespaces con yaml multiples ambientes
[namespaces2](./KubernetesDePrincipianteAExperto/modulo10/namespace02.yaml)

```
kubectl apply -f modulo10/namespace02.yaml
```

## Crear un NS, DEPLOY y SVC  : Valindando como funcionan los DNS

Los namespaces se agregan en metadata.namespace de los diferentes objetos
[ns-dns-03.yaml](./KubernetesDePrincipianteAExperto/modulo10/ns-dns-03.yaml)

### Consumiendo un servicio desde un pod que no está en el mismo namespace

Se debe pasar el FQDN (Fully Qualified Domain Name)

```
curl nameSvc.nameNS.svc.cluster.local
curl go-backed-hand-on.ci.svc.cluster.local:9090
```

## Trabajando con el context

Trabajar con contextos nos ahorra estar pasando los parametros -ns para consultar los diferentes Objectos

1. Validar el contexto en el que nos encontramos.
```
kubectl config current-context
kubectl config view
```

2. Crear un contexto , para esto hay que pasar parámetros como NombreContexto, namespace  y el usuario. Para los datos se pueden usar los datos generados por los anteriores comandos
```
kubectl config set-context ci-context --namespace=ci --cluster=minikube --user=minikube
```

3. Cambiarnos de contexto.
```
kubectl config use-context ci-context
```

# Modulo11: Limites 
## Limitar el uso de la Ram
![Linites&Requests](./KubernetesDePrincipianteAExperto/modulo11/Limites.png)

1. Ejemplo de uso de limites.

[LimitRam]('./KubernetesDePrincipianteAExperto/modulo11/limites.yaml')

2. Cuando un deploy solicita una cantidad de recursos, pero este se eleva mas de los establecidos, los pod se reinician para ver si estos pueden bajar.

[LimitRam2]('./KubernetesDePrincipianteAExperto/modulo11/limitesRam2.yaml')

3. Cuando un deploy solicita mas recursos de los que se cuentan en un node este quedaran en Pending pues no tiene recursos suficiendes para 
inicializarlo

[LimitRam]('./KubernetesDePrincipianteAExperto/modulo11/limitessRam3.yaml')

Para visualizar que esta causnado el error podemos hacerlo a través de un describe

```
kubectl describe po memory-demo
```
## Limitar uso de CPU

1. Adiferencia de los limites en ram, los pod no se les permite  pasar el limite establecido de cpu por esta razón no se reinician.
[LimitCpu](./KubernetesDePrincipianteAExperto/modulo11/limit-cpu.yaml)

Esto se puede validar usuando el describe del nodo
```
kubectl describe node minikube
```

El describe del node tambien en la parte de Capacity.cpu nos indica cuanta cpu pdemos contar.

2. Cuando se solicta la creación de un pod con mas cpu de la requerida. tampoco es creado.
[LimitCPU2](./KubernetesDePrincipianteAExperto/modulo11/limit-cpu2.yaml)

## QoS
Es una propiedad que se le asigna a un pod


    Guaranteed  -> Cuando el limit es igual al request
    Burstable   -> Cuando el limit es mayor al request
    BestEffort  -> Cuando no se define ni Guaranteed ni Burstable : Son los mas peligrosos debido a que no tienen limites.


# Modulo 12: LimitRange - Aprender a controlar el uso de recursos a Nivel de objetos

Aplicar restricciones de asignación de recursos, los administradores de clústeres se aseguran del cumplimiento del consumo de recursos por espacio de Namespace

Un LimitRange permite aplicar las siguientes políticas:
    Imponer restricciones de requisitos de recursos a Pods o Contenedores por Namespace.
    Imponer las limitaciones de recursos mínimas/máximas para Pods o Contenedores dentro de un Namespace.
    Especificar requisitos y límites de recursos predeterminados para Pods o Contenedores de un Namespace.
    Imponer una relación de proporción entre los requisitos y el límite de un recurso.
    Imponer el cumplimiento de las demandas de almacenamiento mínimo/máximo para Solicitudes de Volúmenes Persistentes.

[Más Información](https://kubernetes.io/docs/concepts/policy/limit-range//)


Es un objeto que permite controlar limites a nivel de objetos, colocando limites por defectos. El limitRange opera en los objetos del namespace donde ha sido creado.

crendo un Namespace y un LimintRange:
[limit-range-mem-cpu](./KubernetesDePrincipianteAExperto/modulo12/limitrange-mem.yaml)

```
kubectl get limitrange -n dev-kpe
```

Describir el recurso para ver los limites que contienen este limitrange, junto con un container pero a este no se le asinga limites
```
kubectl describe limitrange mem-limit-range -n dev-kpe
```

# Modulo 13: ResourceQuota

ResourceQuota vs Limit Range.

![ResourceQuota](./KubernetesDePrincipianteAExperto/modulo13/ResourceQuota.png)

[Más Información](https://kubernetes.io/docs/concepts/policy/resource-quotas/)

[Archivo ResourceQuta](./KubernetesDePrincipianteAExperto/modulo13/ResourceQuota.yaml)



```
kubectl apply -f KubernetesDePrincipianteAExperto/modulo13/ResourceQuota.yaml
kubectl get quota -n dev-kpe
```

## Limitar el numero de pods

Agregar el spec.hard.pod: "2"

```
apiVersion: v1
kind: ResourceQuota
metadata:
  name: pod-demo
  namespace: doscont-quota
spec:
  hard:
    pods: "2"
```
[Más información](https://kubernetes.io/docs/tasks/administer-cluster/manage-resources/quota-pod-namespace/)

# Modulo 14 Health Checks & Probes

![Probes](./KubernetesDePrincipianteAExperto/modulo14/Probes.png)

Kubelet es el encargado de ejecutar liveness y readiness probes 'dignosticos' para saber sus respectivos estados. 
Estos probes lo puede hacer por:
* [comando](./KubernetesDePrincipianteAExperto/modulo14/liveness.yaml) -> Restorna un valor 0
* [TCP](./KubernetesDePrincipianteAExperto/modulo14/liveness-tcp.yaml)     -> A un puerto
* [HTTP](./KubernetesDePrincipianteAExperto/modulo14/liveness-http.yaml)    -> Petición http auna ruta y valida por códigos de respuesta.

[Más información](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-startup-probes/)

# Módulo 15 : ConfigMaps & Environment Variables 
Creando variables de ambientes usando manifiestos

spec.containers.env.name y spec.containers.env.value
[Variables](./KubernetesDePrincipianteAExperto/modulo15-env/env.yaml)

Leyendo las definiciones  de un pod y dejandolas como variables de ambiente dentro del mismo

spec.containers.env.name
spec.containers.env.valueFrom
spec.containers.env.valueFrom.fieldRef
spec.containers.env.valueFrom.fieldRef.fieldPath

[VariablesRef](./KubernetesDePrincipianteAExperto/modulo15-env/ref.yaml)

```
kubectl exec -ti dapi-envars-fieldref -- sh
```

Una vez adentro del pod ya podemos ver las variables


```
  env
  echo $MY_POD_SERVICE_ACCOUNT
```

## ConfigMap
![ConfigMap](./KubernetesDePrincipianteAExperto/modulo15-env/configmap.png)
[Más Información](https://kubernetes.io/docs/concepts/configuration/configmap/)

Es un objeto utilizado para almacenar datos datos no confidencial, en formato Key: Value
El objetivo de las configmap es separar las configuraciones de las aplicaciones de los pods

Creando un configMap desde un archivo

[Archivo ConfigMap](./KubernetesDePrincipianteAExperto/modulo15-env/configMaps/configMapNgnix.conf)

``` 
kubectl create configmap ngnix-config --from-file=KubernetesDePrincipianteAExperto/modulo15-env/configMapNgnix.conf
``` 

Para obtener los configMaps creados

```
kubectl get cm
```

```
kubectl describe cm ngnix-config
```

Tambien se puede asignar los contenidos de una carpeta a un configMap, Para este caso chrear un
ºº
```
echo HolaMundo > KubernetesDePrincipianteAExperto/modulo15-env/configMaps/index.html
``` 

``` 
kubectl create configmap ngnix-folder --from-file=KubernetesDePrincipianteAExperto/modulo15-env/
``` 

### ConfigMap con Manifiestos
[Archivo](./KubernetesDePrincipianteAExperto/modulo15-env/configMaps/configMaEnv.yaml)

# Módulo 16: Secrets

Es similar es aun configMap pero es para almacenar datos sensibles (Contraseñas, Tokens, Key).
Separando la configuración del  POD
[Más información](https://kubernetes.io/docs/concepts/configuration/secret/)


## Creando un secreto a través de comando

Crear un archivo con los secretos
[Archivo](ernetesDePrincipianteAExperto/modulo16-secrets/secret-files/secrets)

```
kubectl create secret generic mysecret --from-file=KubernetesDePrincipianteAExperto/modulo16-secrets/secret-files/secrets
```

Al hacer un describe no se ve el contenido del secreto.
```
kubectl describe secrets mysecret
```

Para ver el contenido de un secreto debe hacer con  -o yaml y en el campo data: se podran ver el nombre del archivo y el secreto en base64

```
kubectl get secrets mysecret -o yaml
```

Para decodificar este valor podemos hacer un 

```
echo c2VjcmV0bzE9aG9sYQpzZWNyZXQyPWFkaW9z | base64 --decode
```

Por eso de esta forma es insegura

## Creando un secret desde manifiesto

En un manifiesto cuando se hace del data se los valores de las llaves se deben colocar en base64

[Secret](KubernetesDePrincipianteAExperto/modulo16-secrets/secrets01.yaml)

Usando en campo stringData y no data no se utilizaran en base64

## Como gestionar los secrets de fomra segura

No se debería colocar en CVS (git) los secrets porque contiene información sensicle para eso podemos crear manifiestos con variables de ambiente.

La siguiente solución a esto es un poco manual pero muy práctica. vamos a trabajar con el comando envsubst el cual busca en un archivo todo lo que parezca una variable y genera un nuevo archivo con los valores de ambiente del sistema

1.  Crear variables de ambientes
```
export USER2=Pruebas
export PWD2=PruebasPWD
```

2. Crear un manifiesto que tenga los valores de las llaves como variables de ambiente
[SecretsEnv.yaml](KubernetesDePrincipianteAExperto/modulo16-secrets/secretsEnv.yaml)

3. usar el comando envsubst quien recibe < el archivo con las llaves y hace la salida > de un nuevo archivo.
```
envsubst < KubernetesDePrincipianteAExperto/modulo16-secrets/secretsEnv.yaml > KubernetesDePrincipianteAExperto/modulo16-secrets/secretsEnvFinal.yaml
```

## Usando los secrets en un pod Vol
[SecretsVol](KubernetesDePrincipianteAExperto/modulo16-secrets/secrets-vol.yaml)

## Usando los secrets para que sean variables ambiente

[SecretsEnvToPod](KubernetesDePrincipianteAExperto/modulo16-secrets/secretsEnvToPod.yaml)

# Módulo 17 y 18: Volumes

[Más Información](https://kubernetes.io/docs/concepts/storage/volumes/)

[EmptyDir](https://kubernetes.io/docs/concepts/storage/volumes/#emptydir) -> Información para compartir entre containers de un mismo pod. Si un container muere y se recrea uno nuevo la información persiste. Pero en caso que el pod se elimine, la información (Volumen) es eliminada junto con el pod.

[EmptyDir.yaml](KubernetesDePrincipianteAExperto/modulo17-volmunes/emptydir.yaml)
```
kubectl apply -f KubernetesDePrincipianteAExperto/modulo17-volmunes/emptydir.yaml 
```

Usualmente se usa como caché


[hostPath](https://kubernetes.io/docs/concepts/storage/volumes/#hostpath)
Un volumen dentro del Nodo el cual es compartido en los pod. Así se elimine el Pod el volumen no se ve afectado. Generalmente hostPath es usado para temas de desarrollo.
![hostPath](./KubernetesDePrincipianteAExperto/modulo17-volmunes/hostpath.png)

[PV - PVC]
PV -> Persitent Volume
PVC -> Persient Volume Claim

## Creando un PV
[Más Información](https://kubernetes.io/docs/tasks/configure-pod-container/configure-persistent-volume-storage/)

El pv es el encagargado de crear los discos.

```
kubectl apply -f KubernetesDePrincipianteAExperto/modulo17-volmunes/pv.yaml
```

PVC : Reclamar el espacio creado en PV

```
kubectl get pv --show-labels
```

## Montando un pvc en POD

[PodUsandoPVC](KubernetesDePrincipianteAExperto/modulo17-volmunes/pod-pvc.yaml)

Para editar alguna propiedad de un PVC podemos usar el comando kubectl edit

```
kubectl edit pvc mypvc
```

# Módulo 19: RBAC

Se trabajará con la creación de roles y bindings usando certificados. 

Ver el certifado de RBAC con el que estamos trabajando

``` 
kubectl config view 
```

1. Crear una llave privada
```
openssl genrsa -out jcisneros.key 2048
```

2. Generar un CSR (Certificate Signing Request)
```
openssl req -new -key jcisneros.key -out jcisneros.csr -subj "/CN=javier/O=dev"
```

3. Enviar y Solicitar la firma del CSR
```
openssl x509 -req -in jcisneros.csr -CA /home/javing77/.minikube/ca.crt -CAkey /home/javing77/.minikube/ca.key -CAcreateserial -out jcisneros.crt -days 365
```

4. Validar la firma del CSR
```
openssl verify -CAfile /home/javing77/.minikube/ca.crt jcisneros.crt
```

5. Ver detalles del crt
```
openssl x509 -in jcisneros.crt -text -noout
```

6. El crt se debe enviar al usuario y este lo utilizará para conectar

7. Ver el usuario actual de kubernetes
```
kubectl config current-context
``` 

8. Obctener los contexto 
```
kubectl config get-contexts
```


## Esto debería ir un script
10. crear y conectar a un cluster de minikube usando el certificado de RBAC

10.1. Obtener informaciön del cluster
```
kubectl cluster-info
```

10.2. Crear un docker container. Validar que queden en la misma red. Para esto puede usar el comando docker network ls
  
  ```
docker run -ti -v $PWD:/test -w /test -v /home/javing77/.minikube/ca.crt:/test/ca.crt -v /usr/local/bin/kubectl:/usr/bin/kubectl --network=minikube  alpine sh  
```

10.3. Crear un cluster y conectarlo al cluster de minikube.
  
  ```
  kubectl config set-cluster minikube --server=https://192.168.49.2:8443 --certificate-authority=/test/ca.crt
  kubectl config view
  ```
10.4 Agregar las credenciales.
  
  ```
  kubectl config set-credentials javier --client-certificate=/test/jcisneros.crt --client-key=/test/jcisneros.key
  kubectl config view
  ```

10.5 Configurar el contexto.
    
    ```
    kubectl config set-context minikube --cluster=minikube --user=javier
    kubectl config use-context minikube
    ```

Resumén Script:
```
kubectl config set-cluster minikube --server=https://192.168.49.2:8443 --certificate-authority=ca.crt
kubectl config set-credentials javier --client-certificate=jcisneros.crt --client-key=jcisneros.key
kubectl config set-context javier --cluster=minikube --user=javier
kubectl config use-context javier
kubectl config get-contexts
 ```

 **Recodar - Cambio de Contextos:** kubectl config use-context minikube
 **Recodar - Eliminar un Contexto:** kubectl config unset contexts.minikubeDev


11. Validar que minikube ten el RBAC habilitado

```
kubectl cluster-info dump | grep authorization-mode
```
## Creaundo un ROL y un BINDING

Verbos 
![Verbos](./KubernetesDePrincipianteAExperto/modulo19-rbac/verbs.png)


[rol.yaml](KubernetesDePrincipianteAExperto/modulo19-rbac/rol.yaml)

```
kubectl apply -f rol.yaml
kubectl get roles
``` 

ROLE BINGING:
[role-binding.yaml](KubernetesDePrincipianteAExperto/modulo19-rbac/role-binding.yaml)
```
kubectl apply -f KubernetesDePrincipianteAExperto/modulo19-rbac/role-binding.yaml
kubectl get rolebinding
```

Al hacerle un describe al rolebinding podemos ver que el rolebinding tiene una propiedad que es: Subject y esta nos dice que usuario tiene asignado el rolebinding.

```
kubectl describe rolebinding read-pods
```

Con esto cuando se use el contexto de javier ya podrá hacer consultas sobres los pods en default.

## Volviendo un usuario administrador

Ver los clusterole que existe.

```
kubectl get clusterroles
```

# Módulo 20: ServiceAccounts

Siempre se crearáun un ServiceAccount por defecto para cada Namespace que se crea. El cual está asociado a un token.

Mostrar nuestros servicesaccount
```
kubectl get sa
kubectl describe sa default
```

Para ver los secrets que tiene un serviceaccount
```
kubectl get secret
```

## Creando un serviceaccount

[sa.yaml](KubernetesDePrincipianteAExperto/modulo20-serviceaccounts/sa.yaml)

```
kubectl apply -f sa.yaml
```

Para el siguiente ejercicio se crearan dos pods. se accederá a uno de los pods, se instalará el paquete curl y se le hará una petición a la API de kubernetes. para saber cual es la ip hacer un **kubectl get svc**

[nginx01.yaml](KubernetesDePrincipianteAExperto/modulo20-serviceaccounts/nginx01.yaml)

```
kubectl apply -f nginx01.yaml
```

Ahora ingresar al pod y hacer una petición a la API de kubernetes.

```
kubectl exec -it nginxsa1 -- sh
apk add curl
curl https://10.96.0.1/api/v1/namespaces/default/pods --insecure
```

**Nota:** Cuando un pod es creado en este se crea un token que se le asigna al pod. 
```
cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

La respuesta por ahora es permiso deneegado.

## Enviando el token en la petición

1. Almacenar el resultado del token en una variable
```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
```
2. Hacer la petición
```
curl -H "Authorization: Bearer ${TOKEN}" https://10.96.0.1/api/v1 --insecure
```
Este token tiene restricciones, aunque nos dió una respuesta

## Crear un deployment y que este use nuestro ServiceAccount

[deplosa.yaml](KubernetesDePrincipianteAExperto/modulo20-serviceaccounts/deplosa.yaml)

```
kubectl apply -f deplosa.yaml
```

Entrar en el pod y hacer una petición a la API de kubernetes.

```
TOKEN=$(cat /var/run/secrets/kubernetes.io/serviceaccount/token)
curl -H "Authorization: Bearer ${TOKEN}" https://10.96.0.1/api/v1 --insecure
```

# Módulo 21: Ingress

