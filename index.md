# Certified Kubernetes Administrator

## Cluster Architecture

![cluster achitecture](./assets/cluster-architecture.png)

- Un container engine (agente) debe estar instalado en todos los nodos (por ejemplo Docker, containerd o Rocket)

### Worker Nodes

Trackea y monitorea los containers. Sirven para distribuir la carga.

### Kubelet

Corre en cada nodo del cluster y espera instrucciones del kubeapi server y crea o destruye containers.
Primero el kubeapi server le pregunta al scheduler en que nodo poner cierto pod, y después el kubeapi server le dice al kubelet para crearlo.
Este lo crea, descargando la imagen y creando el container. No viene instalado por defecto con kubeadmn, tienes que instalarlo a mano en cada nodo.

