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

# 2 Fazendo Deploy

Quando uma aplicação é criada, o Kubernetes Deployment Controller monitora continuamente essa instância. Se o node hospedado cair ou for deletado, o Deployment Controller recoloca a instância com outro node no cluster. **Esta é um mecanismo de auto-recuperação para falhas de endereçamento ou manutenção**.

### Deploy do app no Kubernetes

![](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

### Launch Cluster
```bash
minikube start --wait=false
minikube start --wait=false
```
### Kubectl run
```bash
kubectl run http --image=katacoda/docker-http-server:latest --replicas=1
kubectl get deployments
kubectl describe deployment http
```

### Kubectl expose
```bash
kubectl expose deployment http --external-ip="172.17.0.15" --port=8000 --target-port=80
curl http://172.17.0.15:8000
```

### Kubectl run and expose
```bash
kubectl run httpexposed --image=katacoda/docker-http-server:latest --replicas=1 --port=80 --hostport=8001
curl http://172.17.0.15:8001
kubectl get svc
docker ps | grep httpexposed
```

### Scale containers
```bash
kubectl scale --replicas=3 deployment http
kubectl get pods
kubectl describe svc http
curl http://172.17.0.15:8000
```

# 3 Explorando o App

Um *Pod* é uma abstração do Kubernetes que representa um grupo de um ou mais containers (como o Docker ou rkt), e alguns recursos compartilhados para esses containers.
- Armazenamento compartilhado.
- Rede, como um único endereço IP.
- Informações de como rodar cada container, qual porta usar, versão da imagem.
![](https://d33wubrfki0l68.cloudfront.net/fe03f68d8ede9815184852ca2a4fd30325e5d15a/98064/docs/tutorials/kubernetes-basics/public/images/module_03_pods.svg)

Um pod sempre roda dentro de um *Node*. Um node é uma máquina de trabalho no Kubernetes e pode ser uma máquina virtual ou física, dependendo do cluster. Cada node é gerenciado pelo Master. Um node pode ter múltiplos pods.
Cada node roda pelo menos:
- Kubelet, processo de comunicação entre o Master e o Node, gerencia os pods e os containers rodando na máquina.
- Um container em tempo de execução (como o Docker, rkt) responsável por extrair a imagem do container de um registro, descompactar o container e executar o aplicativo.
![](https://d33wubrfki0l68.cloudfront.net/5cb72d407cbe2755e581b6de757e0d81760d5b86/a9df9/docs/tutorials/kubernetes-basics/public/images/module_03_nodes.svg)

Algumas operações do Kubernetes:
- *kubectl get* lista os recursos
- *kubectl describle* mostra informações detalhadas de um recurso
- *kubectl logs* printa os logs de um container em um pod
- *kubectl exec* executa um comando em um container em um pod


### Criando o Deployment

Configuração com o arquivo yaml
![](deployment.yaml)

```bash
kubectl create -f deployment.yaml
kubectl get deployment
kubectl descrive deployment webapp1
```
### Criando o Service

Configuração com o arquivo yaml
![](service.yaml)

```bash
kubectl create -f service.yaml
kubectl get svc
kubectl descrive svc webapp1-svc
curl host01:30080
```

### Escalando o Deployment

atualizar o deployment.yaml para `replicas: 4`

```bash
kubectl apply -f deployment.yaml
kubectl get deployment
kubectl get pods
curl host01:30080
```
