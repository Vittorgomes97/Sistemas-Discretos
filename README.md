
Primeiro Projeto SED

# Sistemas-Discretos- Projeto da disciplina de Sistemas Discretos - UFCG - Período 25.2

# Descrição Detalhada do Projeto – Célula de Manufatura Automatizada


## 1. Visão Geral do Sistema


O sistema em estudo consiste em uma **célula de manufatura automatizada** composta por duas estações de processamento independentes (**Máquina 1 – M1** e **Máquina 2 – M2**), um **robô industrial compartilhado** responsável pelo transporte de peças, e uma **esteira de saída (buffer)** com capacidade de armazenamento limitada.


O objetivo principal da célula é transformar matéria-prima bruta em **peças acabadas**, garantindo que o fluxo de produção ocorra de forma contínua, segura e eficiente. O desafio central do projeto está no **controle da coordenação entre os recursos compartilhados**, evitando situações de **overflow**, **deadlock** e **operações inválidas**, comuns em sistemas automatizados mal supervisionados.


---


## 2. Detalhamento dos Componentes do Sistema


A seguir, são descritos o funcionamento físico e lógico de cada componente da planta.


### 2.1 Máquinas de Processamento (M1 e M2)


As máquinas de processamento são **idênticas em funcionamento**, porém **operam de forma independente**, podendo concluir seus ciclos em tempos distintos.


#### Funcionamento
- Cada máquina inicia em estado de repouso.
- Ao receber um comando de início, a máquina começa o processamento da peça (assume-se que a matéria-prima está sempre disponível).
- O tempo de processamento é indeterminado.
- Ao concluir o processamento, a máquina entra no estado **"Peça Pronta" (Done)** e aguarda a retirada da peça pelo robô.


#### Restrições Físicas
- A máquina **não pode iniciar um novo ciclo** enquanto a peça processada não for retirada.
- A peça **não pode ser retirada** antes da finalização do processamento.
- O estado "Peça Pronta" bloqueia a máquina até a ação do robô.


---


### 2.2 Robô Industrial (Robô)


O robô é o **agente central de transporte** da célula e atua como elo entre as máquinas de processamento e o buffer de saída.


#### Funcionamento
- O robô inicia em uma posição neutra, sem carga.
- Ele pode deslocar-se até a M1 ou M2 para coletar uma peça processada.
- Após a coleta, transporta a peça até o buffer de saída para deposição.
- Depois de depositar a peça, retorna ao estado livre.


#### Restrições Físicas
- **Capacidade unitária**: o robô só pode transportar uma peça por vez.
- **Sequenciamento**:
- Não pode coletar uma peça se já estiver carregando outra.
- Não pode depositar uma peça se não estiver carregando.
- **Sincronia**:
- A ação de coleta só pode ocorrer se a máquina estiver no estado "Peça Pronta".


---


### 2.3 Buffer de Saída (Esteira)


O buffer de saída é uma esteira ou zona de armazenamento temporário onde as peças finalizadas são depositadas antes de seguirem para o próximo setor produtivo ou expedição.


#### Funcionamento
- O buffer aceita peças trazidas pelo robô.
- As peças permanecem armazenadas até serem removidas por um agente externo (operador, empilhadeira ou processo automatizado).


#### Restrições Físicas
- **Capacidade limitada**: o buffer possui apenas **2 posições**, valor escolhido para evidenciar situações de bloqueio e congestionamento.
- **Overflow**: não é permitido depositar peças quando o buffer está cheio, pois isso representa falha operacional e risco de danos.


---


## 3. Fluxo Operacional e Interações


O ciclo nominal de operação do sistema segue a lógica abaixo:


1. Um sinal de início é enviado para a M1 e M2.
2. As máquinas processam as peças e, ao final, sinalizam o término do processamento (evento não controlável).
3. O robô, estando livre, desloca-se até uma máquina que possua uma peça pronta.
4. O robô coleta a peça, liberando imediatamente a máquina para um novo ciclo.
5. A peça é transportada até o buffer de saída.
6. Se houver espaço disponível, o robô deposita a peça e retorna ao estado livre.
7. Um evento externo remove peças do buffer, liberando espaço para novas deposições.


---


## 4. Problemas Potenciais do Sistema


Na ausência de um **supervisor (controlador lógico)** adequado, o sistema pode apresentar falhas críticas, como:


### 4.1 Colisão de Recursos
- O robô tentar coletar peças simultaneamente da M1 e da M2.


### 4.2 Violação de Capacidade
- Tentativa de depositar peças em um buffer já cheio, causando **overflow** e possível dano físico ao sistema.


### 4.3 Bloqueio (Deadlock)
**Cenário típico**:
- O buffer está cheio.
- O robô está segurando uma peça e aguardando espaço no buffer.
- As máquinas M1 e M2 terminaram o processamento e aguardam o robô.


**Consequência**:
- Nenhum componente pode avançar, e o sistema entra em **travamento completo**, interrompendo a produção.


### 4.4 Operações Inválidas
- Envio de comandos de coleta para máquinas que ainda estão processando.
- Ações de depósito sem carga no robô.


---


## 5. Projeto 


