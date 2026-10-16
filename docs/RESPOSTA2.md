# Respostas: Segunda Prática (Kubernetes)

## 1) O que representa o nó apresentado pelo comando `kubectl get nodes`?

Um **nó (node)** é uma máquina, física ou virtual, que faz parte do cluster Kubernetes e onde os Pods são executados. Cada nó roda os componentes necessários para isso, como o `kubelet` (que gerencia os Pods do nó) e o runtime de containers.

No Minikube, o cluster possui um único nó, que acumula as funções de plano de controle (*control-plane*) e de trabalho (*worker*). Por isso o comando `kubectl get nodes` mostra apenas o nó `minikube`, com status `Ready` quando está saudável e pronto para receber Pods.

## 2) Por que a aplicação deve utilizar `redis` em `REDIS_HOST`, em vez do endereço IP de um Pod Redis?

Porque o IP de um Pod **não é estável**. Sempre que o Pod é recriado (por falha, atualização ou remoção), o Kubernetes atribui um novo IP. Se a aplicação usasse o IP fixo, perderia a conexão com o Redis assim que o Pod fosse substituído.

O nome `redis` é o nome do **Service** do Redis. O Kubernetes possui um DNS interno que resolve esse nome para o Service, que por sua vez encaminha o tráfego para o Pod atual do Redis. Assim, a aplicação continua funcionando mesmo que o Pod mude de IP, sem precisar alterar nenhuma configuração.

## 3) Por que a aplicação possui 3 réplicas enquanto o Redis possui apenas 1 réplica nesta prática?

A aplicação Flask é **sem estado** (*stateless*): ela não guarda dados localmente, apenas lê e atualiza o contador no Redis. Por isso, pode ter várias réplicas idênticas, que dividem as requisições entre si, aumentando a capacidade e a disponibilidade.

O Redis, por outro lado, é **com estado** (*stateful*): ele guarda o valor do contador em memória. Se houvesse várias réplicas, cada uma teria seu próprio contador independente, e o número de acessos exibido dependeria de qual réplica respondeu, gerando valores inconsistentes. Para replicar um banco de dados com consistência seria necessário configurar replicação (por exemplo, Redis com réplicas ou Sentinel/Cluster), o que está fora do escopo desta prática.

## 4) Qual é a função do Service do Redis?

O Service do Redis fornece um **ponto de acesso estável** (nome e IP fixos dentro do cluster) para os Pods do Redis, que são efêmeros e podem mudar de IP quando recriados.

Ele funciona assim:

- recebe as conexões destinadas ao nome `redis` na porta 6379;
- seleciona os Pods que possuem a label `app: redis`;
- encaminha as requisições para esses Pods.

Dessa forma, os Pods da aplicação sempre encontram o Redis pelo mesmo nome, independentemente de qual Pod esteja ativo no momento. Por ser do tipo `ClusterIP` (padrão), o Redis fica acessível apenas dentro do cluster, o que também é mais seguro.

## 5) Como o Service `contador` sabe quais Pods devem receber as requisições?

Por meio de **labels e selectors**. Os Pods criados pelo Deployment da aplicação recebem a label `app: contador` (definida no `template` do Deployment). O Service `contador` possui o seletor `app: contador` no campo `selector`.

O Kubernetes monitora continuamente quais Pods possuem labels que correspondem ao seletor do Service e mantém uma lista atualizada deles (os *endpoints*). Quando um Pod é criado, removido ou recriado, essa lista é atualizada automaticamente, e o Service passa a distribuir as requisições apenas entre os Pods ativos que correspondem ao seletor.

## 6) O que aconteceu com os Pods quando o número de réplicas foi alterado de 3 para 5 e depois para 2?

- **De 3 para 5:** o Deployment detectou que o estado desejado (5 Pods) era diferente do estado atual (3 Pods) e criou **2 Pods novos**, a partir da mesma imagem `contador:1.0`. Após alguns instantes, o `kubectl get pods` mostrou 5 Pods da aplicação em execução.
- **De 5 para 2:** o Deployment passou a ter mais Pods do que o desejado e **encerrou 3 Pods**, que aparecem por um curto período com status `Terminating` antes de desaparecerem. Ao final, restaram 2 Pods da aplicação.

Em ambos os casos o Kubernetes age para levar o estado atual ao estado desejado, declarado no campo `replicas`. O Pod do Redis não foi afetado, pois pertence a outro Deployment.

## 7) O Pod removido foi simplesmente perdido? Explique o que aconteceu após sua exclusão.

Não. O Pod removido foi **substituído automaticamente**. O Deployment (por meio do ReplicaSet) mantém o número de réplicas definido no estado desejado. Ao excluir um Pod, o estado atual passou a ter menos Pods do que o desejado, e o Kubernetes criou imediatamente um **novo Pod** (com novo nome e novo IP) para repor o que foi removido.

Esse comportamento é chamado de **self-healing** (autorrecuperação): o Kubernetes detecta a divergência entre o estado atual e o desejado e a corrige sem intervenção manual. Enquanto o novo Pod sobe, os demais continuam atendendo as requisições, então a aplicação permanece disponível.

## 8) O que caracteriza um Rolling Update? Por que ele é útil em aplicações que precisam permanecer disponíveis durante uma atualização?

O **Rolling Update** é a estratégia padrão de atualização dos Deployments. Em vez de derrubar todos os Pods da versão antiga e só depois subir os da nova, o Kubernetes faz a troca **gradual**: cria alguns Pods da nova versão, espera que fiquem prontos e só então remove Pods da versão antiga, repetindo o processo até que todos estejam atualizados. A quantidade de Pods trocados por vez é controlada pelos parâmetros `maxSurge` e `maxUnavailable`.

Ele é útil porque, durante toda a atualização, sempre existem Pods ativos atendendo as requisições. Isso evita **indisponibilidade** (*downtime*), permite atualizar a aplicação em produção sem que os usuários percebam e reduz o risco: se a nova versão apresentar problemas, a atualização pode ser interrompida ou revertida antes de afetar todos os Pods.

## 9) Qual é a função do comando `kubectl rollout undo`?

O comando `kubectl rollout undo deployment/contador-deployment` **desfaz a última atualização** do Deployment, retornando-o para a **revisão anterior**. O Kubernetes guarda o histórico de revisões (visível com `kubectl rollout history`) e, ao executar o `undo`, realiza um novo Rolling Update, desta vez de volta à versão anterior (no caso, a imagem `contador:1.0`).

É o recurso de **rollback**, útil quando uma nova versão apresenta falhas: permite restaurar rapidamente uma versão que funcionava, sem precisar editar manifestos nem reconstruir imagens, e mantendo a aplicação disponível durante a reversão.
