#**Aplicativo de monitoramento de recursos nativo em cloud no K8s!**

##Coisas que você vai aprender 🤯

```
1.Python: como criar um aplicativo de monitoramento em Python usando Flask e psutil.
2.Como executar um aplicativo Python localmente.
3.Aprender Docker e como dockerizar um aplicativo Python em um contêiner.
    1.Criar Dockerfile.
    2.Construir uma imagem Docker.
    3.Executar um contêiner.
    4.Comandos Docker.
4.Criar um repositório ECR usando o Python Boto3 e enviar a imagem Docker para o ECR.
5.Aprender Kubernetes e como criar um cluster EKS e um grupos de nós.
6.Criar deployments e services usando o Python.
```

##**Pré-requisitos!**

-[x]Conta AWS.
-[x]Configurar a CLI da AWS.
-[x]Python3 instalado.
-[x]Docker e Kubectl instalados.
-[x]Editor de código (Vscode).

#✨Vamos começar o projeto!✨

## **Parte 1: Implantação do aplicativo Flask localmente**

### **Passo 1: Clonar o código**

Clone o código do repositório:

```
$ git clone <repository_url>
```

### **Passo 2: Instalar as dependências**

O aplicativo usa as bibliotecas **psutil, Flask, Plotly e boto3**. Instale-as usando o pip:

```
pip3 install -r requirements.txt
```

### **Passo 3: Executar o aplicativo**

Para executar o aplicativo, navegue até o diretório raiz do projeto e execute o seguinte comando:

```
$ python3 app.py
```

Isso iniciará o servidor Flask em **'localhost:5000'.** Acesse http://localhost:5000/ em seu navegador para acessar o aplicativo.

## **Parte 2: Colocando o aplicativo Flask em um contêiner Docker**

### **Passo 1: Criar um Dockerfile**

Crie um **Dockerfile** no diretório raiz do projeto com o seguinte conteúdo:

```
# Use the official Python image as the base image
FROM python:3.9-slim-buster

# Set the working directory in the container
WORKDIR /app

# Copy the requirements file to the working directory
COPY requirements.txt .

RUN pip3 install --no-cache-dir -r requirements.txt

# Copy the application code to the working directory
COPY . .

# Set the environment variables for the Flask app
ENV FLASK_RUN_HOST=0.0.0.0

# Expose the port on which the Flask app will run
EXPOSE 5000

# Start the Flask app when the container is run
CMD ["flask", "run"]
```

### **Passo 2: Construir a imagem Docker*

Para construir a imagem Docker, execute o seguinte comando:

```
$ docker build -t <image_name> .
```

### **Passo 3: Executar o contêiner Docker**

Para executar o contêiner Docker, execute o seguinte comando:

```
$ docker run -p 5000:5000 <image_name>
```

Isso iniciará o servidor Flask em um contêiner Docker em **'localhost:5000'**. Acesse http://localhost:5000/ em seu navegador para acessar o aplicativo.

## **Parte 3: Enviando a imagem Docker para o ECR**

### **Passo 1: Criar um repositório ECR**

Crie um repositório ECR usando o Python:

```
import boto3

# Create an ECR client
ecr_client = boto3.client('ecr')

# Create a new ECR repository
repository_name = 'my-ecr-repo'
response = ecr_client.create_repository(repositoryName=repository_name)

# Print the repository URI
repository_uri = response['repository']['repositoryUri']
print(repository_uri)
```

### **Passo 2: Enviar a imagem Docker para o ECR**

Envie a imagem Docker para o ECR usando os comandos push no terminal:

```
 $ docker push <ecr_repo_uri>:<tag>
```

## **Parte 4: Criando um cluster EKS e implantando o aplicativo usando Python**

### **Passo 1: Criar um cluster EKS**

### **Passo 2: Criar um grupo de nós**

### **Passo 3: Criar deployment and service**

```jsx
from kubernetes import client, config

# Load Kubernetes configuration
config.load_kube_config()

# Create a Kubernetes API client
api_client = client.ApiClient()

# Define the deployment
deployment = client.V1Deployment(
    metadata=client.V1ObjectMeta(name="my-flask-app"),
    spec=client.V1DeploymentSpec(
        replicas=1,
        selector=client.V1LabelSelector(
            match_labels={"app": "my-flask-app"}
        ),
        template=client.V1PodTemplateSpec(
            metadata=client.V1ObjectMeta(
                labels={"app": "my-flask-app"}
            ),
            spec=client.V1PodSpec(
                containers=[
                    client.V1Container(
                        name="my-flask-container",
                        image="568373317874.dkr.ecr.us-east-1.amazonaws.com/my-cloud-native-repo:latest",
                        ports=[client.V1ContainerPort(container_port=5000)]
                    )
                ]
            )
        )
    )
)

# Create the deployment
api_instance = client.AppsV1Api(api_client)
api_instance.create_namespaced_deployment(
    namespace="default",
    body=deployment
)

# Define the service
service = client.V1Service(
    metadata=client.V1ObjectMeta(name="my-flask-service"),
    spec=client.V1ServiceSpec(
        selector={"app": "my-flask-app"},
        ports=[client.V1ServicePort(port=5000)]
    )
)

# Create the service
api_instance = client.CoreV1Api(api_client)
api_instance.create_namespaced_service(
    namespace="default",
    body=service
)
```

Certifique-se de editar o nome da imagem na linha 25 com a URI da sua imagem.

- Uma vez que você execute este arquivo através do comando “python3 eks.py”, o deployment e service serão criados.
- Verifique executando os seguintes comandos:

```jsx
kubectl get deployment -n default (check deployments)
kubectl get service -n default (check service)
kubectl get pods -n default (to check the pods)
```

Assim que o pod estiver em execução, execute o redirecionamento de porta para expor o serviço

kubectl port-forward service/<nome_do_servico> 5000:5000