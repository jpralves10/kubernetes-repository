
# Kubernetes

## Instalando Kubernetes

1º. Baixar os instaladores kubectl e minikube

2º. Criar um diretório chamado apps no diretório home do usuário e dentro deste criar um díretório
chamado kube e então colocar o executável do kubectl aqui dentro.

3º. Adicionar o caminho para o executável do kubectl no path do sistema.

**`kubectl version --client`**
4º. Testar kubectl funcionando

5º. Executar o instalador do Minikube 

**`minikube config set driver docker`**
6º. Configurar o docker como driver padrão (container runtime) para o Kubernetes

**`minikube start`** 
7º. Inicializa/Reativa um cluster

**`minikube delete`**
Deleta um cluster existente, caso necessário

## Comandos Básicos 

**`minikube status`**
Checa o status do kubernetes

**`kubectl cluster-info`**
Obtem informações do cluster

**`kubectl get nodes`**
Obtem todos os nodes ativos

**`kubectl run nginx --image nginx`**
Cria um pod chamado nginx utilizando a imagem nginx

**`kubectl get pods`**
Obtem uma lista de pods dentro do cluster

**`kubectl get pods -o wide`**
Obtem uma lista de pods dentro do cluster com detalhes

**`kubectl describe pod nginx`**
Descreve detalhadamente um determinado pod

## Kubernetes com YAML

*Tipos de Definição*

Kind | Version
--------------
Pod | v1
Service | v1
ReplicaSet | apps/v1
Deployment | apps/v1

Exemplo de arquivo de definição yaml para criação de Pod
```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-2
  labels:
    app: servidor-web
spec:
  containers:
    - name: nginx-container
    image: nginx
```

**`kubectl create -f arquivo.yaml`**
Criar o objeto definido dentro do cluster

**`kubectl apply -f arquivo.yaml`**
Criar o objeto definido dentro do cluster

**`kubectl replace -f arquivo.yaml`**
Atualizar as definições do objeto

**`kubectl delete pod nginx-2`**
Remover um pod

Exemplo de arquivo de definição yaml para criação de ReplicaSet
```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: aplicacao-rs
  labels:
    app: aplicacao
    type: frontend
spec:
  template:
    metadata:
      name: aplicacao-pod
      labels:
        app: aplicacao
        type: frontend
    spec:
	  containers:
        - name: nginx-container
          image: nginx
  replicas: 2
  selector:
    matchLabels:
      type: frontend
```

**`kubectl get replicaset`**
Obter replicasets

**`kubectl delete replicaset aplicacao-rs`**
Obter replicasets

**`kubectl scale --replicas=6 -f arquivo.yaml`**
Scalar a aplicação

**`kubectl scale --replicas=8 replicaset aplicacao-rs`**
Scalar a aplicação

**`kubectl describe replicaset aplicacao-rs`**
Descrição detalhada de um replicaset

Podemos criar um arquivo de deployment para gerenciar os deployments dos pods.
Quando se cria um Deployment, automaticamente se cria um replicaset.

Exemplo de arquivo de definição yaml para criação de Deployment
```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: aplicacao-rs
  labels:
    app: aplicacao
    type: frontend
spec:
  template:
    metadata:
      name: aplicacao-pod
      labels:
        app: aplicacao
        type: frontend
    spec:
	  containers:
        - name: nginx-container
          image: nginx
  replicas: 2
  selector:
    matchLabels:
      type: frontend
```

**`kubectl get all`**
Obtem todos os deployments, replicasets e pods

**`kubectl get deployment`**
Obtem todos os deployments

**`kubectl delete deployment aplicacao-rs`**
Deleta o deployment

**`kubectl describe deployment aplicacao-rs`**
Obtem todos os deployments, replicasets e pods

**`kubectl apply -f deploy.yaml --record --save-config`**
Registra (--record) o comando no status do rollout

**`kubectl rollout status deployment/aplicacao-rs`**
Criar o status do rollout

**`kubectl rollout history deployment/aplicacao-rs`**
Criar o status do rollout

**`kubectl rollout undo deployment/aplicacao-rs`**
Desfazer a última atualização feita pelo rollout

**`kubectl rollout undo deployment/aplicacao-rs --to-revision=1`**
Desfazer a última atualização feita pelo rollout

## Kubernetes Redes

**`kubectl describe service kubernetes`**
Detalha o serviço do kubernetes

```
λ kubectl describe service kubernetes 
Name:              kubernetes         
Namespace:         default            
Labels:            component=apiserver
                   provider=kubernetes
Annotations:       <none>             
Selector:          <none>             
Type:              ClusterIP          
IP Family Policy:  SingleStack        
IP Families:       IPv4               
IP:                10.96.0.1          
IPs:               10.96.0.1          
Port:              https  443/TCP     
TargetPort:        8443/TCP           
Endpoints:         192.168.49.2:8443  
Session Affinity:  None               
Events:            <none>     
```

O cluster nasce com dois IPs, sendo um do próprio cluster e o outro da apiServer (endpoint).
É o endereço da apiServer que usamos para acessar o cluster via SSH e etc.
Diferentemete do docker, no kubernetes que recebe endereço IP é o Pod.

**`kubectl describe pod nginx-pd`**
Detalha o pod

**`kubectl cluster-info`**
Informações sobre o cluster e DNS

*Namespace no Kubernetes*

Todos os objetos por default recebem um namespace

**`kubectl get namespace`**
Busca todos os namespaces

**`kubectl get pods -n default`**
Busca todos os pods pertencentes ao namespace default

**`kubectl crate namespace <nome_namespace> --save-config`**
Cria um novo namespace

**`--namespace=<nome_namespace>`**
Utilizar ao final da maioria dos comandos para especificar o namespace

**`kubectl config set-context --current --namespace=frontend`**
Especifica como default outro namespace

*Acessando um Pod*

**`kubectl exec -it <nome_pod> -- bash`**
Busca todos os namespaces

## Kubernetes Services

Existem três tipos de serviço: NodePort, ClusterIP e LoadBalancer
NodePort: habilita o acesso externo ao cluster
ClusterIP: Usado para comunicação interna dentro do cluster
LoadBalancer: É necessário um serviço de balanceamento de carga externo

As portas externas devem pertencer ao range 30000 - 32767

Service do tipo NodePort:
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    type: frontend
  ports:
    - name: http
      targetPort: 80
      port: 80
      nodePort: 30080
  type: NodePort
```

Service do tipo ClusterIP:
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    type: frontend
  ports:
    - name: http
      port: 80
  type: ClusterIP
```

Service do tipo LoadBalancer:
```
apiVersion: v1
kind: Service
metadata:
  name: frontend-svc
spec:
  selector:
    type: frontend
  ports:
    - name: http
      targetPort: 80
      port: 80
  type: LoadBalancer
```

**`kubectl get services`**
Busca todos os namespaces

**`minikube ip`**
Busca o IP do cluster

**`minikube service frontend-svc --url`**
Busca o IP de um service específico






