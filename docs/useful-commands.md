**Correr pod sin usar yaml**

```sh
kubectl run nginx --image=nginx
```

**Crear un pod y agregarle un label**

```sh
kubectl run redis --image=redis:alpine -l tier=db
```

**Crear un service para exponer un pod**

```sh
kubectl expose pod redis --port=6379 --name redis-service
```

**Crear un deployment con 3 replicas**

```sh
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```

**Crear un pod y exponerlo en el puerto 8080 (solo exponer el pod, no crea un servicio)**

```sh
kubectl run custom-nginx --image=nginx --port=8080
```

**Crea un nuevo namespace**

```sh
kubectl create ns dev-ns
```

**Crear un deployment en un namespace con 2 replicas**

```sh
kubectl create deployment redis-deploy -n dev-ns --image redis --replicas 2
```

**Crea un pod y crea un service de tipo ClusterIP automaticamente**

```sh
kubectl run httpd --image httpd:alpine --expose --port 80
```

**Simula crear un deployment y te da el yaml**

```sh
kubectl create deployent --image=nginx nginx --replicas=4 --dry-run=client -o yaml > nginx-deployment.yaml
```

**Cambiarte de namespace**

```sh
kubectl config set-context --current --namespace=<namespace>
```

**Obtener el current namespace**

```sh
kubectl config view | grep namespace
```

**Actualizar un pod**

```sh
kubectl replace --force -f nginx.yaml
```

**cuenta cuantos pods, eliminando la columna**
```sh
kubectl get pod --selector env=prod --no-headers | wc -l
```

**Aplicar un label a un nodo**

```sh
kubectl label nodes node01 key=value
```

**To apply a label to a node**
```sh 
kubectl label nodes node-1 size=Large
```

**Actualizar la imagen de un pod**

```sh
kubectl set image pod/my-app my-container=nginx:latest
```

**Te da el yaml de pod y le pasa un comando**

```
kubectl run --restart=Never --image=busybox static-busybox --dry-run=client -o yaml --command -- sleep 1000 > /etc/kubernetes/manifests/static-busybox.yaml
```

**Obtener los eventos**

```sh
kubectl get events -o wide
```

**Escalar un deployment**

```sh
kubectl scale deployment mydeployment --replicas=3
```

**Ver los recursos de los pods**

```sh
kubectl top pod -a
```
