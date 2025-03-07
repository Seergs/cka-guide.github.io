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
