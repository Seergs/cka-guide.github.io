
# KubeConfig
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: DATA+OMITTED
    server: https://controlplane:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: DATA+OMITTED
    client-key-data: DATA+OMITTED
```
**Nota**
certificate-authority-data: le pones el ca.crt 
Aqui tienes que hacerlo en base64 
```
cat ca.crt | base64 
```

certificate-authority: en donde esta ubicado el crt



El cliente usa el archivo de certificado y la clave para consultar la API Rest de Kubernetes y obtener una lista de pods que usan curl.

![api](../assets/kc1.PNG)

Podemos trasladar esta información a un archivo de configuración llamado kubeconfig y especificar este archivo como la opción kubeconfig en el comando.

```
$ kubectl get pods --kubeconfig config
```

Kubeconfig File
The kubeconfig tien 3 secciones

- Clusters
- Contexts
- USers

Cluster son los cluster que están disponibles, user es quien esta permitido y el context es la combinaciondel cluster con el user, de esa forma se evita en meter los certicados en cada petición que uno haga

Puede especificar el archivo kubeconfig con la vista de configuración de kubectl con el indicador "--kubeconfig"

```
$ kubectl config veiw --kubeconfig=my-custom-config
```


¿Cómo se actualiza el current-context? ¿O se cambia el current-context?

```
$ kubectl config use-context <context-name>
ex: 
$ kubectl config use-context prod-user@production
```


What about namespaces?
Especificar dentro del context el namespace que quieras
![namespaces](../assets/kc9.PNG)


```
apiVersion: v1
kind: Config

clusters:
- name: production
  cluster:
    certificate-authority: /etc/kubernetes/pki/ca.crt
    server: https://controlplane:6443

```

Para cambiar el kubeconfig sin necesidad de usar el indicador "--kubeconfig" 

1. /root/.kube/config o ~/.kube/config sobre escribir el kubeconfig
2. sin sobrescribir creas una variable en "/root/.bashrc
```
export KUBECONFIG=/kubeconfig
source ~/.bashrc
```
