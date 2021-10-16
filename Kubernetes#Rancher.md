# Kubernetes e Rancher :: 

### 1. Instalando Ambiente

*Pre-requisitos*
- 4 máquinas virtuais com 2/4 cpus e 6/8 gb de ram
- 1 domínio
- S.O Ubuntu

Dominio: `exjtecnologia.com`
Sub-dominio: `rancher.exjtecnologia.com` apontar para `server`
Sub-dominio: `*.rancher.exjtecnologia.com` apontar para `K8S-1`, `K8S-2`, `K8S-3`

É preciso entrar em todas as máquinas e instalar o docker

ssh -i "ec2-instance-key.pem" ubuntu@54.227.101.172		- Server - Host A
ssh -i "ec2-instance-key.pem" ubuntu@54.234.253.212		- K8S-1  - Host B
ssh -i "ec2-instance-key.pem" ubuntu@52.90.108.183		- K8S-2  - Host C
ssh -i "ec2-instance-key.pem" ubuntu@54.234.253.212		- K8S-3  - Host D
```
sudo su
curl https://releases.rancher.com/install-docker/20.10.sh | sh
usermod -aG docker ubuntu
docker ps (para verificar a instalação do docker)
```

### 2. Construindo a Aplicação

Construir as imagens dos Containers que iremos usar, colocar elas para rodar
em conjunto com o docker-compose

Entrar no host A e instalar os pacontes abaixo, que incluem Git, Python, Pip
e o Docker-compose
```
$ sudo su
$ apt-get install git -y
$ curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
$ chmod +x /usr/local/bin/docker-compose
$ ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose
$ docker-compose --version
```
Com os pacotes instalados, agora iremos baixar o código fonte e começaremos a
fazer os builds e rodar os containers

Substituir <dockerhub-user> pelo user do DockerHub
```
$ cd /home/ubuntu
$ rm -rf devops/
$ git clone https://github.com/jpralves10/devops
$ cd devops/exercicios/app
```

*Container=REDIS*

Iremos fazer o build da imagem do Redis para a nossa aplicação.
```
$ cd redis
$ docker build -t jpralves/redis:devops .
$ docker run -d --name redis -p 6379:6379 jpralves/redis:devops
$ docker ps
$ docker logs redis
```

*Container=NODE*

Iremos fazer o build do container do NodeJs que contém a nossa aplicação
```
$ cd ../node
$ docker build -t jpralves/node:devops .
```
Agora iremos rodar a imagem do node, fazendo a ligação dela com o container do Redis.
```
$ docker run -d --name node -p 8080:8080 --link redis jpralves/node:devops
$ docker ps
$ docker logs node
```
Com isso temos nossa aplicação rodando, e conectada no Redis. A api para verificação
pode ser acessada em /redis

*Container=NGINX*

Iremos fazer o build do container do nginx, que será nosso balanceador de carga
```
$ cd ../nginx
$ docker build -t jpralves/nginx:devops .
```
Criando o container do nginx a partir da imagem e fazendo a ligação com o container do Node
```
$ docker run -d --name nginx -p 80:80 --link node jpralves/nginx:devops
$ docker ps
```
Podemos acessar então nossa aplicação nas portas 80 e 8080 no ip da nossa instância.

Iremos acessar a api em /redis para nos certificar que está tudo ok, e depois iremos
limpar todos os containers e volumes.
```
$ docker rm -f $(docker ps -a -q)
$ docker volume rm $(docker volume ls)
```

*Docker-Compose*

Nesse exercício que fizemos agora, colocamos os containers para rodar, e interligando eles,
foi possível observar como funciona nossa aplicação que tem um contador de acessos.
Para rodar nosso docker-compose, precisamos remover todos os containers que estão rodando e ir
na raiz do diretório para rodar.

É preciso editar o arquivo docker-compose.yml, onde estão os nomes das imagens e colocar o seu
nome de usuário.

Linha 8 = <dockerhub-user>/nginx:devops
Linha 18 = image: <dockerhub-user>/redis:devops
Linha 37 = image: <dockerhub-user>/node:devops

Após alterar e colocar o nome correto das imagens, rodar o comando de up -d para subir a stack toda.
```
$ cd ..
$ vi docker-compose.yml
$ docker-compose -f docker-compose.yml up -d
$ curl <ip>:80
	---------------------------------
	This page has been viewd 29 times
	---------------------------------
```
Se acessarmos o IP:80, iremos acessar a nossa aplicação. Olhar os logs pelo docker logs, e fazer o
carregamento do banco em /load

Para terminar nossa aplicação temos que rodar o comando do docker-compose abaixo:
```
$ docker-compose down
```

### 3. Rancher-Single Node

*Instalar Rancher - Sigle Node*

Neste exercício iremos instalar o Rancher 2.2.5 versão single node. Isso significa que o Rancher e todos
seus componentes estão em um container.

Entrar no host A, que será usado para hospedar o Rancher Server. Iremos verificar se não tem nenhum	container
rodando ou parado, e depois iremos instalar o Rancher.
```
$ docker ps -a
$ docker run -d --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib/ -p 80:80 -p 443:443 rancher/rancher:v2.4.3
$ sudo docker run -d --privileged --name rancher --restart=unless-stopped -v /opt/rancher:/var/lib -p 80:80 -p 443:443 rancher/rancher:latest
```
Com o Rancher já rodando, irei adicionar a entrada de cada DNS para o IP de cada máquina.
```
$ rancher.<dominio> = IP do host A
```
Usuário: admin
Senha: admin

### 4. Kubernetes

*Criar Cluster Kubernetes*

Nesse exercício iremos criar um cluster Kubernetes. Após criar o cluster, instalar o Kubctl no Host A 
e iremos usar para iteragir com o cluster.

Seguir as instruções na aula para fazer o deployment do cluster.
Após fazer a configuração, o Rancher irá exibir um comando de docker run, para adicionar os hosts.

Criação do Cluster:

*1. Em "Advanced Options" / setar "Nginx Ingress" como disable
*2. Clicar em Next
*3. etcd / controlplane / worker
*4. Selecionar um nome para o nó

Adicionar o host B, host C e D.

Pegar o seu comando no seu rancher.
```
sudo docker run -d --privileged --restart=unless-stopped --net=host -v /etc/kubernetes:/etc/kubernetes -v /var/run:/var/run  rancher/rancher-agent:v2.6.0 --server https://rancher.exjtecnologia.com --token l6rv4d9xrvxncsnlhqm685qrdm89dpp2ltzcjpvdvcmh55tqbq6zmw --ca-checksum a2ab52b231f7b07de19ceb30ea083b48b9810b5e6fa7484a669381a84715a746 --node-name k8s-1 --etcd --controlplane --worker 
```
Será um cluster com 3 nós.
Navegar pelo Rancher e ver os paineis e funcionalidades.


### 5. Kubectl

*Instalar kubectl no host A*

Agora iremos instalar o kubectl, que é a CLI do kubernetes. Através do kubectl é que iremos iteragir com o cluster.
```
$ sudo apt-get update && sudo apt-get install -y apt-transport-https gnupg2
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
$ echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
$ sudo apt-get update
$ sudo apt-get install -y kubectl
$ kubectl version
```

Com o kubectl instalado, pegar as credencias de acesso no Rancher e configurar o kubectl.
Acessar no Rancher a opção "Download KubeConfig" e copiar dentro do arquivo "config"
```
$ mkdir -p ~/.kube
$ vi ~/.kube/config
$ kubectl get nodes
$ kubectl get pods -n kube-system
```

### 5. Traefik - DNS

O Traefik é a aplicação que iremos usar como ingress. Ele irá ficar escutando pelas entradas de DNS que o clustes deve responder.
Ele possui um dashboard de monitoramento e com um resumo de todas as entradas que estão no cluster.
```
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-rbac.yaml
$ kubectl apply -f https://raw.githubusercontent.com/containous/traefik/v1.7/examples/k8s/traefik-ds.yaml
$ kubectl --namespace=kube-system get pods
```
Agora iremos configurar o DNS pelo qual o Traefik irá responder. No arquivo ui.yml, localizar a url, e fazer a alteração.
Após a alteração feita, iremos rodar o comando abaixo para aplicar o deployment no cluster.
```
$ chmod -R 777 /home/ubuntu/devops
$ cd devops/exercicios
$ vi ui.yml
$ kubectl apply -f ui.yml
```

### 5. Longhorn - Volumes

Dentro do Rancher entrar em "Apps & Marketplace" e procurar por Longhorn e clicar em Install

Para fazermos os exercícios do volume, iremos fazer o deployment do pod com o volume, que estará apontando
para um caminho no host.

Fazer o deployment do Longhorn.
```
$ kubectl apply -f mariadb-longhorn-volume.yml
$ kubectl exec -it mysql -n default
```

### 6. Graylog - LOG

O Graylog é a aplicação que iremos usar como agregador de logs do cluster. Os logs dos containers podem ser vistos pelo Rancher,
é um dos níveis de visualização. Pelo Graylog temos outras funcionalidades, e também é possível salvar para posterior pesquisa,
e muitas outras funcionalidades.

Para instalar o Graylog, iremos aplicar o template dele, que está em graylog.yml. Para isso, é preciso que sejam editados 2 pontos
no arquivo.

Linha 265 - value: http://graylog.rancher.<dominio>/api
Linha 341 - host: graylog.rancher.<dominio>

Substituir o {user}, pelo nome do aluno. Após substituir, aplicar e entrar no Graylog para configurar.
```
$ kubectl apply -f graylog.yml
```
Seguir os passos do instrutor para configuração do Graylog.
