# Dynamic Analysis - Debugging (TryHackMe)
###### Solved by @vin1sss 

> This is a Room about Reverse Engineering

## Sala: Dynamic Analysis - Debugging (Análise dinâmica de malware)

[![image.png](https://i.postimg.cc/05SvvHnv/image.png)](https://postimg.cc/Sjm37dH1)

### Introdução

[Esta sala](https://tryhackme.com/room/advanceddynamicanalysis) da plataforma [TryHackMe](https://tryhackme.com/about) aprofunda a **[análise dinâmica](https://github.com/vin1sss/WriteUps/blob/main/attachments/An%C3%A1lise_Din%C3%A2mica.md) avançada** para contornar [evasões comuns usadas por malware](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md) (detecção de VM, *timing/sleep*, checagem de ferramentas, ausência de atividade do usuário). O foco é usar o **[x32dbg/x64dbg](https://x64dbg.com/)** para **controlar o fluxo de execução**, **manipular registradores/flags** e **patchar** o binário, permitindo observar o comportamento real.

* Pré-requisitos recomendados: *Basic Static Analysis*, *Basic Dynamic Analysis* e *Advanced Static Analysis*.
* Objetivo: entender e burlar um **TLS Callback** que tenta detectar análise, desviando do caminho de evasão e, por fim, **patchando** o binário para um bypass persistente.

***

## Tarefa 1 - Introdução

Esta tarefa apresenta o **problema central** da sala: malwares usam [**técnicas de evasão**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md) para burlar a [análise dinâmica](https://github.com/vin1sss/WriteUps/blob/main/attachments/An%C3%A1lise_Din%C3%A2mica.md) básica. Para lidar com isso, o analista passa a **controlar a execução** com depuradores, podendo **manipular registradores/flags em runtime** e **patchar** o binário para forçar o caminho que revela o comportamento real.

**Objetivos de aprendizagem:**

* Reconhecer [**técnicas de evasão**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md) contra análise dinâmica básica.
* Introduzir o uso de [**depuradores**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Debugging.md) para controlar o fluxo de execução.
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

## Tarefa 5 — Depuração na prática

Mostra, na prática, como **executar passo a passo** um binário no **x32dbg/x64dbg**: abrir o `crackme-arebel.exe`, **deixar rodar uma vez** até cair no **TLS Callback**, navegar com **Step Into** até um **salto condicional (Jcc)**, **ler flags** (especialmente **ZF**) e reconhecer que o caminho atual leva a um **fluxo de evasão**.

**Controles e pontos-chave (resumo):**

* **File → Open**: carregue `crackme-arebel.exe`; o depurador **anexa** e pausa antes do início (uma janela de console vazia pode aparecer — é normal).

  [![image.png](https://i.postimg.cc/cJwMGJ6s/image.png)](https://postimg.cc/NyGr1B1V)

* **Run (→) apenas uma vez**: o status passa a *Running* e logo *Paused* em **`INT3 breakpoint "TLS Callback 1"`** (auto-break de TLS).

  [![image.png](https://i.postimg.cc/DzLDpnTY/image.png)](https://postimg.cc/676YysTd)

  * Configure/valide em **Options → Preferences**: **TLS Callbacks** e **System TLS Callbacks** marcados.

    [![image.png](https://i.postimg.cc/GmYqdK5w/image.png)](https://postimg.cc/nXFv14x0)

* **Step Into** dentro do TLS Callback: observe **EIP** avançar, **Registers/Stack** mudarem a cada instrução.

  [![image.png](https://i.postimg.cc/XqfzN0LX/image.png)](https://postimg.cc/0KNfZFLq)

* No **Jcc**, a barra informa **“jump not taken”** e, em **Registers**, **ZF = 1**.

  [![image.png](https://i.postimg.cc/htjp6mkQ/image.png)](https://postimg.cc/4mD6hYNX)

  
* Se travar por engano (Run a mais), **Stop/Restart** o processo e repita.

**Ideia central:**
Aprender a **interpretar quebras em TLS Callback**, **inspecionar flags** para explicar decisões de salto e **comparar ramos** (endereços-alvo) é pré-requisito para, na sequência, **forçar o caminho correto em runtime** e, depois, **patchar** o binário para um bypass persistente.

### Perguntas da tarefa

* No TLS Callback há um **salto condicional**. Esse salto é tomado? (Y/N)

  **Resposta:** `N`

* **Qual é o valor da Zero Flag (ZF)** nesse salto condicional?

  **Resposta:** `1`

* **Qual API** é usada para **enumerar processos em execução**?

  **Resposta:** `CreateToolhelp32Snapshot`

* De qual **DLL do Windows** a API **SuspendThread** é chamada?

  **Resposta:** `kernel32.dll`

***

## Tarefa 6 — Ignorando o caminho de execução indesejado

Dá continuidade direta à prática anterior: **retornar ao TLS Callback**, alcançar o **salto condicional (Jcc)** e **forçar o desvio correto** primeiro em *runtime* (alterando **ZF**), depois de forma **persistente** via **patching**. O objetivo é evitar o **caminho de evasão** (que congela/suspende) e seguir o fluxo legítimo de análise.

**Passos práticos (resumo):**

* **Reinicie a execução** até o **TLS Callback** e avance (**Step Into**) até o **Jne**.

  [![image.png](https://i.postimg.cc/05XTQ89z/image.png)](https://postimg.cc/6TRjbx2K)

* O depurador indica **“jump not taken”** e, nos **Registers**, **ZF = 1**.

  [![image.png](https://i.postimg.cc/RhWbfHvg/image.png)](https://postimg.cc/9wjpHzR9)

  [![image.png](https://i.postimg.cc/tCWGmKHm/image.png)](https://postimg.cc/hfDYX3v9)

* **Altere o ZF para 0** (duplo clique em **ZF** no painel de registradores) → o **salto passa a ser tomado** (*jump taken*), levando ao endereço **`0127116E`**.

  [![image.png](https://i.postimg.cc/FFkM7MzT/image.png)](https://postimg.cc/hhgyY39m)

  [![image.png](https://i.postimg.cc/NFvpyPCV/image.png)](https://postimg.cc/sQKpqKr9)

* Prossiga alguns passos para **validar** que o fluxo seguiu o **branch** desejado (sem entrar no bloco de evasão).

**Por que só mudar ZF não basta:**
A alteração da flag é **temporária** (vale para **esta** execução). Ao reabrir o binário, o fluxo **voltará** ao caminho de evasão se as mesmas condições forem satisfeitas. Para “desarmar” de vez, é preciso fazer o **patching**.

**Patching do binário (opções):**

* **Trocar `jne` → `je`** (Edit/Assemble): inverte a condição; o salto será tomado quando antes não era, sem depender de mexer no ZF.

  [![image.png](https://i.postimg.cc/8PzNsK8X/image.png)](https://postimg.cc/WtCQH710)

* **Converter o condicional em `jmp` (incondicional)**: garante que **sempre** salte; é a opção **mais assertiva**.

  [![image.png](https://i.postimg.cc/qMzS99Vc/image.png)](https://postimg.cc/QHZf7YMV)

* **NOP na instrução sensível** (ex.: `call` em **`01271169`**): *right-click* → **Binary → Fill with NOPs**. Substitui a instrução por **NOPs** de mesmo tamanho (ex.: 5 bytes → 5 NOPs).

  [![image.png](https://i.postimg.cc/XJqkr6g8/image.png)](https://postimg.cc/Q9LTzPw9)

**Como salvar o patch:**

* **File → Patch File** → escolha onde gravar a cópia patchada (ex.: `crackme-arebel2.exe`).

  [![image.png](https://i.postimg.cc/ZYcv1MFT/image.png)](https://postimg.cc/mhPr9V5q)

* Reabra o **binário patchado** para confirmar que o **caminho de evasão** ficou inacessível sem intervenções manuais.

  [![image.png](https://i.postimg.cc/VL2fTqTL/image.png)](https://postimg.cc/N57v2HBW)

**Ideia central:**
Primeiro, **forçamos a branch em runtime** (ZF=0) para comprovar a hipótese. Em seguida, **fixamos o comportamento** com **patching**, eliminando a necessidade de manipular flags a cada execução e estabilizando a análise dinâmica.

### Perguntas da tarefa

* **Como se chama quando o código assembly de um binário é alterado permanentemente para obter o caminho de execução desejado?**

  **Resposta:** `Patching`

***

## Tarefa 7 — Conclusão

Fecha a trilha prática de **análise dinâmica avançada com depuração**: partimos de evasões que enganam a dinâmica básica, evoluímos para **controle fino da execução** (x32dbg/x64dbg), **manipulação de flags/registradores** em runtime e **patching** para estabilizar o fluxo e expor o comportamento real.

**O que foi visto nesta sala:**

* **Evasões vs. análise estática/dinâmica básica:** como alterações de hash, *packing*, imports dinâmicos, checagens de VM/tempo/usuário e detecção de ferramentas **condicionam** o comportamento observado.
* **Depuração como alavanca de controle:** pausar onde importa (ex.: **TLS Callbacks**), **step-in/over**, ler **flags (ZF)** e **registradores**, inspecionar *stack/dump*, *threads*, *handles* e **call stack**.
* **Intervenção em runtime:** alterar **estado** (p.ex., **ZF=0**) para **forçar/evitar branches** e confirmar hipóteses sobre caminhos de evasão.
* **Patching (defang persistente):** transformar `jne→je`, trocar por `jmp` incondicional ou **NOPar** instruções sensíveis e salvar via **Patch File**, eliminando a dependência de intervenção manual a cada execução.

**Ideia central:**
A análise dinâmica só cumpre seu papel quando o analista **conduz o programa** pelo caminho certo. Depurar e patchar não são apenas “truques”: são métodos para **restaurar observabilidade**, reduzir falsos negativos e produzir **evidências confiáveis**.

### Perguntas da tarefa

* *Acesse nossos canais sociais para uma discussão mais aprofundada.*
  **Nenhuma resposta necessária (check de progresso).**

***

**Fim!** 

[![image.png](https://i.postimg.cc/Xv37qW9F/image.png)](https://postimg.cc/VJGcZ3GL)

<br>

<p align="center">
  <img src="https://github.com/vin1sss/WriteUps/blob/main/attachments/ninjaduck.png" width="50"/>
</p>

##
