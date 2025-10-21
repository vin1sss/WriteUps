# Análise Dinâmica

“Análise dinâmica” é a **observação sistemática do comportamento de um programa/sistema em execução**. Em vez de concluir apenas a partir do código (abordagem estática), a análise dinâmica mede **efeitos concretos** — criação de processos, acessos a arquivos, uso de APIs, tráfego de rede, consumo de memória/CPU, eventos temporais — para **inferir capacidades, qualidade e riscos** no contexto real de execução.

Em termos formais, é uma **investigação causal em tempo de execução**: dados de entrada e condições ambientais geram **efeitos observáveis**; correlacionando esses efeitos, inferimos **propriedades funcionais e não funcionais** (corretude, desempenho, confiabilidade, compatibilidade).

---

## Por que existe

1. **Expor o que só aparece em runtime:** componentes configurados/tardios, dados computados, caminhos condicionais, latências reais.
2. **Validar hipóteses da análise estática:** confirmar se o que “parece que faz” **de fato acontece** sob certas condições.
3. **Gerar evidências para engenharia de qualidade:** métricas de desempenho, cobertura de cenários, gargalos, condições de corrida, erros de integração.
4. **Reduzir risco operacional:** entender efeitos colaterais (em arquivos, serviços, rede, memória) antes de liberar em produção.

---

## Como funciona (visão conceitual)

### 1) Arquitetura em camadas

* **Ambiente controlado:** definição de cenário (SO, permissões, rede, dados de teste) e limites de segurança.
* **Camada de observação:** coleta de **telemetria** (processos, arquivos, registros/configs, chamadas de API, rede, métricas de recurso, eventos de tempo).
* **Camada analítica:** correlação temporal/semântica dos eventos para **explicar comportamentos** e **extrair artefatos verificáveis** (logs, traços, medidas).

### 2) Gatilhos comuns

* **Entradas/ações do usuário:** cliques, teclado, abertura de documentos.
* **Eventos de sistema:** inicialização/logon, agendamentos, reinícios de serviço.
* **Condições de rede:** on/offline, latência, limites de banda, respostas simuladas.
* **Parâmetros/ambiente:** variáveis, *feature flags*, relógio, localidade.

### 3) Objetivos táticos

* **Caracterizar fluxos de execução:** quais caminhos são percorridos e quando.
* **Medir propriedades não funcionais:** tempos de resposta, consumo de recursos, estabilidade.
* **Capturar artefatos úteis:** arquivos criados, chaves de configuração, endpoints acessados, formatos de mensagem.

---

## Taxonomia conceitual da análise dinâmica

### A) Observação caixa-preta (passiva)

Execução com monitoramento de efeitos externos (FS, processos, rede, métricas). Útil para **triagem** e **métricas gerais**.

**Racional teórico:** maximiza realismo e minimiza interferência, mas **não revela** estados internos nem decisões finas.

---

### B) Observação instrumentada (semi-invasiva)

*Tracing/hooking* de APIs, eventos do sistema, inspeção de memória em pontos controlados, contadores de desempenho.

**Racional teórico:** aumenta a **granularidade causal** (qual API, parâmetros, sequência), ao custo de possível **efeito observador** (alterar timing/comportamento).

---

### C) Controle ativo (depuração e experimentação)

*Breakpoints*, *single-step*, inspeção/alteração de **registradores, memória e variáveis**, injeção de falhas, variação sistemática de inputs.

**Racional teórico:** permite **isolar e manipular** caminhos e estados para explicar comportamentos, com o trade-off de **interferência** mais alta e maior custo operacional.

---

## Exemplos de **observáveis** (evidências)

* **Sistema/host:** criação de processos e threads, serviços/tarefas, alterações de configuração, bloqueios/erros, uso de CPU/memória/IO.
* **Arquivos e dados:** leituras/escritas, formatos, tamanhos, diretórios de trabalho, *locks*.
* **Comunicação:** conexões estabelecidas, protocolos, latências, *timeouts*, formatos de mensagens.
* **Memória/estado:** estruturas alocadas, strings carregadas em runtime, crescimento e picos.
* **Temporalidade:** ordens de eventos, *backoffs*, esperas, janelas de ativação.

> Observação: a **força explicativa** cresce quando **múltiplas fontes** (host + dados + rede + tempo) **convergem** para a mesma conclusão.

---

## Relação com teste, validação e engenharia

* **Teste funcional:** confirmar requisitos sob cenários realistas.
* **Teste de desempenho/confiabilidade:** *stress/soak/load*, identificação de gargalos e *bottlenecks*.
* **Integração e compatibilidade:** observar efeitos em serviços externos, sistemas legados e ambientes variados.
* **Observabilidade e SRE:** geração de métricas, logs e *traces* que viram **telemetria de produção** (baselines e SLOs).

---

## Limitações e trade-offs

* **Cobertura parcial:** depende do **cenário e inputs** escolhidos; caminhos não exercitados permanecem desconhecidos.
* **Não determinismo:** concorrência e temporização podem **variar resultados**; é preciso replicar condições.
* **Efeito observador:** instrumentação e depuração podem **alterar** o comportamento medido.
* **Custo e segurança:** montar ambientes, isolar impactos e lidar com dados reais **exige disciplina** (tempo e recursos).

---

## Considerações éticas e de pesquisa

Análises devem respeitar **licenças, privacidade e limites de segurança** do ambiente. Executar software com dados sensíveis requer **consentimento, isolamento e minimização**. Pesquisas devem prezar por **reprodutibilidade**, documentação clara de cenários e **responsabilidade** na divulgação de descobertas.
