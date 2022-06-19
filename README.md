### Tunning PFSENSE
#### Melhorar o desempenho do pfSense
___
### Problemas comuns

Esses problemas são causados por hardwares não homologados pela NetGate, isso é normal, já que o sistema é produzido para tirar o máximo de desempenho de um hardware homologado.

Para o melhoramento do sistema, os seguintes podem ser alterados para um melhor desempenho:

____

* ```MBUF```:
    * Responsavel pelo buffes de memória de rede;
    * Sobre o ```MBUF Cluster``` -> Ele vai tentar usar o máximo de memória possível como cache (Somente memória livre), caso ele bata um limite estabelecido, ele limpa demais cache's para ter mais memória para ele;
    * Erros por falta de memória ```MBUF``` são entre ```Kernel Panic``` a ```interfaces inutilizadas```;
    * O consumo do ```MBUF``` é aumentado quando a ações de ```Limiters```;
    * Para aumentar esse valor, edite o arquivo: ```/boot/loader.conf.local```;
    * Dentro dele adicione a váriavel ou edite a já existe ```kern.ipc.nmbclusters```;
    * Caso editar esse valor, fique ciente que talvez seja preciso editar a váriavel ```kern.ipc.nmbjumbop```;

<br>

____

* ```FILAS NIC```:
    * Em sistemas com mais nucleos, se é usado uma fila NIC por nucleo para os processos, o resultado é o paralelismo de tarefas, porém, tal ação pode leva a instabilidade do sistema, para editar esse valor vá para o arquivo: ```/boot/loader.conf.local``` e edite a váriavel: ```hw.*.num_queues="1"```;
    * O nome da váriavel normalmente é diferente para cada sistema, dessa forma, pegue os valores a serem trabalhados via o comando de: ```sysctl -a | grep hw.*```;

<br>

____

* ```Problemas de placa MSI com MSIX```:
    * Desative o MSIX para estabilidade sobre o hardware:
        * Edite o arquivo: ```/boot/loader.conf.local``` e adicione as váriaveis;
            * ```hw.pci.enable_msix="0"```;
            * ```hw.pci.enable_msi="0"```;

<br>

____

* ```PPPoE com NICs de várias filas```:
    * NIC de várias filas funcionam em vários protocolos, como exemplo IPv4/6 TCP/UDP, porém falham em PPPoE;
    * O NIC em várias filas gera um fila HASH ID para corresponder a cada fila;
    * Para o uso do PPPoE é recomendo derrubar o processo da várias filas para um valor correspondente a quantidade de nucleos reais do sistema dentr do ```loader.config.local```;
    * Outras formas de obter desempenho é setando a variavel: ```net.isr.dispatch=deferred``` com o valor de ```deferred``` (Ele causava erros em sistema de 32Bits, porém com o desuso de tais sistemas e sem problemas recentes reportados na versões mais novas, o problema não vem mais ocorrido);
    * Ajustar os valores de ```net.isr.maxthreads``` e ```net.isr.numthreads``` pode gerar ganhos de desempenho adicionais.

<br>

____

* ```TSO/LRO```:
    * ```Hardware TCP Segmentation Offload (TSO)```;
    * ```Hardware Large Receive Offload (LRO)```;
    * Vem por default desativado, já que quase todos os drivers/Hardwares tem problemas com essa config que causa lentidam em taxa de transferencia;
    * Pode ser necessário desativar ela em: ```/boot/loader.conf.local``` com essa linha ```net.inet.tcp.tso="0"```;

<br>

____
* ```Fila de entrada IP (intr_queue)```:
    * Aqui tem que encontrar um ponto de equilibro na tentativa e erro.
    * Confira se o valor de ```net.inet.ip.intr_queue_drops``` for maior que ```0```, se sim, configure o valor de ```net.inet.ip.intr_queue_maxlen``` com o dobro do valor atual, ou até um ponto confortavel ao sistema;

<br>

____
* ```Problemas com placas de rede```:
    * Placas de rede Broadcom:

```
kern.ipc.nmbclusters="1000000"
hw.bce.tso_enable="0"
hw.pci.enable_msix="0"
```

<br>

* ```Perda de pacotes com muitos (pequenos) pacotes UDP```:
    * Perda de muitas quantidades de pacotes com a placa de rede BCE e BGE

```
net.isr.direct_force="1"
net.isr.direct="1"
```

<br>

* ```Cartões Chelsio cxgbe```:
    * Desabilitar recursos não relacionados a função da placa de rede:

```
hw.cxgbe.toecaps_allowed="0"
hw.cxgbe.rdmacaps_allowed="0"
hw.cxgbe.iscsicaps_allowed="0"
hw.cxgbe.fcoecaps_allowed="0"
```

<br>

* ```Placas IGB e EM```:
    * Esgostam rapidamente MBUFS, recomendado um aumento da memória para interfaces com essas identificações:

```
kern.ipc.nmbclusters="1000000"
```

<br>

* ```Placas Intel ix```:

```
kern.ipc.nmbclusters="1000000"
kern.ipc.nmbjumbop="524288"
hw.intr_storm_threshold="10000"
```
____

<br>

* ```Controle de fluxo```:
    * Utilizado para controlar os pacotes das interfaces, em alguns casos, dependente do hardware, é necessário desativar esse recurso (Ex):

<br>

* ```cxgbe```:
```
hw.cxgbe.pause_settings="0"
```

<br>

* ```ixgbe```:
```
hw.ix.flow_control="0"
```

* ```ixgbe```:
```
dev.ix.#.fc="0"
```

* ```igc```:
```
dev.igc.#.fc="0"
```

* ```igb```:
```
dev.igb.#.fc="0"
```

* ```em```:
```
dev.em.#.fc="0"
```

Para ix e outros, o valor de controle de fluxo pode ser ajustado:

* 0 -> Sem controle de fluxo
* 1 -> Pausa de recebimento
* 2 -> Transmitir Pausa
* 3 -> Controle de fluxo total, padrão

____