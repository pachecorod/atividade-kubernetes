# WordPress + MYSQL com o Kubernetes

## 📋 Pré-requisitos
> Ter o Docker e Kubernetes instalados em seu computador.

## 🔧 Instalação
### Opção 1: Docker Desktop, habilitando um cluster com Kubernetes.
Docker Desktop
> https://docs.docker.com/engine/install/

Kubernetes
> **Observação:** Com o Docker Desktop instalado você só precisa habilitar o kubernetes nas configurações.  

![MicrosoftTeams-image](https://user-images.githubusercontent.com/112576171/202451568-98bb22f4-f33d-4285-ad56-a1f0ab5cdc61.png)

### Opção 2: Docker Desktop com minikube.
Docker  
> https://docs.docker.com/engine/install/  

Kubernetes  
> Instalar o Minikube: https://minikube.sigs.k8s.io/docs/start/

## 🚀 Começando  
### 1. Primeiramente, crie um diretório para o seu projeto. 
```
mkdir labwordpress
```
### 2. Mude para o seu diretório.  
```
cd ./labwordpress 
```  
### 3. Criação do namespace.
####  Criando o namespace através de linha de comando.
```  
kubectl create namespace <nome_do_namespace>  
```  
Ou  
#### Criando o namespace através do arquivo de configuração.  
> **Observação:** Crie o arquivo namespace.yaml.
```
New-Item namespace.yaml
```
#### Após criado o arquivo, abra-o em um editor de texto e adicione as seguintes linhas.  
```ruby
apiVersion: v1
kind: Namespace
metadata:
  name: labwordpress 
```  
* ```apiVersion:v1``` -> Versão da api.
* ```kind: Namespace``` -> tipo de objeto a ser criado.
* ```metadata:``` -> especifica informações do objeto a ser criado.
   * ```name: labwordpress``` -> definimos o nome do namespace.
   
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f namespace.yaml  
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

### 🛠️ Criando o Secret.
#### Primeiro iremos criar o arquivo secrets, onde será armazenado as nossas senhas para acesso ao MYSQL.  
Crie o arquivo mysql-secret.yaml.  
```
New-Item mysql-secret.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: v1
kind: Secret
metadata:
  name: labwordpress-secret
  namespace: labwordpress
  labels: 
    app: wordpress
type: Opaque
data:
  mysql-root-username: <seu-usuario>
  mysql-root-password: <sua-senha>
```  
* ```apiVersion:v1``` -> Versão da api.
* ```kind: Secret``` -> tipo de objeto a ser criado.
* ```metadata:``` -> especifica informações do objeto a ser criado.
   * ```name: labwordpress-secret``` -> definimos o nome do secret.
   * ```namespace: labwordpress``` -> definimos o nome do namespace.
* ```data:``` -> especifica os dados inseridos no arquivo. Neste caso usuário e senha.  

Se você estiver em uma máquina windows digite o seguinte comando:  
```
wsl -d <distribuicao-do-wsl> -u <usuario>
```  
> **Observação:** Exemplo:  
```
wsl -d ubuntu -u wpkube
```  
> **Observação:** Defina um usuário e senha para acesso ao MYSQL e WordPress. Os valores usados no arquivo para definir usuário e senha precisam ser criptografados usando o comando abaixo. Depois, substitua os valores no arquivo  ```mysql-secret.yaml```.
```
 echo -n '<seu-usuario>' | tr -d \\n | base64 -w 0 
 ```  
 ```
 echo -n '<sua-senha>' | tr -d \\n | base64 -w 0 
 ```  
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-secret.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

### 🛠️ Subindo a aplicação MYSQL.
#### 1. Criando o service do MYSQL.
Crie o arquivo mysql-service.yaml.  
```
New-Item mysql-service.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.
```ruby
apiVersion: v1
kind: Service
metadata:
  name: wordpress-mysql
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 3308
  selector:
    app: wordpress
    tier: mysql
  clusterIP: None
```  
* Na primeira linha em ```apiVersion:v1``` definimos a versão da api. Na segunda, o tipo de objeto a ser criado```kind: Service```.  
* Em ```metadata:``` especificamos as informações do objeto. Como:  
   * ```name: wordpress-mysql``` -> definimos o nome do service.  
   * ```namespace: labwordpress``` -> definimos o nome do namespace.  
   * Em ```ports:``` especificamos a porta do service: ```- port: 3308```.  
   
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-service.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

#### 2. Criando o PersistentVolumeClaim do MYSQL.
Crie o arquivo mysql-persistent-volume.yaml.  
```
New-Item mysql-persistent-volume.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: mysql-pv-claim
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```  
* Na primeira linha em ```apiVersion:v1``` definimos a versão da api. Na segunda, o tipo de objeto a ser criado```kind: PersistentVolumeClaim```.  
* Em ```metadata:``` especificamos as informações do objeto. Como:  
   * ```name: mysql-pv-claim``` -> definimos o nome do pvc.  
   * ```namespace: labwordpress``` -> definimos o nome do namespace.  
   * E em ```storage:``` definimos o tamanho do pvc.  
   
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-persistent-volume.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

#### 3. Criando o arquivo ConfigMap do MYSQL e WordPress.  
Crie o arquivo mysql-configmap.yaml.  
```
New-Item mysql-configmap.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.
```ruby
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: labwordpress
data:
  db: "wordpress"
  mysql-service: "wordpress-mysql"
```  
* ```apiVersion:v1``` -> Versão da api.
* ```kind: ConfigMap``` -> tipo de objeto a ser criado.
* ```metadata:``` -> especifica informações do objeto a ser criado.
   * ```name: mysql-configmap``` -> definimos o nome do secret.
   * ```namespace: labwordpress``` -> definimos o nome do namespace.
* ```data:``` -> especifica os dados inseridos no arquivo. Neste caso o nome da database e o service do MYSQL.  

> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-configmap.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.

#### 4. Criando o arquivo Deployment do MYSQL.
Crie o arquivo mysql-deployment.yaml.  
```
New-Item mysql-deployment.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-mysql
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: mysql
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: mysql
    spec:
      containers:
      - image: mysql:8.0.31
        name: mysql
        env:
        - name: MYSQL_ROOT_PASSWORD 
          valueFrom:
            secretKeyRef:
              name: labwordpress-secret
              key: mysql-root-password
        - name: MYSQL_DATABASE
          valueFrom:
            configMapKeyRef:
              name: mysql-configmap
              key: db
        - name: MYSQL_USER
          valueFrom:
            secretKeyRef:
              name: labwordpress-secret
              key: mysql-root-username
        - name: MYSQL_PASSWORD
          valueFrom:
            secretKeyRef:
              name: labwordpress-secret
              key: mysql-root-password
        ports:
        - containerPort: 3308
          name: mysql
        volumeMounts:
        - name: mysql-persistent-storage-lab
          mountPath: /var/lib/mysql
      volumes:
      - name: mysql-persistent-storage-lab
        persistentVolumeClaim:
          claimName: mysql-pv-claim
```  
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-deployment.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

### 🛠️ Subindo a aplicação WordPess.
#### 1. Criando o service do Wordpress.  
Crie o arquivo wordpress-service.yaml.   
```
New-Item wordpress-service.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: v1
kind: Service
metadata:
  labels:
    app: wordpress
  name: wordpress
  namespace: labwordpress
spec:
  ports:
  - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: ClusterIP
```  
* Na primeira linha em ```apiVersion:v1``` definimos a versão da api. Na segunda, o tipo de objeto a ser criado```kind: Service```.  
* Em ```metadata:``` especificamos as informações do objeto. Como:  
   * ```name: wordpress``` -> definimos o nome do service.  
   * ```namespace: labwordpress``` -> definimos o nome do namespace.  
   * Em ```ports:``` especificamos a porta do service: ```- port: 80```.  

> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f wordpress-service.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

#### 2. Criando o PersistentVolumeClaim do WordPress.
Crie o arquivo wordpress-persistent-volume.yaml.  
```
New-Item wordpress-persistent-volume.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: wp-pv-claim
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 3Gi
```  
* Na primeira linha em ```apiVersion:v1``` definimos a versão da api. Na segunda, o tipo de objeto a ser criado```kind: PersistentVolumeClaim```.  
* Em ```metadata:``` especificamos as informações do objeto. Como:  
   * ```name: wp-pv-claim``` -> definimos o nome do pvc.  
   * ```namespace: labwordpress``` -> definimos o nome do namespace.  
   * E em ```storage:``` definimos o tamanho do pvc.  
   
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f wordpress-persistent-volume.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

#### 3. Criando o arquivo Deployment do WordPress.
Crie o arquivo wordpress-deployment.yaml.  
```
New-Item wordpress-deployment.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress
  namespace: labwordpress
  labels:
    app: wordpresss
spec:
  selector:
    matchLabels:
      app: wordpress
      tier: frontend
  strategy:
    type: Recreate
  template:
    metadata:
      labels:
        app: wordpress
        tier: frontend
    spec:
      containers:
      - image: wordpress:6.0.2
        name: wordpress
        env:
        - name: WORDPRESS_DB_HOST
          valueFrom:
            configMapKeyRef:
              name: mysql-configmap
              key: mysql-service
        - name: WORDPRESS_DB_USER
          valueFrom:
            secretKeyRef:
              name: labwordpress-secret
              key: mysql-root-username
        - name: WORDPRESS_DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: labwordpress-secret
              key: mysql-root-password
        ports:
        - containerPort: 80
          name: wordpress
        volumeMounts:
        - name: wordpress-persistent-storage-lab
          mountPath: /var/www/html
      volumes:
      - name: wordpress-persistent-storage-lab
        persistentVolumeClaim:
          claimName: wp-pv-claim
```  
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f wordpress-deployment.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

Pronto! Agora você pode conferir se está tudo funcionando usando o comando:  
```  
kubectl get all -n labwordpress
```  
![WhatsApp Image 2022-11-19 at 19 26 37](https://user-images.githubusercontent.com/112576171/203058094-e30c9cd6-f8f3-4feb-9ad8-1460a43ae0f7.jpeg)  

#### 4. Implementando o ingress do WordPress.  
Como configuramos nossa aplicação WordPress do tipo ClusterIP, para termos acesso externo a essa aplicação precisamos criar um ingress. Mas primeiro vamos implementar um ingress controller em nosso cluster rodando o comando abaixo:

``` 
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.5.1/deploy/static/provider/cloud/deploy.yaml  
``` 
Confira se os pods estão rodando com o comando abaixo:
``` 
kubectl get pods -n ingress-nginx
```

Antes de implementar nosso ingress, vamos adicionar no arquivo ``` C:\Windows\System32\drivers\etc\hosts ``` do nosso computador, o nosso IP e o DNS que escolhemos para que o ingress redirecione nosso serviço WordPress.  

A linha vai ficar assim: ```127.0.0.1  wpkubernetes.com```.

> **Observação:** lembre de trocar o ```wpkubernetes.com``` pelo seu DNS.  

Agora podemos partir para nosso arquivo de configuração.  

Crie o arquivo wordpress-ingress.yaml.  
```
New-Item wordpress-ingress.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: wordpress
  namespace: labwordpress
spec:
  ingressClassName: nginx
  rules:
  - host: wpkubernetes.com
    http:
      paths:
      - backend:
          service:
            name: wordpress
            port:
              number: 80
        path: /
        pathType: Prefix
```  
> **Observação:** na linha ``` - host: wpkubernetes.com ``` você modifica colocando seu DNS.  

> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f wordpress-ingress.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra.  

> Você pode consultar se o ingress foi implementado rodando o seguinte comando:
```  
kubectl get ingress -n labwordpress
```
Na coluna ```HOSTS```, copie a URI e cole no seu browser para acessar a aplicação do WordPress.  

![WhatsApp Image 2022-11-19 at 19 27 45](https://user-images.githubusercontent.com/112576171/203058033-27f4d29b-9472-4d70-b319-46368df521b4.jpeg)

Pronto! Você tem acesso a sua aplicação do WordPress. Agora, vamos partir para as configurações iniciais.

## 1. Escolha do idioma do site  
Selecione o idioma e clique em ```Continue```. 

![WhatsApp Image 2022-11-18 at 12 08 37](https://user-images.githubusercontent.com/112576171/203059995-2cf4749f-4766-4ded-a13c-0223169bc23e.jpeg)  

## 2. Fazer a instalação  
Preencha os campos com as suas informações. Escolha um título para o site e preencha com o usuário, senha e e-mail.  
No final, clique em ```Install WordPress```  

![WhatsApp Image 2022-11-18 at 12 10 06](https://user-images.githubusercontent.com/112576171/203060039-7a3ecf61-72af-4a3c-a716-ef4f06c9ff91.jpeg)  

## 3. Fazer o login  
Digite seu usuário e senha. Depois, clique em ```Log in```.  

![WhatsApp Image 2022-11-19 at 17 04 15](https://user-images.githubusercontent.com/112576171/203060087-522d0097-3d55-42a7-8501-7a8f4f54218b.jpeg)  

## 4. Acesso ao Dashboard  
Você agora tem acesso ao seu dashboard do WordPress.  

![WhatsApp Image 2022-11-19 at 17 05 46](https://user-images.githubusercontent.com/112576171/203060128-624506f6-0b85-46f0-bcd7-b5259a51d4ad.jpeg)  

## 📌 Versões utilizadas 
Docker Desktop: 20.10.17      
Kubernetes: v1.25.0    
Wordpress: 6.0.2     
MySQL: 8.0.31      


## ✒️ Autoras
Maria Eduarda Lopes Maldonado.  
Márcia Cristina Henriques da Cruz.
