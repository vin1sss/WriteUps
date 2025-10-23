# Anti-Reverse Engineering (TryHackMe)
###### Solved by @vin1sss 

> This is a Room about Reverse Engineering

## Sala: Anti-Reverse Engineering (Engenharia anti-reversa)

[![image.png](https://i.postimg.cc/KYTbP8yP/image.png)](https://postimg.cc/CZw9YY55)

### Introdução

[Esta sala](https://tryhackme.com/room/antireverseengineering) da plataforma TryHackMe aprofunda **técnicas de [anti–engenharia reversa](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md)** para contornar barreiras como **anti-depuração** (ex.: `IsDebuggerPresent`, uso de `SuspendThread`), **detecção de VM** (ex.: WMI/`Win32_TemperatureProbe`) e **ofuscação via packers** (ex.: UPX/Themida). O foco é usar o **[x32dbg/x64dbg](https://x64dbg.com/)** e ferramentas de apoio (**DetectItEasy, PEStudio, Scylla**) para **controlar o fluxo de execução**, **editar memória/registradores (incl. EIP/RIP)** e **patchar** instruções (NOPs/saltos), revelando o comportamento real.

**Pré-requisitos recomendados:** Basic Dynamic Analysis (e avançada), noções de Assembly (registradores/flags) e conceitos básicos de C (condições/fluxo).

**Objetivo:** identificar e **burlar** anti-depuração, **detecção de VM** e **packers** por meio de **patching** (NOPs/saltos), **edição de memória/fluxo** e **(des)empacotamento** com DIE/PEStudio/Scylla, obtendo uma amostra analisável sem as barreiras de anti-RE.

***

## Tarefa 1 - Introdução

Esta tarefa enquadra a **“corrida armamentista”** entre analistas e autores de malware e apresenta o foco da sala: **[anti–engenharia reversa](https://github.com/vin1sss/WriteUps/blob/main/attachments/Evas%C3%B5es_usadas_por_malwares.md)**. Você verá como amostras adotam **detecção de VM**, **ofuscação via *packers*** e **anti-depuração** para dificultar a análise — e como **contornar** esses mecanismos usando **x32dbg/x64dbg** e ferramentas de apoio (DetectItEasy, PEStudio, Scylla), por meio de **patching**, **edição de memória/fluxo** e **(des)empacotamento**.

**Objetivos de aprendizagem:**

* Entender **por que** autores usam técnicas de **anti-RE**.
* Reconhecer **detecção de VM**, **packers/ofuscação** e **anti-depuração**.
* Aprender **contornos práticos**: patching (NOPs/saltos), edição de memória/registradores e *(des)empacotamento*.
* Ler **trechos de código** para identificar e compreender implementações de anti-RE.

**Pré-requisitos recomendados:**

* Análise Dinâmica (Básica e Avançada): usar depurador, ler **assembly**, aplicar **patches**.
* **Assembly** básico (registradores/flags).
* **C** básico (condições, fluxo do programa).

**Pergunta da tarefa:** *Vamos!* → **Nenhuma resposta necessária.**

***

## Tarefa 2 — Anti-depuração (visão geral)

Apresenta [**depuração**](https://github.com/vin1sss/WriteUps/blob/main/attachments/Debugging.md) como prática de inspecionar a execução (passo a passo, estado, chamadas) e o conjunto de técnicas que **tentam impedir** essa inspeção (**anti-debugging**). Você verá *checks* de presença de depurador, manipulação de estado/estruturas de debug e **código auto-modificante** — base para os bypasses práticos nas próximas tarefas.

**Depuradores comuns (referência):**

* **[x32dbg / x64dbg](https://x64dbg.com/)**
* **[Ollydbg](https://www.ollydbg.de/)**
* **[IDA Pro](https://hex-rays.com/ida-pro)**
* **[Ghidra](https://www.plugged.ninja/2024/06/ghidra-estrutura-de-engenharia-reversa-de-software-de-codigo-aberto/)**

**Técnicas de anti-depuração (resumo):**

* **Presença de depurador:** varredura de processos/janelas/artefatos e APIs do SO (ex.: **`IsDebuggerPresent`**), inclusive detecção de *hardware breakpoints*.
* **Adulteração do estado de debug:** corromper/alterar estruturas e **registros de depuração** (ou *logs*) para quebrar o controle de execução do depurador.
* **Código auto-modificante:** o binário **muda a si mesmo em runtime**, embaralhando o fluxo e dificultando o *stepping* previsível.

**Ideia central:**
Anti-depuração **quebra a previsibilidade** do analista: se detectar instrumentos, **desvia**, **dorme**, **suspende threads** ou **altera o próprio código** — tudo para degradar a observação. Na próxima tarefa, veremos o uso de **`SuspendThread`** para travar a depuração.

### Pergunta da tarefa

* **Qual é o nome da função de API do Windows** usada em uma técnica antidepuração comum que detecta se um depurador está em execução?

  **Resposta:** `IsDebuggerPresent`

***

## Tarefa 3 — Anti-depuração usando **SuspendThread**

Demonstra, na prática, uma técnica de **anti-depuração** que congela o depurador: o binário identifica janelas de ferramentas de análise (ex.: “debugger”, “dbg”, “debug”) e chama a API **`SuspendThread`** para **suspender os *threads* do depurador**, tornando-o **inoperante**. Em seguida, você **contorna** isso via **patching** (NOPs) e aprende a **exportar/importar patches** no x32dbg.

**Como a técnica funciona (visão rápida):**

* Enumera *threads* e **janelas** do sistema (API **`EnumWindows`**).
* Se o título da janela contém *“debugger/dbg/debug”*, considera que há depurador.
* Chama **`SuspendThread`** nos *threads* do depurador → o x32dbg/x64dbg **congela**.
* O malware segue executando **sem ser inspecionado**.

**Passo a passo (congelando o depurador de propósito):**

1. Abra o **x32dbg** (Área de Trabalho).

    [![image.png](https://i.postimg.cc/CKZCqfBz/image.png)](https://postimg.cc/cKS8qCcS)

2. **File → Open** e selecione `C:\Malware\suspend-thread.exe`.

    [![image.png](https://i.postimg.cc/BvDL351J/image.png)](https://postimg.cc/1gm5cqHd)

    [![image.png](https://i.postimg.cc/1RMgy344/image.png)](https://postimg.cc/nMjVkZ4t)

3. Pressione **F9 duas vezes**.
4. Observe o **x64dbg/x32dbg travado**; confirme no **Task Manager** (se necessário, *End Task*).

    [![image.png](https://i.postimg.cc/FsMh4Mb4/image.png)](https://postimg.cc/mtyv366X)

**Patching para neutralizar `SuspendThread`:**

1. Reabra o **x32dbg** e carregue `suspend-thread.exe` (**F3** para abrir).
2. Pressione **F9** uma vez para alcançar o **EntryPoint**.

    [![image.png](https://i.postimg.cc/wM5nYr93/image.png)](https://postimg.cc/y3dLFvgz)

3. Clique com o direito → **Search for → Current module → Intermodular calls**.

    [![image.png](https://i.postimg.cc/mgkpmk1G/image.png)](https://postimg.cc/Yvcx9pxX)

4. No filtro, digite **`SuspendThread`** e **duplo clique** na entrada encontrada.

    [![image.png](https://i.postimg.cc/TY7C5x7H/image.png)](https://postimg.cc/rK4S7BLS)

    [![image.png](https://i.postimg.cc/VNy9Xkq6/image.png)](https://postimg.cc/G4QsCrSw)

    [![image.png](https://i.postimg.cc/W35JYNkh/image.png)](https://postimg.cc/FdJR7QG4)

5. Na linha da chamada, **right-click → Binary → Fill with NOPs** → **OK**.

    [![image.png](https://i.postimg.cc/4dy3dzYJ/image.png)](https://postimg.cc/nj8xS9K5)

    [![image.png](https://i.postimg.cc/667dbZYY/image.png)](https://postimg.cc/Th6LpK8D)

   * Os bytes do *call* serão definidos como **`90`** (NOP). Ao chegar aqui, **não faz nada** e segue adiante.

        [![image.png](https://i.postimg.cc/8zbWfmzp/image.png)](https://postimg.cc/mPth0M9J) 

6. Pressione **F9**: a execução **prossegue até o fim sem travar** o depurador (mesmo que o console imprima “Debugger found… Suspending!”, o *call* foi neutralizado).

    [![image.png](https://i.postimg.cc/y8fxzQNG/image.png)](https://postimg.cc/nXDp41BK)

**Persistindo seus patches (exportar/importar):**

1. Pare a depuração com **Alt+F2** (não rode **F9** ainda).
2. Menu **View → Patch file…** para abrir a janela de *patches*.

    [![image.png](https://i.postimg.cc/HkJDVB2t/image.png)](https://postimg.cc/fVhgP713)

3. Confira as entradas (vários endereços NOPados) e clique em **Export** → **Yes** → salve o arquivo.

    [![image.png](https://i.postimg.cc/7hPjDVh2/image.png)](https://postimg.cc/JDgp5jbr)

    [![image.png](https://i.postimg.cc/NfzZqRcp/image.png)](https://postimg.cc/4mpWp7n9)

    [![image.png](https://i.postimg.cc/76Km1yPt/image.png)](https://postimg.cc/hz7Lg6dV)

4. Feche/pare, recomece a depuração do binário.
5. **View → Patch file… → Import** e selecione o arquivo salvo.

    [![image.png](https://i.postimg.cc/fL15qfXw/image.png)](https://postimg.cc/S27WJ8D5)

6. Feche a janela, **F9**: os patches são **reaplicados automaticamente**.

**Ideia central:**
`SuspendThread` é um **kill-switch de observabilidade**. **NOPar** a chamada impede o congelamento do depurador e **restaura o controle** da sessão, permitindo avançar na engenharia reversa. Exportar/importar patches **economiza tempo** e garante **reprodutibilidade** entre sessões.

### Perguntas da tarefa

* **Qual é a função da API do Windows** que **enumera janelas** na tela para que o malware verifique o nome da janela?

  **Resposta:** `EnumWindows`

* **Qual é o valor hexadecimal** de uma instrução **NOP**?

  **Resposta:** `90`

* **Qual é a instrução** encontrada no local de memória **004011CB**?

  **Resposta:** `add esp, 8`

***

## Tarefa 4 — Detecção de VM (visão geral)

Mostra como amostras **detectam virtualização/sandbox** para **reduzir sinais** e confundir a análise (executar só um mínimo, autodestruir, causar dano ou nem rodar). A ideia é reconhecer os **sinais típicos de VM** e saber como **disfarçar** o ambiente quando necessário.

**Como a técnica funciona (visão rápida):**

* **Processos característicos:** busca por serviços de VM (ex.: **`vboxservice`** no VirtualBox; `vmtoolsd`/`vmtools` em VMware).
* **Softwares instalados (Registro):** varredura em
  `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Uninstall` por depuradores/forenses.
* **Impressão digital de rede:** checagem de **OUIs** típicos no MAC (ex.: `00-05-69`, `00-0c-29`, `00-1c-14`, **`00-50-56`**/VMware).
* **Recursos da máquina:** pouca **RAM/CPU** sugere laboratório.
* **Periféricos ausentes:** impressoras/câmeras raramente configuradas em VMs.
* **Domínio corporativo:** leitura de `LoggonServer`/`ComputerName` (AD).
* **Temporização:** latências/execuções que variam entre físico e VM.

**Mitigação (disfarce de VM em laboratório):**

* **Remover/alterar artefatos:** esconder entradas de Registro, **trocar MAC**, simular periféricos.
* **Automatizar:** usar scripts como *VMwareCloak* / *VBoxCloak* (cobertura parcial).
* **Plano B:** se ainda assim houver bloqueio, **forçar fluxo** via depuração/patching (como nas tarefas anteriores).

**Ideia central:**
Detecção de VM é **heurística** e combinatória. Reduzir artefatos ajuda, mas o analista deve estar pronto para **intervir no fluxo** quando a amostra insistir em permanecer “silenciosa”.

### Perguntas da tarefa

* **Qual é o nome do processo** identificável usado para checar se a máquina está rodando no **VirtualBox**?

  **Resposta:** `vboxservice`

* **O que é o OUI** atribuído automaticamente especificamente à **VMware**?

  **Resposta:** `00:50:56`

* Usando o **Gerenciador de Tarefas**, **qual processo** indica que a máquina desta sala é uma **VM do Amazon EC2**?

  **Resposta:** `amazon-ssm-agent.exe`

    [![image.png](https://i.postimg.cc/VkPpkmJX/image.png)](https://postimg.cc/zVp0dmSf)

***

## Tarefa 5 — Detecção de VM por **temperatura** (WMI/SMBIOS)

Demonstra, na prática, uma técnica de **detecção de VM** baseada em **WMI**: o binário consulta sensores térmicos via SMBIOS (classe **`Win32_TemperatureProbe`** / `MSAcpi_ThermalZoneTemperature`). Em ambientes virtualizados, a consulta costuma retornar **“Not Supported”** (ou nenhum objeto), e o programa **desvia** para um caminho “benigno”. Em seguida, você **contorna** isso no **x32dbg** **manipulando memória** (forçando `uReturn=1`) e **redirecionando o fluxo com o EIP**.

**Como a técnica funciona (visão rápida):**

* Consulta WMI por leitura térmica (**SMBIOS**).
* Em **VM**: resultado **inexistente** → `uReturn = 0` → caminho “VM detectada” (comportamento reduzido).
* Em **máquina física**: leitura válida → fluxo “normal” (ex.: imprime a temperatura).

**Passo a passo (reproduzindo e burlando no x32dbg):**

1. Abra o **x32dbg** → **File → Open** → `C:\Malware\vm-detection.exe`.

    [![image.png](https://i.postimg.cc/9Q168sfz/image.png)](https://postimg.cc/3k4fWSmH)

2. **F9** para iniciar (para no **EntryPoint**).

    [![image.png](https://i.postimg.cc/wjBfvM6M/image.png)](https://postimg.cc/Mvhb3WfJ)

3. **Ctrl+G** → digite **`004010E0`** → **OK**.

    [![image.png](https://i.postimg.cc/j2dNXW3H/image.png)](https://postimg.cc/sQqvf2rx)

4. **F2** para marcar *breakpoint* → **F9** para parar nesse endereço.

    [![image.png](https://i.postimg.cc/Y9tmrGHJ/image.png)](https://postimg.cc/cK5JTJ6m)

    [![image.png](https://i.postimg.cc/J02y5pkw/image.png)](https://postimg.cc/Lgj6HBDv)

5. **F8** até **`004010FD`** (comparação de **`[ebp-18]`** ≡ `uReturn` com zero).

    [![image.png](https://i.postimg.cc/85nk2jPT/image.png)](https://postimg.cc/VrncMsCh)

6. Clique com o direito na linha com **`[ebp-18]`** → **Follow in dump → [ebp-18]** (ex.: **`0019FF08`**).

    [![image.png](https://i.postimg.cc/mhW44Mx4/image.png)](https://postimg.cc/V0gpBJLZ)

    [![image.png](https://i.postimg.cc/dtcDwZjW/image.png)](https://postimg.cc/CdJw41tD)

7. No *Dump*, **duplo clique** no byte `00` mais à direita → **Modify Value** → coloque **`01`** → **OK** (força `uReturn=1`).

    [![image.png](https://i.postimg.cc/CxLhxGB4/image.png)](https://postimg.cc/F785PJxf)

8. Volte à aba **CPU** e **F8** algumas vezes: o código **não sai** mais e segue o caminho “não-VM”.

**Saltando direto com o registrador EIP (pular para a impressão):**

1. No painel **Registers**, clique com o direito em **EIP** → **Modify Value**.

    [![image.png](https://i.postimg.cc/HkjqNB89/image.png)](https://postimg.cc/cvykg7mv)

2. Defina **`00401134`** (trecho do `printf` de temperatura, bloco `00401130–00401139`).

    [![image.png](https://i.postimg.cc/hvpYydSk/image.png)](https://postimg.cc/BPPNttPp)

3. **F8** para confirmar o salto; **F9** para executar: observe a **saída de temperatura** (fluxo “não-VM”).

    [![image.png](https://i.postimg.cc/WbdSLKSJ/image.png)](https://postimg.cc/LnMjjxcH)

    [![image.png](https://i.postimg.cc/c4Ck161S/image.png)](https://postimg.cc/hfF1pD7y)

**Ideia central:**
Checagens térmicas via WMI/SMBIOS são **heurísticas arquiteturais**. Em engenharia reversa, você pode **forçar o contexto**: editar **valores em memória** (ex.: `uReturn`) ou **reposicionar o EIP** para observar o comportamento útil sem ser bloqueado pela detecção de VM.

### Perguntas da tarefa

* No trecho de código C, **qual é a consulta WQL completa** usada para obter a temperatura da classe **Win32_TemperatureProbe**?

  **Resposta:** `SELECT * FROM MSAcpi_ThermalZoneTemperature`

  [![image.png](https://i.postimg.cc/gc8vPZ6D/image.png)](https://postimg.cc/CdMZGd7B)

* **Qual registrador** contém o endereço de memória que informa ao depurador **qual instrução executar em seguida**?

  **Resposta:** `EIP`

* Antes de `uReturn` ser comparado a zero, **qual é a localização de memória apontada por `[ebp-4]`**?

  **Resposta:** `0019FF1C`

  [![image.png](https://i.postimg.cc/85tgF5FT/image.png)](https://postimg.cc/vggNkGLK)

  [![image.png](https://i.postimg.cc/0jbFbCRW/image.png)](https://postimg.cc/XB6QPdvd)

***

## Tarefa 6 — **Packers** (visão geral)

Apresenta a **ofuscação por empacotamento**: ferramentas que **compactam/criptografam** um executável e o colocam dentro de um **wrapper** (stub) que **desempacota em tempo de execução**. Muitos *packers* ainda somam **anti-debugging**, **imports dinâmicos** e alteração de **seções/entropia**, o que prejudica assinaturas estáticas e exige **análise dinâmica** ou **desempacotamento**.

**Como funciona (visão rápida):**

* Substitui o PE original por um **stub** + **payload comprimido/criptografado**.
* Em execução, o stub **descriptografa** o payload e o **mapeia na memória**, reconstruindo imports/IAT conforme necessário.
* Pode alterar **nomes de seção** (ex.: `UPX0/UPX1/UPX2`), **tabela de imports** e elevar a **entropia** (indício forte de empacotamento).

**Por que é usado (e por quem):**

* **Malware:** esconder strings/APIs, retardar análise, encolher binário, acoplar anti-RE.
* **Software legítimo:** proteção de IP e antitamper (ex.: *Themida* em jogos).

**Implicações para análise:**

* **Estática** fica limitada: pouca visibilidade de strings, imports e código real.
* **Dinâmica** se torna chave: identificar o packer, **desempacotar** (ferramenta/serviço) ou **despejar da memória** pós-unpack.
* Indicadores práticos: **alta entropia**, seções incomuns, imports mínimos, stub curto chamando **LoadLibrary/GetProcAddress**.

**Exemplos comuns (vistos em campo):**
`UPX`, `Themida`, `VMProtect`, `ASPack`, `MPress`, `PELock`, `ExeStealth`, `Alternate EXE Packer`, `hXOR-Packer`, `Milfuscator`.

**Ideia central:**
*Packers* **adiam** a visibilidade do código real para o **runtime**; por isso, a estratégia é **identificar** o packer e **trazer o payload à luz** (automação quando possível; manual com depurador/IAT fix quando necessário).

### Perguntas da tarefa

* **Qual é a string decodificada** da Base64 `"VGhpcyBpcyBhIEJBU0U2NCBlbmNvZGVkIHN0cmluZy4="`?

  **Resposta:** `This is a BASE64 encoded string.`

  [![image.png](https://i.postimg.cc/KzRhMbtT/image.png)](https://postimg.cc/N9qz3WJG)

***

## Tarefa 7 — Identificando e **desempacotando**

Demonstra, na prática, como **lidar com binários empacotados**: primeiro **identificar** o *packer* (assinaturas/entropia em DIE/PEStudio) e, quando necessário, **desempacotar manualmente** via **depurador** (capturar o payload **já descompactado na memória**) e **corrigir a IAT** com o plugin **Scylla**.

**Como a técnica funciona (visão rápida):**

* **Identificação por assinatura/entropia:** o **Detect It Easy (DIE)** tenta reconhecer o *packer*; a aba **Entropy** revela seções com **alta entropia** (indício de compressão/criptografia).
* **Pistas em PEStudio:** nomes de seção “não padrão” (ex.: `UPX0/UPX1/UPX2`) e metadados (ex.: **Microsoft Linker version**, entropia por seção) sugerem **UPX** e afins.
* **Automatizado vs. manual:** alguns *packers* (ex.: **UPX**) possuem **desempacotador**; outros requerem **dump de memória** + **fix de IAT** para obter um executável limpo.
* **Marcadores de execução (exemplo UPX):** ao atingir um endereço “ponte” (ex.: `004172D4`) o stub já **descompactou** o corpo “real”, que **inicia** em `00401262`. É nesse ponto que fazemos o **dump**.

**Passo a passo (do reconhecimento ao binário limpo):**

1. **Identificar o packer**

1) Clique direito em `C:\Malware\packed.exe` → **Detect it easy (DIE)**.

    [![image.png](https://i.postimg.cc/HnXKwknj/image.png)](https://postimg.cc/q6Bj008d)

2) Verifique o **best guess** e abra **Entropy** para confirmar seções “**packed**”.

    [![image.png](https://i.postimg.cc/LsJxk801/image.png)](https://postimg.cc/gwPymmpz)

    [![image.png](https://i.postimg.cc/15W0WZF0/image.png)](https://postimg.cc/8j6r59ts)

3) Abra o **pestudio** → *File → Open* → `packed.exe` → **sections (self-modifying)** e confira nomes de seção (`UPX0/UPX1/UPX2`) e **entropias**.

    [![image.png](https://i.postimg.cc/Cx3qtMxT/image.png)](https://postimg.cc/4Y1yHgcB)

    [![image.png](https://i.postimg.cc/fWqkpWHR/image.png)](https://postimg.cc/MXRWjw2C)

2. **Desempacotar manualmente (dump + IAT fix com Scylla)**

1) Abra `packed.exe` no **x32dbg**.
2) **CTRL+G** → vá para `004172D4` → **F2** (breakpoint) → **F9** (duas vezes) para parar ali.

    [![image.png](https://i.postimg.cc/SxFQbnck/image.png)](https://postimg.cc/jLcYzd8F)

    [![image.png](https://i.postimg.cc/gjLpfpJn/image.png)](https://postimg.cc/N2tZHWRc)

    [![image.png](https://i.postimg.cc/7PjJbYS0/image.png)](https://postimg.cc/dDmVxFHV)

3) **F7** para saltar/jumpar até `00401262` (início do programa **já descompactado**).

    [![image.png](https://i.postimg.cc/pdVp39tp/image.png)](https://postimg.cc/pm7WFLrH)

4) Menu **Plugins → Scylla** → **Dump** (salve o dump ao lado do original).

    [![image.png](https://i.postimg.cc/QxLVhYr5/image.png)](https://postimg.cc/SjrmGrRN)

    [![image.png](https://i.postimg.cc/YCRSNpLQ/image.png)](https://postimg.cc/rDdcM2Jp)

    [![image.png](https://i.postimg.cc/W4fbKXnS/image.png)](https://postimg.cc/1gwQV0bN)

5) No Scylla: **IAT Autosearch** → **OK** → **Get Imports** → remova entradas inválidas (*Cut thunk*).

    [![image.png](https://i.postimg.cc/VNFXCQmx/image.png)](https://postimg.cc/XZprRPZL)

    [![image.png](https://i.postimg.cc/mZKVR4DN/image.png)](https://postimg.cc/Dmd1dt18)

6) **Fix Dump** → selecione o dump salvo → será criado um `*_SCY.exe` com IAT corrigida.

    [![image.png](https://i.postimg.cc/cHrMykNx/image.png)](https://postimg.cc/vgw65zBk)

7) Execute o `*_SCY.exe`: o programa roda **sem o wrapper**, pronto para análise.

    [![image.png](https://i.postimg.cc/SKKcC1Rs/image.png)](https://postimg.cc/jDGWrXz0)

    [![image.png](https://i.postimg.cc/xCwHr0Rg/image.png)](https://postimg.cc/TLchgvBW)

**Ideia central:**
Empacotadores **adiam a visibilidade** do código até o **runtime**. Ao **capturar o payload** já em memória e **corrigir importações**, você obtém um binário **analisável**, contornando a ofuscação do *packer*.

### Perguntas da tarefa

* **De acordo com o DetectItEasy, qual é a versão do Microsoft Linker** usada para **linkar** `packed.exe`?

  **Resposta:** `14.16`

* **De acordo com o pestudio, qual é a entropia** da seção **UPX2** de `packed.exe`?

  **Resposta:** `2.006`

***

## Tarefa 8 — Conclusão

Encerrando a sala, ficou claro o **caráter de “corrida armamentista”** entre analistas e autores de malware: para cada nova técnica defensiva, surge uma nova **anti-RE** (anti-engenharia reversa). Praticamos como **detectar, contornar e instrumentar** essas barreiras para voltar a enxergar o comportamento real.

**Foi visto:**

* **Anti-depuração (overview):** reconhecimento de depuradores (ex.: `IsDebuggerPresent`) e táticas para degradar ferramentas de análise.
* **Anti-depuração na prática:** congelamento via **`SuspendThread`** e **bypass por patching** (NOPs), com **exportação/importação de patches** no x32dbg.
* **Detecção de VM (overview):** processos, chaves de registro, OUIs/MAC, recursos, periféricos, domínio, **timing**.
* **Detecção de VM por temperatura (WMI):** manipulação direta de memória (`[ebp-18]`), ajuste de fluxo com **EIP** para garantir o caminho desejado.
* **Packers (overview):** por que elevam entropia, escondem importações e atrasam a visibilidade do payload.
* **Identificação e desempacotamento:** **DIE/PEStudio** (assinaturas/entropia/seções `UPX*`), **dump em memória** no x32dbg e **fix de IAT** com **Scylla**.

**Ideia central:** técnicas anti-RE **condicionam** o comportamento ao ambiente/ferramentas. Ao **controlar execução** (breakpoints, step, registradores), **modificar código** (patches) e **operar na memória** (dump/IAT), você restaura a **observabilidade** e retoma a análise.


### Perguntas da tarefa

Nenhuma resposta necessária.

***

**Fim!** 

[![image.png](https://i.postimg.cc/kGwnQsB6/image.png)](https://postimg.cc/MfM2xy0q)

<br>

<p align="center">
  <img src="https://github.com/vin1sss/WriteUps/blob/main/attachments/ninjaduck.png" width="50"/>
</p>

##
