# kubernetes-tutorial
Tutorial criado utilizando o material disponibilizado no [site do Kubernetes](https://kubernetes.io/docs/tutorials/kubernetes-basics/)

Site de apoio [katacode](https://www.katacoda.com/courses/kubernetes)

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

Quando uma aplicação é criada, o Kubernetes Deployment Controller monitora continuamente essa instância. Se o node hospedado cair ou for deletado, o Deployment Controller recoloca a instância com outro node no cluster. **Este é um mecanismo de auto-recuperação para falhas de endereçamento ou manutenção**.

### Deploy do app no Kubernetes

![](https://d33wubrfki0l68.cloudfront.net/152c845f25df8e69dd24dd7b0836a289747e258a/4a1d2/docs/tutorials/kubernetes-basics/public/images/module_02_first_app.svg)

### Launch Cluster
```bash
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

# 4 Expondo o app

### Services

Um node é mortal. Ele tem ciclo de vida. Quando um node morre, os pods que rodam no nó ficam perdidos. O ReplicaSet deve então dinamicamente direcionar de volta ao estado desejado criando um novo pod para manter a aplicação rodando.
Um `service` é uma abstração que define um conjunto lógico de Pods e uma política para poder acessá-los. 
Como os pods possuem um único IP, esses IPs não ficam expostos ao cluster sem um service.
Services podem ser expostos em diferentes formas especificando um `type` no ServiceSpec.

- ClusterIP    | Expõe o Service em um IP interno no cluster. Service fica disponível apenas dentro do cluster.
- NodePort     | Expõe o Service na mesma porta do Node selecionado no cluster utilizando NAT. Possível acesso externo utilizando `<NodeIP>:<NodePort>`
- LoadBalancer | Cria um loadBalancer na núvem atual e define um IP externo fixo ao service.
- ExternalName | Expõe o Service utilizando um nome arbitrário (especificado pelo `externalName`) retornando um CNAME. Nenhum proxy é utilizado. Requer a v1.7 ou maior do `kube-dns`.

![](https://d33wubrfki0l68.cloudfront.net/cc38b0f3c0fd94e66495e3a4198f2096cdecd3d5/ace10/docs/tutorials/kubernetes-basics/public/images/module_04_services.svg)

É possível alcançar um grupo de pods utilizando labels e selectors.
- Designar objetos para desenvolvimento, teste e produção.
- Injetar tags de versão
- Classificar um objeto utilizando tags.

![](https://d33wubrfki0l68.cloudfront.net/b964c59cdc1979dd4e1904c25f43745564ef6bee/f3351/docs/tutorials/kubernetes-basics/public/images/module_04_labels.svg)

### Criando um novo service

```bash
kubectl get pods
kubectl get services
kubectl expose deployment/kubernetes-bootcamp --type="NodePort" -- 8080
kubectl get services

kubectl describe services/kubernetes-bootcamp

export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT

curl $(minikube ip):$NODE_PORT
```

### Utilizando labels

```bash
kubectl describe deployment
kubectl get pods -l run=kubernetes-bootcamp
kubectl get services -l run=kubernetes-bootcamp

export POD_NAME=$(kubectl get pods -o go-template --template '{{range .items}}{{.metadata.name}}{{"\n"}}{{end}}')
echo Name of the Pod: $POD_NAME

kubectl label pod $POD_NAME app=v1
kubectl describe pods $POD_NAME
kubectl get pods -l app=v1
```

### Deletando um Service

```bash
kubectl delete service -l run=kubernetes-bootcamp
kubectl get services
curl $(minikube ip):$NODE_PORT
kubectl exec -ti $POD_NAME curl localhost:8080
```

# 5 Escalando o app

Nos processos anteriores, nós subimos apenas um pod rodando a aplicação. Quando o tráfego aumenta, precisamos escalar nossa aplicação para suprir a demanda.

![](https://d33wubrfki0l68.cloudfront.net/043eb67914e9474e30a303553d5a4c6c7301f378/0d8f6/docs/tutorials/kubernetes-basics/public/images/module_05_scaling1.svg)

![](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/b5f56/docs/tutorials/kubernetes-basics/public/images/module_05_scaling2.svg)

> O escalonamento é realizado alterando o número de réplicas em uma implantação.

### Escalando o Deployment

```bash
kubectl get deployments
kubectl scale deployments/kubernetes-bootcamp --replicas=4
kubectl get deployments
kubectl get pods -o wide
kubectl describe deployments/kubernetes-bootcamp
```

### Balanceamento de carga (Load Balancing)

```bash
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT

curl $(minikube ip):$NODE_PORT
```

### Desescalando (Scale Down)

```bash
kubectl scale deployments/kubernetes-bootcamp --replicas=2
kubectl get deployments
kubectl get pods -o wide
```

# 6 Atualizando o app

Usuários esperam que aplicações estejam disponíveis o tempo todo, e desenvolvedores esperam poder criar deploys várias vezes ao dia. No Kubernetes isto é feito através do rolling updates.
*Rolling Updates* permitem atualização de Deployments com zero downtime ao incrementar atualizações aos Pods.

![](https://d33wubrfki0l68.cloudfront.net/30f75140a581110443397192d70a4cdb37df7bfc/fa906/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates1.svg)

![](https://d33wubrfki0l68.cloudfront.net/678bcc3281bfcc588e87c73ffdc73c7a8380aca9/703a2/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates2.svg)

![](https://d33wubrfki0l68.cloudfront.net/9b57c000ea41aca21842da9e1d596cf22f1b9561/91786/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates3.svg)

![](https://d33wubrfki0l68.cloudfront.net/6d8bc1ebb4dc67051242bc828d3ae849dbeedb93/fbfa8/docs/tutorials/kubernetes-basics/public/images/module_06_rollingupdates4.svg)

> Se um Deployment estiver exposto publicamente, o Service vai balancear o tráfego apenas para Pods disponíveis durante o update.

Rolling updates permitem as seguintes funções:
- Promover uma aplicação de um ambiente para outro (via atualização de imagem de container)
- Rollback para versões anteriores
- Integração contínua e Entrega contínua com zer downtime

### Atualizando a versão do app

```bash
kubectl get deployments
kubectl get pods
kubectl describe pods
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2
kubectl get pods
```
### Verificar um update

```bash
kubectl describe services/kubernetes-bootcamp
export NODE_PORT=$(kubectl get services/kubernetes-bootcamp -o go-template='{{(index .spec.ports 0).nodePort}}')
echo NODE_PORT=$NODE_PORT

curl $(minikube ip):$NODE_PORT

kubectl rollout status deployments/kubernetes-bootcamp
kubectl describe pods
```

### Reverter um update (Rollback)

```bash
kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10
kubectl get deployments
kubectl get pods
kubectl describe pods

kubectl rollout undo deployments/kubernetes-bootcamp

kubectl get pods
kubectl describe pods
```
# Utilizando configurações via Yaml

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
