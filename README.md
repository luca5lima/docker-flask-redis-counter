# 🐳 Docker Web Counter - Docker Compose e Kubernetes

Este projeto demonstra a criação, a orquestração e a implantação de uma aplicação web conteinerizada. O sistema consiste em uma aplicação desenvolvida em **Flask** que se comunica com um banco de dados em memória **Redis** para contabilizar e persistir o número de acessos.

A aplicação é executada de duas formas:

- **Primeira Prática:** com **Docker** e **Docker Compose**, em uma única máquina.
- **Segunda Prática:** com **Kubernetes** (Minikube), com múltiplas réplicas, balanceamento de carga, recuperação automática, escalabilidade e atualizações sem indisponibilidade.

Projeto desenvolvido para a disciplina de Sistemas Distribuídos na Universidade Federal do Ceará (UFC) - Campus Crateús.

## 🏗️ Arquitetura

### Primeira Prática (Docker Compose)

`Navegador → Container App (Flask) → Rede Interna Docker → Container Redis → Volume (Persistência)`

- **App (Flask):** Servidor web que recebe as requisições e exibe a interface.
- **Redis:** Banco de dados em memória que armazena o contador.
- **Docker Network:** Garante a comunicação entre os containers, que se encontram pelo nome do serviço.
- **Docker Volume:** Garante que os dados do contador não sejam perdidos ao reiniciar os containers.

### Segunda Prática (Kubernetes)

`Navegador → Service contador (NodePort) → Pods do Flask (3 réplicas) → Service redis (ClusterIP) → Pod do Redis`

- **Deployment `contador-deployment`:** mantém 3 Pods da aplicação Flask.
- **Deployment `redis-deployment`:** mantém 1 Pod do Redis.
- **Service `contador` (NodePort):** expõe a aplicação para fora do cluster e distribui as requisições entre os Pods.
- **Service `redis` (ClusterIP):** fornece um endereço estável (`redis`) para o Redis, mesmo que o Pod seja recriado e mude de IP.

## 📁 Estrutura de arquivos

```
.
└── app/
   ├── app.py                   # Aplicação Flask
   ├── requirements.txt         # Dependências Python
   ├── Dockerfile               # Receita da imagem da aplicação
   └── docker-compose.yml       # Execução com Docker Compose (Primeira Prática)
└── k8s/                        # Manifestos Kubernetes (Segunda Prática)
    ├── app-deployment.yaml
    ├── app-service.yaml
    ├── redis-deployment.yaml
    └── redis-service.yaml
```

> A localização exata de cada arquivo pode variar. Execute os comandos a partir da pasta onde estão o `Dockerfile` e o `docker-compose.yml`.

## 🚀 Primeira Prática: executando com Docker Compose

**Pré-requisito:** Ter o [Docker](https://www.docker.com/) instalado.

1. Clone este repositório:
   ```bash
   git clone https://github.com/luca5lima/docker-flask-redis-counter
   cd nome-do-repositorio
   ```
2. Suba a infraestrutura completa:
   ```bash
   docker compose up -d --build
   ```
3. Acesse a aplicação no navegador:
   - http://localhost:5050
4. Derrube a infraestrutura sem perder os dados armazenados no volume:
   ```bash
   docker compose down
   ```

> Evite `docker compose down -v`, pois a flag `-v` remove o volume e zera o contador.

## ☸️ Segunda Prática: executando com Kubernetes

**Pré-requisitos:** [Docker](https://www.docker.com/), [Minikube](https://minikube.sigs.k8s.io/) e [kubectl](https://kubernetes.io/docs/tasks/tools/) instalados.

### 1. Iniciar o cluster

```bash
minikube start
kubectl get nodes
```

O comando `kubectl get nodes` lista os nós do cluster. Um nó é uma máquina (física ou virtual) onde os Pods são executados. No Minikube, existe um único nó que representa o cluster inteiro.

### 2. Construir a imagem dentro do Minikube

A imagem precisa estar disponível dentro do Minikube para que o Kubernetes possa usá-la sem um registry externo. A partir da pasta do `Dockerfile`:

```bash
minikube image build -t contador:1.0 .
minikube image ls | grep contador
```

### 3. Aplicar os manifestos

```bash
kubectl apply -f k8s/
```

Verifique os recursos criados:

```bash
kubectl get pods
kubectl get deployments
kubectl get services
```

O esperado é: **3 Pods** da aplicação, **1 Pod** do Redis, **2 Deployments** e **2 Services**.

### 4. Acessar a aplicação

```bash
minikube service contador --url
```

Abra o endereço apresentado no navegador. O contador deve funcionar da mesma forma que na primeira prática.

> No Windows, com o driver Docker, mantenha o terminal aberto enquanto estiver acessando a aplicação.

### 5. Escalar a aplicação

```bash
kubectl scale deployment contador-deployment --replicas=5
kubectl get pods

kubectl scale deployment contador-deployment --replicas=2
kubectl get pods
```

O Kubernetes cria ou remove Pods até que o número de réplicas corresponda ao estado desejado.

### 6. Self-Healing (recuperação automática)

Remova um Pod da aplicação:

```bash
kubectl get pods
kubectl delete pod <nome-do-pod>
kubectl get pods
```

O Deployment detecta que o estado atual difere do desejado e cria automaticamente um novo Pod para substituir o removido.

### 7. Rolling Update

Altere a mensagem da aplicação (por exemplo, para `Olá, Kubernetes!`), reconstrua a imagem com uma nova versão e atualize o Deployment:

```bash
minikube image build -t contador:2.0 .
```

No arquivo `k8s/app-deployment.yaml`, troque a imagem para `contador:2.0` e aplique:

```bash
kubectl apply -f k8s/app-deployment.yaml
kubectl rollout status deployment/contador-deployment
kubectl get pods
kubectl rollout history deployment/contador-deployment
kubectl rollout history deployment/contador-deployment --revision=2
```

Os Pods da versão antiga são substituídos gradualmente pelos da nova, e a aplicação permanece disponível durante a atualização.

### 8. Rollback

Para voltar à versão anterior:

```bash
kubectl rollout undo deployment/contador-deployment
kubectl rollout status deployment/contador-deployment
```

### 9. Encerrar

```bash
kubectl delete -f k8s/
minikube stop
```

## 📚 Conceitos principais

| Conceito | Função |
|---|---|
| **Dockerfile** | Descreve como construir a imagem da aplicação, garantindo que ela rode igual em qualquer ambiente. |
| **Docker Compose** | Define e executa vários containers (rede, volume, variáveis) com um único arquivo e um único comando. |
| **Pod** | Menor unidade do Kubernetes; executa um ou mais containers. |
| **Deployment** | Mantém o número desejado de Pods, recria os que falham e controla as atualizações. |
| **Service** | Fornece um endereço estável e balanceia as requisições entre os Pods selecionados por label. |
| **Rolling Update** | Substitui os Pods aos poucos, sem derrubar a aplicação. |
| **Rollback** | Retorna o Deployment para uma revisão anterior (`kubectl rollout undo`). |

## 📝 Observações

- O Redis roda com **1 réplica** porque cada instância mantém seu próprio contador em memória; várias réplicas gerariam contagens divergentes. A aplicação, por ser sem estado, pode ter várias réplicas.
- O Deployment do Redis na Segunda Prática não utiliza volume, então o contador é perdido se o Pod do Redis for recriado.
