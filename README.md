
# Kubernetes

## KUBERNETES IDE

LENS // THE KUBERNETES IDE

https://k8slens.dev/

## Kubernetes com Minikube

1º. Baixar os instaladores kubectl e minikube

2º. Criar um diretório chamado apps no diretório home do usuário e dentro deste criar um díretório <br>
chamado kube e então colocar o executável do kubectl aqui dentro.

3º. Adicionar o caminho para o executável do kubectl no path do sistema.

**`kubectl version --client`** <br>
4º. Testar kubectl funcionando

5º. Executar o instalador do Minikube 

**`minikube config set driver docker`** <br>
6º. Configurar o docker como driver padrão (container runtime) para o Kubernetes

**`minikube start`** <br>
7º. Inicializa/Reativa um cluster

**`minikube delete`** <br>
Deleta um cluster existente, caso necessário

Alterando a configuração para executá-lo no virtualbox. <br> 
Então você pode alcançar seu pod sql sem tunelamento.

**`minikube config set vm-driver virtualbox`** <br>
Change the minikube driver to virtualbox

*Comandos Minikube Úteis*

**`minikube status`** <br>
Checa o status do kubernetes

**`minikube ip`** <br>
Busca o IP do cluster

**`minikube service frontend-svc --url`** <br>
Busca o IP de um service específico

## Kubernetes com Kind

**`kind create cluster --name clusterfull`** <br>
Cria um cluster com apenas um nó

**`kind delete cluster --name clusterfull`** <br>
Cria um cluster com apenas um nó

Arquivo yaml de manifesto para criar cluster com vários nodes
```
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4

nodes:
- role: control-plane
- role: worker
- role: worker
- role: worker
- role: worker
```

**`kind create cluster --name clusterfull --config=kind.yaml`** <br>
Cria um cluster baseado no arquivo de manifesto


## Kubernetes com AWS EKS

[1 - EKS Started]https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html 

Cliente AWS <br>
[2 - AWS Cli]https://docs.aws.amazon.com/eks/latest/userguide/getting-started-console.html#gs-console-install-awscli

AWS-IAM-Authenticator <br>
[3 - IAM Authenticator]https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html

*---*

1º. Criar Função <br>
Entrar no IAM > Criar Função > Selecionar EKS > EKS Cluster <br>
Em Permissões "AmazonEKSClusterPolicy" cliar em próximo <br>
Em Revisar inserir o nome da função de "eks-cluster-role", cliar em "Criar função" <br>

2º. Criar Função <br>
Entrar no IAM > Criar Função > Selecionar EC2 <br>
Em Permissões "AmazonEKS_CNI_Policy", "AmazonEKSWorkerNodePolicy" e "AmazonEC2ContainerRegistryReadOnly" cliar em próximo <br>
Em Revisar inserir o nome da função de "eks-worker-node-role", cliar em "Criar função" <br>

3º. Acessar CloudFormation <br>
Criar um EKS VPC baseado em um template (1º link) <br>
```
aws cloudformation create-stack --region us-east-1 --stack-name my-eks-vpc-stack --template-url https://amazon-eks.s3.us-west-2.amazonaws.com/cloudformation/2020-10-29/amazon-eks-vpc-private-subnets.yaml
```

4º. Criar Cluster EKS <br>
Clicar em "Criar cluster", escolher nome "eks-aplicativos" <br>
Em "Função de serviço do cluster" selecionar "eks-cluster-role" <br>
Em VPC selecionar "my-eks-vpc-stack" <br>
Em Grupo de segurança selecionar "my-eks-vpc-stack" <br>
Em Cluster endpoint selecionar "Plublic e private" <br>
Clicar em "Criar cluster" <br>

5º. Add Cluster EKS Kubectl <br>
**`aws eks --region us-east-1 update-kubeconfig --name <nome_cluster>`** <br>
Relaciona o cluster ao kubectl <br>

6º. Criando Variável de Ambiente <br>
Adicione o caminho desse arquivo à variável de ambiente KUBECONFIG para que o kubectl saiba onde procurar a configuração do cluster. <br>
Copiar endereço que está no bash "C:\Users\jpral\.kube\config" <br>
Para shells Bash no macOS ou Linux: <br>
export KUBECONFIG=$KUBECONFIG:~/.kube/config-<devel> <br>
Para PowerShell no Windows: <br>
$ENV:KUBECONFIG="{0};{1}" -f  $ENV:KUBECONFIG, "$ENV:C:\Users\jpral\.kube\config" <br>

7º. Criar Node dentro do Cluster <br>
Clicar em "Add Node Group" <br>
Inserir o nome de "eks-group-node" <br>
Em "Função do IAM do nó" selecionar "eks-worker-node-role" <br>
Em "Tipos de instância" selecionar "t3.micro" <br>
Habilitar "Configurar acesso SSH aos nós" <br>
Selecionar "Par de chaves SSH" <br>
Selecionar em "Grupos de segurança" o grupo "my-eks-vpc-stack" <br>

8º. Criar os Pods <br>
Com o kubctl criar os deployments e services <br>
Para os services do tipo "LoadBalancer", será apresentado um External-IP <br>
Exemplo: `http://a317cdce4596542c5a9e434b6a6c0f39-11935794.us-east-1.elb.amazonaws.com` <br>


## Comandos Básicos 

*Multiple Clusters kubectl*

**`kubectl config get-contexts`** <br>
Obtem todos os contextos

**`kubectl config use-context cluster-2`** <br>
Switched to context "cluster-2".

Delete Cluster/Context/Users <br>
**`kubectl config unset users.gke_project_zone_name`** <br>
**`kubectl config unset contexts.aws_cluster1-kubernetes`** <br>
**`kubectl config unset clusters.foobar-baz`** <br>

*Comandos Úteis*

**`kubectl cluster-info`** <br>
Obtem informações do cluster

**`kubectl get nodes`** <br>
Obtem todos os nodes ativos

**`kubectl run nginx --image nginx`** <br>
Cria um pod chamado nginx utilizando a imagem nginx

**`kubectl run -it exjtecnologia-k8s --image=jpralves/exjtecnologia-k8s:latest --port=8081`** <br>

**`kubectl get pods`** <br>
Obtem uma lista de pods dentro do cluster

**`kubectl get pods -o wide`** <br>
Obtem uma lista de pods dentro do cluster com detalhes

**`kubectl describe pod nginx`** <br>
Descreve detalhadamente um determinado pod

*Delete All in Namespace*

**`kubectl delete all --all`** <br>
To delete everything from the default namespace

**`kubectl delete all --all -n {namespace}`** <br>
To delete everything from a certain namespace

**`kubectl delete namespace {namespace}`** <br>
**`kubectl create namespace {namespace}`** <br>
delete a namespace and re-create it


## Kubernetes com YAML

*Tipos de Definição*

Kind | Version
-----|---------
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

**`kubectl create -f arquivo.yaml`** <br>
Criar o objeto definido dentro do cluster

**`kubectl apply -f arquivo.yaml`** <br>
Criar o objeto definido dentro do cluster

**`kubectl apply -f <folder>/ --record`** <br>
TOdos os objetos no diretorio corrente são executados

**`kubectl replace -f arquivo.yaml`** <br>
Atualizar as definições do objeto

**`kubectl delete pod nginx-2`** <br>
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

**`kubectl get replicaset`** <br>
Obter replicasets

**`kubectl delete replicaset aplicacao-rs`** <br>
Obter replicasets

**`kubectl scale --replicas=6 -f arquivo.yaml`** <br>
Scalar a aplicação

**`kubectl scale --replicas=8 replicaset aplicacao-rs`** <br>
Scalar a aplicação

**`kubectl describe replicaset aplicacao-rs`** <br>
Descrição detalhada de um replicaset

Podemos criar um arquivo de deployment para gerenciar os deployments dos pods. <br>
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

**`kubectl get all`** <br>
Obtem todos os deployments, replicasets e pods

**`kubectl get all -n <namespace>`** <br>
Obtem todos os deployments, replicasets e pods de um namespace específico

**`kubectl get deployment`** <br>
Obtem todos os deployments

**`kubectl delete deployment aplicacao-rs`** <br>
Deleta o deployment

**`kubectl describe deployment aplicacao-rs`** <br>
Obtem todos os deployments, replicasets e pods

**`kubectl apply -f deploy.yaml --record --save-config`** <br>
Registra (--record) o comando no status do rollout

**`kubectl rollout status deployment/aplicacao-rs`** <br>
Criar o status do rollout

**`kubectl rollout history deployment/aplicacao-rs`** <br>
Criar o status do rollout

**`kubectl rollout undo deployment/aplicacao-rs`** <br>
Desfazer a última atualização feita pelo rollout

**`kubectl rollout undo deployment/aplicacao-rs --to-revision=1`** <br>
Desfazer a última atualização feita pelo rollout

## Kubernetes Redes

**`kubectl describe service kubernetes`** <br>
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

**`kubectl describe pod nginx-pd`** <br>
Detalha o pod

**`kubectl cluster-info`** <br>
Informações sobre o cluster e DNS

*Namespace no Kubernetes*

Todos os objetos por default recebem um namespace

**`kubectl get namespace`** <br>
Busca todos os namespaces

**`kubectl get pods -n default`** <br>
Busca todos os pods pertencentes ao namespace default

**`kubectl crate namespace <nome_namespace> --save-config`** <br>
Cria um novo namespace

**`--namespace=<nome_namespace>`** <br>
Utilizar ao final da maioria dos comandos para especificar o namespace

**`kubectl config set-context --current --namespace=frontend`** <br>
Especifica como default outro namespace

*Acessando um Pod*

**`kubectl exec -it <nome_pod> -- bash`** <br>
Acessa o bash do pod

## Kubernetes Services

Existem três tipos de serviço: NodePort, ClusterIP e LoadBalancer <br>
NodePort: habilita o acesso externo ao cluster <br>
ClusterIP: Usado para comunicação interna dentro do cluster <br>
LoadBalancer: É necessário um serviço de balanceamento de carga externo,
ele também gera um ip-externo quando se está em um cloud provider (AWS, Azure, Google)

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

**`kubectl get services`** <br>
Busca todos os namespaces

**`kubectl port-forward <service> <port>:<port>`** <br>
Abre um canal de acesso externo ao serviço

## Kubernetes Docker Registry

To pull a private DockerHub hosted image from a Kubernetes YAML:

```
DOCKER_REGISTRY_SERVER=docker.io
DOCKER_USER=jpralves
DOCKER_EMAIL=jpralves10@gmail.com
DOCKER_PASSWORD=***

kubectl create secret docker-registry myregistrykey \
  --docker-server=$DOCKER_REGISTRY_SERVER \
  --docker-username=$DOCKER_USER \
  --docker-password=$DOCKER_PASSWORD \
  --docker-email=$DOCKER_EMAIL
```

Referênciando o docker-registry:

```
apiVersion: v1
kind: Pod
metadata:
  name: whatever
spec:
  containers:
    - name: whatever
      image: DOCKER_USER/PRIVATE_REPO_NAME:latest
      imagePullPolicy: Always
      command: [ "echo", "SUCCESS" ]
  imagePullSecrets:
    - name: myregistrykey
```

## Kubernetes Volumes

* Volume
* Persistent Volume
* Persistent Volume Claim
* Storage Class

https://kubernetes.io/pt-br/docs/concepts/storage/persistent-volumes/


*Statico* <br>
POD -> PersistentVolumeClaim -> PersistentVolume -> volume

*Dinâmico* <br>
POD -> PersistentVolumeClaim -> StorageClass -> PersistentVolume -> volume

*Volume*

`awsElasticBlockStore` Bastante utilizado quando se deseja utilizar um volume distribuido
`emptyDir` De caráter temporário, morre quando o pod morre
`gcePersistentDisk` Bastante utilizado quando se deseja utilizar um volume distribuido
`hostPath` Cria um diretório no nó do cluster, não será acessível se o pod for criado em outro nó
`nfs` Bastante utilizado quando se deseja utilizar um volume distribuido

*Persistent Volume*

É preciso existir um PersistentVolume para representar um volume dentro do cluster 

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv0003
spec:
  capacity:
    storage: 5Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Recycle
  storageClassName: slow
  mountOptions:
    - hard
	- nfsvers=4.1
  nfs:
    path: /temp
	server: 172.17.0.2
```

*Persistent Volume Claim*

É a requisiçao de uso de um PersistentVolume, o PersistentVolume é consultado através do label

```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: myclaim
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  resources:
    requests:
      storage: 8Gi
  storageClassName: slow
  selector:
    matchLabels:
      release: "stable"
```

Os Pods podem ter acesso ao armazenamento utilizando a requisição como um volume. <br>
Para isso, a requisição tem que estar no mesmo namespace que o Pod. Ao localizar  <br>
a requisição no namespace do Pod, o cluster passa o PersistentVolume para a requisição.

```
apiVersion: v1
kind: Pod
metadata:
  name: mypod
spec:
  containers:
    - name: myfrontend
      image: nginx
      volumeMounts:
      - mountPath: "/var/www/html"
        name: mypd
  volumes:
    - name: mypd
      persistentVolumeClaim:
        claimName: myclaim
```

*Storage Class*

Simboliza a criação de volume dinamicamente
Serviço que aloca o volume dinamicamente

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: azurefile
provisioner: kubernetes.io/azure-file
parameters:
  skuName: Standard_LRS
  location: eastus
  storageAccount: azure_storage_account
```














