## Multi Containers

Se usa cuando quieres correr mas de un container en el mismo Pod, no es tan común.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: simple-webapp
  labels:
    name: simple-webapp
spec:
  containers:
  - name: simple-webapp
    image: simple-webapp
    ports:
    - containerPort: 8080
  - name: log-agent
    image: log-agent
```

En un pod multi-contenedor, se espera que cada contenedor ejecute un proceso que permanezca activo durante todo el ciclo de vida del POD. Por ejemplo, en el pod multi-contenedor del que hablamos anteriormente que tiene una aplicación web y un agente de registro, se espera que ambos contenedores permanezcan activos en todo momento. Se espera que el proceso que se ejecuta en el contenedor del agente de registro permanezca activo mientras la aplicación web esté en funcionamiento. Si alguno de ellos falla, el POD se reinicia.
Sin embargo, en ocasiones es posible que desees ejecutar un proceso que se complete en un contenedor. Por ejemplo, un proceso que extrae código o un binario desde un repositorio que será utilizado por la aplicación web principal. Esa es una tarea que se ejecutará solo una vez cuando el pod se cree por primera vez. O un proceso que espera a que un servicio externo o una base de datos esté activa antes de que la aplicación real se inicie. Ahí es donde entran los initContainers.
Un initContainer se configura en un pod como todos los demás contenedores, excepto que se especifica dentro de una sección initContainers, de esta manera:

## Init Containers

Se pueden agregar varios, corre de manera secuencial y si falla se reinicia hasta que sea satisfactorio.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: myapp-pod
  labels:
    app: myapp
spec:
  containers:
  - name: myapp-container
    image: busybox:1.28
    command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
  - name: init-myservice
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup myservice; do echo waiting for myservice; sleep 2; done;']
  - name: init-mydb
    image: busybox:1.28
    command: ['sh', '-c', 'until nslookup mydb; do echo waiting for mydb; sleep 2; done;']
```
