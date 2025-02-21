## Backup - Resource Configs

Este comando hace un backup de todos los recursos del cluster:

```sh
kubectl get all --all-namespaces -o yaml > all-deploy-services.yaml (only for few resource groups)
```

## Backend & Restore - ETCD

- Si el api-server es un service del sistema, primero haz el stop. Se puede revisar con  `systemctl status kube-apiserver`


```sh
service kube-apiserver stop
```

- La dirección local del etcd for defecto es: 127.0.0.1:2379

### ETCD como pod

Para saber la información de etcd describe el pod:

```sh
kubectl describe pod -n kube-system etcd-controlplane
```

Esta información se usa en los comandos de `etcdtl`

1. Haz el snapshot

    ```sh
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
      --cacert=<trusted-ca-file> --cert=<cert-file> --key=<key-file> \
      snapshot save <backup-file-location>
    ```

2. Para hacer el restore usa este comando

    ```sh
    ETCDCTL_API=3 etcdctl --data-dir <data-dir-location> \
      snapshot restore <backup-file-location>
    ```

    Si no funciona el comando así, intenta pasandole las demás configuraciones.

3. Actualiza el etcd manifest: /etc/kubernetes/manifests/etcd.yaml

    ```yaml
    volumes:
    - hostPath:
      path: /var/lib/etcd-from-backup
      type: DirectoryOrCreate
    name: etcd-data
    ```

    Cuando se actualiza el manifest, el pod de ETCD automaticamente se va a recrear porque es un static pod.

    **Nota:** Aquí se actualiza el volumen y no el `--data-dir` del command porque en el volumen es donde se hace referencia al restore del snashot en el host. Y el `--data-dir` es solo para dentro del container.

### ETCD como external service

1. Verifica si es un pod o un service

    Para revisar si es un pod: 

    ```sh
    kubectl get pods -n kube-system
    ```

    Para revisar si un service:

    ```sh
    systemctl status etcd
    ```
    Si no lo encuentras, verifica el api-server en el parametro de etcd, ahí podrías veritificar si es un external server.

2. Haz el snapshot:

    **Nota:** Si el server de etcd se encuentra en el cluster, puedes hacer el snapshot desde ahí, si no, haz el ssh al server del etcd para que lo saques

    - Y mueve el backup al nodo que se requiera, ya sea:
        - Desde el nodo1 copia el backup de etc-server: `scp etc-server:path/to/snashot /path/to/snashot`
        - Desde el etcd-server copia el backup al nodo 1: `scp /path/to/snashot nodo1:/path/to/snashot`

3. Haz el restore

    ```sh
    ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379 \
      --cacert=/etc/etcd/pki/ca.pem --cert=/etc/etcd/pki/etcd.pem \
      --key=/etc/etcd/pki/etcd-key.pem snapshot restore /root/cluster2.db \
      --data-dir /var/lib/etcd-data-new
    ```

4. Actualiza el service

    Si no sabes el path del service sacalo con `sytemctl status etcd`

    ```sh
    vi /etc/systemd/system/etcd.service and add the new value for data-dir:
    ```

5. Actualiza los permisos del folder de data

    ```
    chown -R etcd:etcd /var/lib/etcd-data-new

    ls -ld /var/lib/etcd-data-new/
    ```

6. Reinicia los servicios

    ```sh
    systemctl daemon-reload 
    etcd-server ~ ➜  systemctl restart etcd
    ```

    Y si el api-server es un service tambien reinicialo

    ```sh
    service kube-apiserver start
    ```

    **Nota:** Esto no viene en la documentación oficial
