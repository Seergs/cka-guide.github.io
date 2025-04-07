Hay dos tipos:

- Ingress
- Egress

Para crear un network policy:

Se agrega el label del pod que quieres relacionar: 

```yaml
spec:
  podSelector:
    matchLabels:
```

Luego agregas el `from` cuando es ingress e igual pones el label del pod al que le vas a permitir que se comunique contigo/obtenga info de ti y agregar el puerto

```yaml
ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
```

**Ejemplo de ingress**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
 name: db-policy
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          role: api-pod
    ports:
    - protocol: TCP
      port: 3306
```

Puedes permitir por IP y bloquear las que no

```yaml
- ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
```

Igual puedes permitir solo de namespaces specificos

```yaml
 - namespaceSelector:
        matchLabels:
          project: myproject
```

**Ejemplo de ingress y egress**

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
  - Ingress
  - Egress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.17.0.0/16
        except:
        - 172.17.1.0/24
    - namespaceSelector:
        matchLabels:
          project: myproject
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 6379
  egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 5978
```

Y para egress igual, si te piden dos pods diferentes pones dos `to`

```yaml
egress:
  - to:
    - ipBlock:
        cidr: 10.0.0.0/24
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 5978

  - to:
    - podSelector:
        matchLabels:
          role: frontend
    ports:
    - protocol: TCP
      port: 5978
```
