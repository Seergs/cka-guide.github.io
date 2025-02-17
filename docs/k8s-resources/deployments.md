
Deployments crea automáticamente replicasets, que a su vez crea pods.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp-deployment
  labels:
    app: myapp
    type: front-end
spec:
 template:
    metadata:
      name: myapp-pod
      labels:
        app: myapp
        type: front-end
    spec:
     containers:
     - name: nginx-container
       image: nginx
 replicas: 3
 selector:
   matchLabels:
    type: front-end
```

Solo aplicamos y listo!

```sh
kubectl create -f deployment-definition.yaml
```

O de forma imperativa
```sh
kubectl create deployment webapp --image=kodekloud/webapp-color --replicas=3
```


- **Recreate**: Se bajan todos los pods y se suben todos
- **RollingUpdate**: Se baja uno y se se sube uno

![deployment commands](../assets/deployment-commands.png.jpg)
