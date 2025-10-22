# Dynamic Analysis - Debugging (TryHackMe)
###### Solved by @vin1sss

> This is a Room about Reverse Engineering

## Sala: Dynamic Analysis - Debugging (Análise dinâmica de malware)

[![image.png](https://i.postimg.cc/05SvvHnv/image.png)](https://postimg.cc/Sjm37dH1)

### Introdução

Esta sala da plataforma [TryHackMe](https://tryhackme.com/about) aprofunda a **[análise dinâmica](https://github.com/vin1sss/WriteUps/blob/main/attachments/An%C3%A1lise_Din%C3%A2mica.md) avançada** para contornar [evasões comuns usadas por malware](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md) (detecção de VM, *timing/sleep*, checagem de ferramentas, ausência de atividade do usuário). O foco é usar **x32dbg/x64dbg** para **controlar o fluxo de execução**, **manipular registradores/flags** e **patchar** o binário, permitindo observar o comportamento real.

* Pré-requisitos recomendados: *Basic Static Analysis*, *Basic Dynamic Analysis* e *Advanced Static Analysis*.
* Objetivo: entender e burlar um **TLS Callback** que tenta detectar análise, desviando do caminho de evasão e, por fim, **patchando** o binário para um bypass persistente.

***

## Tarefa 1 - Introdução

Esta tarefa apresenta o **problema central** da sala: malwares usam [**técnicas de evasão**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md) para burlar a [análise dinâmica](https://github.com/vin1sss/WriteUps/blob/main/attachments/An%C3%A1lise_Din%C3%A2mica.md) básica. Para lidar com isso, o analista passa a **controlar a execução** com depuradores, podendo **manipular registradores/flags em runtime** e **patchar** o binário para forçar o caminho que revela o comportamento real.

**Objetivos de aprendizagem:**

* Reconhecer **técnicas de evasão** contra análise dinâmica básica.
* Introduzir o uso de **depuradores** para controlar o fluxo de execução.
* **Manipular fluxo em tempo de execução** (registradores/flags, parâmetros).
* **Patchar** o binário para atravessar a evasão e expor o conteúdo malicioso.

**Pré-requisitos recomendados:**

* Análise Estática Básica
* Análise Dinâmica Básica
* Análise Estática Avançada

***

## Tarefa 2 - A necessidade de análise dinâmica avançada

Esta tarefa mostra por que a análise dinâmica **básica** já não basta: amostras modernas aplicam **evasões** para parecerem benignas em laboratório. A solução é ganhar **mais controle de execução** (nas próximas tarefas), indo além da simples observação.

### Evasão da análise estática (sem executar)

* **Alteração de hash**: pequenas mutações quebram detecção por hash.
* **Assinaturas AV**: variação de padrões para driblar *signatures*.
* **Ofuscação de strings**: decodificadas só em runtime (URLs, C2, chaves).
* **Imports dinâmicos**: `LoadLibrary*/GetProcAddress` ocultam APIs no PE.
* **Packing/obfuscação**: código “embrulhado” que só aparece **em memória** ao executar.

### Evasão da análise dinâmica **básica** (executando, mas sem controle fino)

* **Detecção de VM/sandbox**: chaves/artefatos de virtualização, 1 CPU, pouca RAM.
* **Ataques de tempo**: *sleep* longo e verificação de manipulação de relógio.
* **Traços de uso humano**: ausência de mouse/teclado, histórico vazio, pouco *uptime*.
* **Detecção de ferramentas**: enumeração de processos/janelas (ProcMon, ProcExp, Olly, etc.) para desviar fluxo.

> Ideia central: o malware **condiciona** o comportamento ao ambiente; sem controle de execução, o analista vê um **caminho benigno**.

### Perguntas da tarefa

* Às vezes, o malware checa o tempo antes/depois de certas instruções para saber se está sendo analisado. **Que técnica de análise é contornada por esse ataque?**

  **Resposta:** `basic dynamic analysis`

* **Qual técnica popular** é usada para esconder o código da análise estática e **desembrulhar em runtime**?
  
  **Resposta:** `Packing`

***

## Tarefa 3 - Introdução à depuração

Apresenta a [**depuração (debugging)**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Debugging.md) como ferramenta para ganhar **controle fino da execução**: pausar, inspecionar e **alterar estado** (registradores, memória, variáveis) a cada instrução. Isso permite **contornar obstáculos** e observar o comportamento real do programa.

**Tipos de depuradores (resumo):**

* **Nível de origem (source-level):** trabalha no **código-fonte**, mostra variáveis locais e contexto de alto nível.
* **Nível de assembly (assembly-level):** atua sobre **binários compilados**, expõe **registradores da CPU, memória, stack** e permite *step-in/over/out*. É o padrão para engenharia reversa.
* **Nível de kernel (kernel-level):** depura **no kernel**; requer geralmente **duas máquinas** (host alvo + host depurador). Interromper o kernel pausa **todo o sistema**.

**Ideia central:**
Com um depurador, o analista deixa de apenas observar para **intervir** no fluxo, ajustando flags/registradores e **forçando caminhos** que, de outra forma, não seriam percorridos.

### Perguntas da tarefa

* Podemos recuperar o **código pré-compilação** de um binário para depuração? (Y/N)

  **Resposta:** `N`

* **Qual tipo** de depurador é usado para depurar **binários compilados**?

  **Resposta:** `assembly-level debugger`

* Qual depurador opera no **nível mais baixo** entre os discutidos?

  **Resposta:** `Kernel-level debugger`

***

## Tarefa 4 - Familiarização com um depurador

Apresenta a **VM do laboratório (FLARE-like)** e o uso inicial do **x32dbg**: abrir o executável, entender a **CPU tab** (desmontagem, registradores, *stack/dump*) e localizar as **abas auxiliares** (Breakpoints, Memory Map, Call stack, Threads, Handles). A execução **pausada** em um *System breakpoint*, o que permite inspecionar o estado antes de prosseguir.

**Interface e navegação (resumo):**

* **Abertura/ambiente:** inicie a máquina da sala (ou conecte-se: **usuario:** `administrator` • **senha:** `Passw0rd!`). Execute **x32dbg** (32-bit) via *Desktop → Tools → debuggers*. Em **File → Open**, carregue o binário de exemplo `crackme-arebel1.exe`.

  [![image.png](https://i.postimg.cc/26MNgTtv/image.png)](https://postimg.cc/N96nH8Bf)

* **CPU tab (principal):**

  * **Disassembly** (centro) com o **Instruction Pointer (EIP/RIP)** na próxima instrução.

  [![image.png](https://i.postimg.cc/VkghtDmp/image.png)](https://postimg.cc/0z6ZRG60)

  * **Registers** (direita) com registradores e flags.

  [![image.png](https://i.postimg.cc/prvQqMBG/image.png)](https://postimg.cc/S2Zzx53C)

  * **Dump** (esq.) e **Stack** (dir.) no rodapé, além do **timer** da sessão.

  [![image.png](https://i.postimg.cc/9Qs7LjmD/image.png)](https://postimg.cc/jDQ2jmJT)
* **Breakpoints:** status/gestão dos *breakpoints*; ative clicando o ponto à esquerda da instrução na **CPU tab**.

  [![image.png](https://i.postimg.cc/0yK6MHY1/image.png)](https://postimg.cc/0zx5hnXV)

  [![image.png](https://i.postimg.cc/j2CqVLvx/image.png)](https://postimg.cc/fSnQmW6p)

* **Memory Map:** visão dos mapeamentos de memória do processo.

  [![image.png](https://i.postimg.cc/GhGtbsd3/image.png)](https://postimg.cc/WhpTnzdx)

* **Call stack:** cadeia de chamadas atual (retornos/frames).

  [![image.png](https://i.postimg.cc/c4RxpyBY/image.png)](https://postimg.cc/xcdVmZ7T)

* **Threads:** *threads* ativas do processo.

  [![image.png](https://i.postimg.cc/Gm3C6B2n/image.png)](https://postimg.cc/8jY96zMZ)

* **Handles:** recursos abertos (arquivos, processos, *pipes*, chaves etc.).

  [![image.png](https://i.postimg.cc/Gm4MBrgv/image.png)](https://postimg.cc/DSKQHVTz)

**Ideia central:**
Esta tarefa estabelece o **conhecimento da interface** do xdbg: saber **onde olhar** (CPU/regs/stack/dump) e **como pausar/retomar** (breakpoints) é pré-requisito para, nas próximas etapas, **controlar o fluxo**, *stepar* instruções críticas e **forçar caminhos** durante a análise.

### Perguntas da tarefa

* Em qual guia a visualização de desmontagem é mostrada no x32dbg?

  **Resposta:** `CPU tab`

* Se um processo abrir um arquivo ou um processo, onde podemos ver informações sobre esse recurso aberto?

  **Resposta:** `Handles tab`

***

