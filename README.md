# Passo 1: Configuração do Ambiente Kubernetes
  Instale o Kubernetes:

Dependendo da sua escolha de implantação (microk8s, k3s ou EKS da AWS), siga as respectivas instruções de instalação fornecidas pelos provedores.

## 1. Crie um Namespace:

    kubectl create namespace wordpress

## Passo 2: Provisionamento do MySQL
## 1. Crie um arquivo de manifesto para o Deployment do MySQL:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: mysql-deployment
      namespace: wordpress
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: mysql
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - name: mysql
            image: mysql:latest
            env:
            - name: MYSQL_ROOT_PASSWORD
              value: YOUR_ROOT_PASSWORD
            - name: MYSQL_DATABASE
              value: wordpress
            ports:
            - containerPort: 3306
## 2. Crie um arquivo de manifesto para o Service do MySQL:

    apiVersion: v1
    kind: Service
    metadata:
      name: mysql-service
      namespace: wordpress
    spec:
      selector:
        app: mysql
      ports:
      - protocol: TCP
        port: 3306
        targetPort: 3306

## 3. Aplique os manifestos:

    kubectl apply -f mysql-deployment.yaml
    kubectl apply -f mysql-service.yaml

## Passo 3: Provisionamento do Wordpress

## 1. Crie um arquivo de manifesto para o Deployment do Wordpress:

    apiVersion: apps/v1
    kind: Deployment
    metadata:
      name: wordpress-deployment
      namespace: wordpress
    spec:
      replicas: 1
      selector:
        matchLabels:
          app: wordpress
      template:
        metadata:
          labels:
            app: wordpress
        spec:
          containers:
          - name: wordpress
            image: wordpress:latest
            env:
            - name: WORDPRESS_DB_HOST
              value: mysql-service
            - name: WORDPRESS_DB_USER
              value: root
            - name: WORDPRESS_DB_PASSWORD
              value: YOUR_ROOT_PASSWORD
            ports:
            - containerPort: 80
## 2. Crie um arquivo de manifesto para o Service do Wordpress:

    yaml
    Copy code
    apiVersion: v1
    kind: Service
    metadata:
      name: wordpress-service
      namespace: wordpress
    spec:
      selector:
        app: wordpress
      ports:
      - protocol: TCP
        port: 80
        targetPort: 80

## 3. Aplique os manifestos:

    kubectl apply -f wordpress-deployment.yaml
    kubectl apply -f wordpress-service.yaml

## Passo 4: Configuração do Ingress (opcional)
  Se desejar expor o Wordpress para o mundo externo, configure um Ingress. Aqui está um exemplo básico:

## 1. Crie um arquivo de manifesto para o Ingress:

    yaml
    Copy code
    apiVersion: networking.k8s.io/v1
    kind: Ingress
    metadata:
      name: wordpress-ingress
      namespace: wordpress
    spec:
      rules:
      - host: YOUR_DOMAIN
        http:
          paths:
          - pathType: Prefix
            path: /
            backend:
              service:
                name: wordpress-service
                port:
                  number: 80
 2. Aplique o manifesto:

  kubectl apply -f wordpress-ingress.yaml

## Passo 5: Teste e Validação

## 1. Verifique se os pods estão em execução:
    kubectl get pods -n wordpress

## 2. Acesse o Wordpress via navegador usando o IP ou o domínio configurado.

## Simulação de Falha e Recuperação
  Para simular a falha de um pod, delete-o:

    kubectl delete pod POD_NAME -n wordpress
    Substitua POD_NAME pelo nome do pod que deseja excluir.

O Kubernetes automaticamente provisionará um novo pod para manter o estado do sistema.
