# Authorization

Hay diferentes mecanismos de autorización compatibles con Kubernetes

- Node Authorization
- Attribute-based Authorization (ABAC)
- Role-Based Authorization (RBAC)
- Webhook


## Node Authorization

 Se configura como parte del proceso de configuración de kubelet para garantizar que el kubelet solo pueda hacer las cosas que están permitidas en el clúster.


## Attribute-based Authorization (ABAC)

Se configura mediante politicas, por ejemplo este define que el usuario alice, tiene acceso a pods en el namespace de frontend

```json
{
  "apiVersion": "abac.authorization.kubernetes.io/v1beta1",
  "kind": "Policy",
  "spec": {
    "user": "alice",
    "namespace": "frontend",
    "resource": "pods",
    "readonly": true
  }
}
```

## Role-Based Authorization (RBAC)

Este tipo funciona medante 4 componentes principales:

1. **Roles**: Definen los permisos al nivel del namespace, especificando que acciones se pueden realizar sobre recursos específico (pod, servicios)

    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: Role
    metadata:
      namespace: default
      name: pod-reader
    rules:
    - apiGroups: [""]
      resources: ["pods"]
      verbs: ["get", "list", "watch"]
    ```

2. **ClusterRole**: Similar a los roles pero con alcance al cluster completo, permitiendo gestionar recursos no namespaced como nodos.
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRole
    metadata:
      name: node-admin
    rules:
    - apiGroups: [""]
      resources: ["nodes"]
      verbs: ["get", "list", "watch", "update"]
    ```
    
3. **RoleBindings**: Conectan a los roles con los usuarios, grupos o service accounts dentro de un namespace
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: read-pods
      namespace: default
    subjects:
    - kind: User
      name: jane
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: Role
      name: pod-reader
      apiGroup: rbac.authorization.k8s.io
    ```

4. **ClusterRoleBiding**: Vinculan ClusterRoles con usuarios a nivel cluster
    ```yaml
    apiVersion: rbac.authorization.k8s.io/v1
    kind: ClusterRoleBinding
    metadata:
      name: node-admin-binding
    subjects:
    - kind: User
      name: admin
      apiGroup: rbac.authorization.k8s.io
    roleRef:
      kind: ClusterRole
      name: node-admin
      apiGroup: rbac.authorization.k8s.io
    ```

RBAC también se puede configurar imperativamente con los siguientes comandos:

```
# Create a role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
kubectl create role pod-reader --verb=get --verb=list --verb=watch --resource=pods

# Create a role named "pod-reader" with ResourceName specified
kubectl create role pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod

# Create a role named "foo" with API Group specified
kubectl create role foo --verb=get,list,watch --resource=rs.apps

# Create a cluster role named "pod-reader" that allows user to perform "get", "watch" and "list" on pods
kubectl create clusterrole pod-reader --verb=get,list,watch --resource=pods

# Create a cluster role named "pod-reader" with ResourceName specified
kubectl create clusterrole pod-reader --verb=get --resource=pods --resource-name=readablepod --resource-name=anotherpod

# Create a cluster role named "foo" with API Group specified
kubectl create clusterrole foo --verb=get,list,watch --resource=rs.apps
```

Rolebindings:

```
# Create a role binding for user1, user2, and group1 using the admin cluster role
kubectl create rolebinding admin --clusterrole=admin --user=user1 --user=user2 --group=group1

# Create a role binding for service account monitoring:sa-dev using the admin role
kubectl create rolebinding admin-binding --role=admin --serviceaccount=monitoring:sa-dev

# Create a cluster role binding for user1, user2, and group1 using the cluster-admin cluster role
kubectl create clusterrolebinding cluster-admin --clusterrole=cluster-admin --user=user1 --user=user2 --group=group1
```

**Nota**: Se puede utilizar un ClusterRole y ligarlo a un RoleBinding, este patrón tiene varias ventajas:

1.	Reutilización de roles: Puedes definir un conjunto de permisos una vez (en el ClusterRole) y reutilizarlo en múltiples namespaces
2.	Consistencia: Mantienes una definición única de permisos
3.	Mantenibilidad: Si necesitas modificar los permisos, solo cambias el ClusterRole y se actualiza en todos los namespaces donde se usa


## Webhook

Este modo funciona mediante un servicio HTTP externo, el cual deficide si las llamadas están autorizadas o no.

Tu configuras un servicio y pones la url del server, este servicio debe responder con algo como

```json
{
  "apiVersion": "authorization.k8s.io/v1",
  "kind": "SubjectAccessReview",
  "status": {
    "allowed": true,
    "reason": "user is authorized"
  }
}
```
