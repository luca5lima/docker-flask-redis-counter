# Respostas: Primeira Prática (Docker)

## 1) Comunicação entre serviços:
Os containers Flask e Redis se comunicam através de uma Docker Network (rede virtual fechada).

## 2) Uso de nomes de serviço vs. IP:
Utilizar o nome do serviço (ex: redis) permite que o DNS interno do Docker resolva automaticamente o IP dinâmico do container, que muda a cada reinicialização.

## 3) Uso da flag --build:
É necessária ao realizar alterações no código-fonte para forçar o Docker a ignorar o cache antigo e reconstruir a imagem do zero com a nova versão da aplicação.

## 4) Função do Dockerfile:
É um roteiro (script) que define as instruções para criar a imagem da aplicação (sistema base, dependências e comandos de execução).

## 5) Função do Redis:
Atua como um banco de dados ágil em memória para incrementar e armazenar a variável do número de acessos.

## 6) Descoberta de endereço (App -> Redis):
O container Flask encontra o Redis usando o DNS embutido do Docker, que traduz a variável de ambiente (o nome do serviço) para o IP correspondente na rede virtual.

## 7) Função do Volume:
Garante a persistência dos dados no disco do host. Se o container do Redis for destruído, a contagem de acessos não será zerada.

## 8) Vantagem do Docker Compose:
Automatiza a subida, configuração de redes, volumes e variáveis de múltiplos containers em um único comando, eliminando a complexidade de execução manual no terminal.