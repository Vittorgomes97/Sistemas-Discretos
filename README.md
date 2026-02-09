
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

Nesta seção são apresentados os modelos de estados que representam o comportamento funcional de cada componente da célula de manufatura automatizada. Os diagramas ilustram a lógica de operação, os eventos permitidos e as restrições físicas descritas anteriormente, servindo como base para a implementação do controle supervisor.

### 6.1 Modelo da Máquina 1 (M1)

O diagrama da Máquina 1 representa seu ciclo completo de operação. A máquina inicia no estado **idle**, aguardando o comando `start_m1` para iniciar o processamento. Ao receber esse comando, a M1 transita para o estado **processing**, onde executa o processamento da peça por um tempo indeterminado. Ao final, ocorre o evento `finish_m1`, levando a máquina ao estado **done**, que indica que a peça está pronta para ser retirada. A transição `get_m1` só pode ocorrer quando o robô coleta a peça, liberando a máquina para retornar ao estado **idle** e possibilitar um novo ciclo de produção. Esse modelo garante que a máquina não processe uma nova peça enquanto a anterior não for removida.

<div align="center">
  <img src="https://github.com/user-attachments/assets/060ec413-744c-4c02-a377-a1724f3bb42f" width="300px" />
</div>

### 6.2 Modelo da Máquina 2 (M2)

O modelo da Máquina 2 segue a mesma lógica funcional da Máquina 1, refletindo que ambas são idênticas em comportamento, porém independentes. A M2 inicia em **idle**, passa para **processing** após o comando `start_m2` e, ao concluir o processamento (`finish_m2`), entra no estado **done**. A peça só pode ser retirada pelo robô através do evento `get_m2`, retornando a máquina ao estado **idle**. Essa modelagem assegura o respeito às restrições físicas e evita retiradas prematuras ou reinício indevido do processamento.

<div align="center">
  <img src="https://github.com/user-attachments/assets/903b462c-e0f7-4866-8de5-fff2693e27b3" width="300px" />
</div>

### 6.3 Modelo do Buffer de Saída

O diagrama do buffer representa sua capacidade limitada de armazenamento. O estado **B0** indica buffer vazio, enquanto **B1** e **B3** representam níveis crescentes de ocupação. A transição `put_buffer` ocorre quando o robô deposita uma peça, aumentando a ocupação do buffer, desde que exista espaço disponível. A transição `remove_buffer` representa a retirada de uma peça por um agente externo, liberando espaço. Esse modelo impede o transbordamento (overflow), garantindo que o robô não deposite peças quando o buffer estiver cheio.

<div align="center">
  <img src="https://github.com/user-attachments/assets/76ea89ba-eeee-453b-aa10-23eb3edfe922" width="300px" />
</div>

### 6.4 Modelo do Robô Industrial

O robô é modelado como o elemento central de coordenação do sistema. Ele inicia no estado **free**, sem carga, podendo deslocar-se até a Máquina 1 (`go_m1`) ou Máquina 2 (`go_m2`). Ao chegar à máquina correspondente (**At_M1** ou **At_M2**), o robô pode executar `get_m1` ou `get_m2`, desde que a máquina esteja no estado **done**, passando então para o estado **Carrying**, no qual transporta uma peça. A única ação permitida nesse estado é `put_buffer`, que deposita a peça no buffer e retorna o robô ao estado **free**. Esse modelo garante a capacidade unitária do robô e evita conflitos de transporte ou operações inválidas.

<div align="center">
  <img src="https://github.com/user-attachments/assets/418b422e-9595-4214-82c1-b733c25cf0d7" width="300px" />
</div>

## 7. Vídeo Demonstrativo do Funcionamento

Com o objetivo de complementar a descrição teórica e a modelagem apresentada ao longo do projeto, foi desenvolvido um **vídeo demonstrativo** que ilustra o funcionamento da célula de manufatura automatizada em operação.

🔗 **Acesse o vídeo demonstrativo:**  
[https://youtu.be/i_YFYhjIBA8?si=qRuYvhwU87N6bokD]


## 8. Conclusão

Este projeto apresentou o desenvolvimento e a modelagem de uma célula de manufatura automatizada composta por duas máquinas de processamento independentes, um robô industrial compartilhado e um buffer de saída com capacidade limitada. Ao longo do trabalho, foram detalhados o funcionamento físico e lógico de cada componente, bem como suas interações, restrições operacionais e possíveis falhas decorrentes da ausência de um controle adequado.

A modelagem por meio de diagramas de estados permitiu representar de forma clara e estruturada o comportamento de cada elemento do sistema, evidenciando eventos controláveis, não controláveis e condições de sincronização. Essa abordagem possibilitou a identificação de situações críticas, como transbordamento do buffer, operações inválidas e cenários de deadlock, reforçando a importância da implementação de um sistema supervisor.

Os resultados obtidos demonstram que a coordenação eficiente dos recursos compartilhados, em especial do robô industrial e do buffer de saída, é essencial para garantir a continuidade da produção, a segurança operacional e a confiabilidade do sistema. A limitação intencional da capacidade do buffer mostrou-se eficaz para evidenciar problemas de bloqueio e validar a necessidade de políticas de controle bem definidas.

Por fim, o projeto estabelece uma base sólida para trabalhos futuros, como a implementação de um supervisor lógico formal, a aplicação de Redes de Petri ou autômatos supervisionados, e a validação do modelo em ambientes de simulação ou sistemas reais de automação industrial. Dessa forma, o estudo contribui para a compreensão prática e teórica de sistemas a eventos discretos aplicados à manufatura automatizada.
