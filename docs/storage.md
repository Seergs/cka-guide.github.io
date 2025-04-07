## Montar configmap como volumen

```yaml
  volumes:
    - name: config-vol
      configMap:
        name: log-config
        items:
          - key: log_level
            path: log_level.conf
```

## Montar un volume con mountpath

```yaml
kind: Pod
metadata:
  name: hostpath-example-linux
spec:
  os: { name: linux }
  nodeSelector:
    kubernetes.io/os: linux
  containers:
  - name: example-container
    image: registry.k8s.io/test-webserver
    volumeMounts:
    - mountPath: /foo
      name: example-volume
      readOnly: true
  volumes:
  - name: example-volume
    # mount /data/foo, but only if that directory already exists
    hostPath:
      path: /data/foo # directory location on host
      type: Directory # this field is optional
```

## Persitent Volume

La diferencia entre este y un volumen es que el volumen se va a perder al
morirse el pod, además que un persisten volume está ligado a un PVC

```yaml
kind: PersistentVolume
apiVersion: v1
metadata:
    name: pv-vol1
spec:
    accessModes: [ "ReadWriteOnce" ]
    capacity:
     storage: 1Gi
    hostPath:
     path: /tmp/data
```

El PV es un recurso más estático. Una vez creado, el PV es “concreto” y está 
directamente asociado con un volumen físico o lógico. 
Cualquier ajuste posterior (como cambiar el tipo de almacenamiento o las propiedades 
del volumen) requiere modificar o reemplazar el PV.

## PersistentVolumeClaim

Se pueden ligar por medio de label o si no, k8s busca un PV para
ligarse automaticamnete comparando sus recursos, y configuraciones

Por ejemplo aquí, se escogería un PV que tenga al menos 1Gi y access mode "ReadWriteOnce"

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  resources:
   requests:
     storage: 1Gi
```

Y así se usa este PVC en un pod

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: web
  volumes:
    - name: web
      persistentVolumeClaim:
        claimName: myclaim
```

## StorageClass

Este te creará automaticamnete el PV, pero tu si debes crear tu PVC y ponerlo al pod

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
   name: google-storage
provisioner: kubernetes.io/gce-pd
parameters:
    type: pd-standard [ pd-standard | pd-ssd ]
    replication-type: none [ none | regional-pd ]
```

```yaml
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: myclaim
spec:
  accessModes: [ "ReadWriteOnce" ]
  storageClassName: google-storage       
  resources:
   requests:
     storage: 500Mi
```

Al utilizar por ejemplo este provider the `gce-pd`, el cual es un Cloud Provider
como de Google Cloud Platform, aun así debemos crear a mano el disco en Cloud 
para poder usarlo en k8s

Tambien podemos definir un StorageClass que no crea el volumen para auto provisioning

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner # indicates that this StorageClass does not support automatic provisioning
volumeBindingMode: WaitForFirstConsumer
```
