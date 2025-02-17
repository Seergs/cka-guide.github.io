## Definición en yaml

Se requieren forzosamente estos 4 valores:
```yaml
apiVersion:
kind:
metadata:

spec:
```

Si no hay un scheduler tu puedes asignarlo directamente en el yaml con:
nodeName:
o creando un archivo de tipo Binding y pasandolo por un http request

## Static Pods

Por si quieres tener un nodo funcionando "standalone" sin un master node
puedes configurar un directorio (/etc/kubernetes/manifests) donde colocas los yamls
de los pods y el kubelet se encaragará de aplicarlos y actualizalos.
**Solo funciona con Pods, no funciona con Deployments, ni replica sets, etc**

Si tienes un cluster, tambien podrías ver los static pods pero no se pueden modificar, solo ver.
Solo se pueden modificar/editar desde el nodo donde se creó
