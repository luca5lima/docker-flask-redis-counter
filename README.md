# 🐳 Docker Web Counter - Arquitetura e Persistência

Este projeto demonstra a criação e orquestração de uma aplicação web conteinerizada utilizando **Docker** e **Docker Compose**. O sistema consiste em uma aplicação web desenvolvida em **Flask** que se comunica com um banco de dados em memória **Redis** para contabilizar e persistir o número de acessos de forma distribuída.

Projeto desenvolvido para a disciplina de Sistemas Distribuídos na Universidade Federal do Ceará (UFC) - Campus Crateús.

## 🏗️ Arquitetura

O tráfego flui da seguinte maneira:
`Navegador → Container App (Flask) → Rede Interna Docker → Container Banco (Redis) → Volume (Persistência)`

*   **App (Flask):** Servidor web que recebe as requisições e exibe a interface.
*   **Redis:** Banco de dados em memória para armazenar o contador de forma rápida.
*   **Docker Network:** Garante a comunicação isolada e segura entre os containers.
*   **Docker Volume:** Garante que os dados do contador não sejam perdidos ao reiniciar o servidor.

## 🚀 Como executar o projeto

**Pré-requisitos:** Ter o [Docker](https://www.docker.com/) instalado na sua máquina.

1. Clone este repositório:
   ```bash
   git clone [https://github.com/seu-usuario/nome-do-repositorio.git](https://github.com/seu-usuario/nome-do-repositorio.git)
   cd nome-do-repositorio
   ```
2. Suba a infraestrutura completa utilizando o Docker Compose:
   ```bash
   docker compose up -d --build
   ```
3. Acesse a aplicação no seu navegador:
   - http://localhost:5000
5. Para derrubar a infraestrutura sem perder os dados armazenados no volume:
   ```bash
   docker compose down
   ```
