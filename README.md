# Weather API - Execução no Kubernetes (Kind)

Este README fornece os passos para construir, carregar a imagem Docker e executar a aplicação `weather-api` no Kubernetes utilizando o **Kind**.

## Pré-requisitos

- Docker
- Kubernetes (Kind)
- Kubectl

## Passos para Execução

### 1. Criar o Cluster no Kind

Primeiro, crie um arquivo de configuração para o Kind com múltiplos nós. Crie um arquivo chamado `kind-cluster.yml` com o seguinte conteúdo:

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
nodes:
  - role: control-plane
  - role: worker
  - role: worker
  - role: worker
```

Depois, execute o comando abaixo para criar o cluster com o Kind:

```bash
kind create cluster --config kind-cluster.yml --name demo-cluster-3
```

Isso cria um cluster chamado `demo-cluster-3` com 1 nó de controle e 3 nós de trabalho.

### 2. Construir a Imagem Docker

Primeiro, você precisa construir a imagem Docker da aplicação:

```bash
docker build -t weather-api .
```

Isso cria a imagem `weather-api` localmente.

### 3. Verificar as Imagens Docker

Após a construção da imagem, você pode verificar se ela foi criada corretamente:

```bash
docker images
```

### 4. Carregar a Imagem Docker para o Kind

Agora, carregue a imagem Docker para o seu cluster Kubernetes no Kind:

```bash
kind load docker-image weather-api --name demo-cluster-3
```

### 5. Aplicar o Manifesto do Kubernetes

Aplique o arquivo `deploy.yml`, que contém o Deployment e o Service para a aplicação:

```bash
kubectl apply -f deploy.yml
```

### 6. Verificar os Pods

Após a aplicação do manifesto, verifique se os pods estão sendo executados corretamente:

```bash
kubectl get pods
```

### 7. Verificar os Logs do Pod

Caso haja algum problema ou para monitorar a execução, você pode verificar os logs do pod:

```bash
kubectl logs deploy/api
```

### 8. Forward de Porta (Port-Forward)

Para acessar a aplicação localmente, use o comando `kubectl port-forward` para encaminhar a porta do serviço do Kubernetes para sua máquina local:

```bash
kubectl port-forward service/api 5088:80
```

Isso mapeia a porta `80` do serviço Kubernetes para a porta `5088` na sua máquina local.

### 9. Acessar a Aplicação

Agora você pode acessar a aplicação diretamente através de `curl` ou no navegador, acessando:

```bash
http://127.0.0.1:5088
```

### 10. Verificar os Serviços

Caso queira verificar os serviços em execução no cluster, use o comando:

```bash
kubectl get services
```

Para obter mais detalhes sobre o serviço, como a porta de acesso, você pode usar:

```bash
kubectl describe service api
```

### 11. Executar o Pod (Opcional)

Se você precisar acessar o pod diretamente para debugging ou inspeção, você pode executar um shell interativo:

```bash
kubectl exec -it <nome-do-pod> -- bash
```

### 12. Reiniciar o Deployment (Opcional)

Se você precisar atualizar ou reiniciar os recursos no Kubernetes, você pode aplicar novamente o manifesto:

```bash
kubectl apply -f deploy.yml
```

Ou reiniciar o Deployment diretamente:

```bash
kubectl rollout restart deployment/api
```

### 13. Verificar o Status do Pod

Se necessário, você pode verificar novamente o status do pod após uma atualização ou reinício:

```bash
kubectl get pods
```

### Observações

- **EXPOSE no Dockerfile**: A aplicação está configurada para escutar na porta `8080` dentro do container. No Kubernetes, mapeamos essa porta para a porta `80` do serviço.
- **Port-Forwarding**: O `kubectl port-forward` permite que você acesse a aplicação localmente através da porta que você configurou (`5088:80`).
- **Ajustes em Produção**: Para produção, considere a criação de um serviço LoadBalancer ou NodePort no Kubernetes em vez de usar port-forwarding.
