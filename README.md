# WordPress + MYSQL com o Kubernetes

## 📋 Pré-requisitos
> Ter o Docker e Kubernetes instalados em seu computador.

## 🔧 Instalação
### Opção 1
Docker Desktop
> https://docs.docker.com/engine/install/

Kubernetes
> **Observação:** Com o Docker Desktop instalado você só precisa habilitar o kubernetes nas configurações.  

![MicrosoftTeams-image](https://user-images.githubusercontent.com/112576171/202451568-98bb22f4-f33d-4285-ad56-a1f0ab5cdc61.png)

### Opção 2 
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
  mysql-root-password: <sua_senha>
```  
> **Observação:** os valor usado no arquivo para definir a senha precisa ser criptografado usando o comando abaixo.   
```
 echo -n '<sua_senha>' | base64  
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
  name: mysql-service
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
> Salve o arquivo e use o comando abaixo.  
```  
kubectl apply -f mysql-persistent-volume.yaml
```  
⚠️ Comando apply
> Lembre-se que o comando apply deve ser sempre onde o arquivo se encontra. 

#### 3. Criando o arquivo Deployment do MYSQL.
Crie o arquivo mysql-deployment.yaml.  
```
New-Item mysql-deployment.yaml
```  
Abra o arquivo em um editor e adicione as seguintes linhas.  
```ruby
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mysql
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  replicas: 1
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
  name: wordpress-service
  namespace: labwordpress
  labels:
    app: wordpress
spec:
  ports:
    - port: 80
  selector:
    app: wordpress
    tier: frontend
  type: LoadBalancer
```  
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
    app: wordpress
spec:
  replicas: 1
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
          value: mysql
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
