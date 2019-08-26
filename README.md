# kubernetes-tutorial
Tutorial criado utilizando o material disponibilizado no site do Kubernetes

# 1 Criando um cluster Kubernetes 

Diagrama de Cluster
![](https://d33wubrfki0l68.cloudfront.net/99d9808dcbf2880a996ed50d308a186b5900cec9/40b94/docs/tutorials/kubernetes-basics/public/images/module_01_cluster.svg)

- O Master gerencia o cluster e os nodes são usados para hospedar as aplicações em execução.
- O node é uma VM ou computador físico que serve a uma worker machine em um Cluster Kubernetes.
- O node se comunica com o master utilizando o Kubernetes API.

### Utilizando o Minikube
```bash
minikube version
minikube start
```

após iniciar:
```bash
kubectl version
kubectl cluster-info
kubectl get nodes
```
