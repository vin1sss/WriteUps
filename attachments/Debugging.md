# Depuração (Debugging)

“Depuração” é o **processo sistemático de observar, isolar, explicar e corrigir** comportamentos indesejados de um programa **em execução**. Diferente de teste (que verifica se algo está certo), a depuração busca **por que algo deu errado**, combinando **observação** do estado interno/externo com **intervenções controladas** (pausas, inspeção, alteração de variáveis, reexecução de trechos).

Em termos formais, é uma **investigação causal em tempo de execução**: partindo de um sintoma, formulamos hipóteses, coletamos evidências, **refinamos o modelo mental** do sistema e convergimos para a **causa-raiz** e sua **correção verificável**.

---

## Por que existe

1. **Encontrar e corrigir defeitos**: reduzir MTTR (tempo médio para recuperação) e restaurar a funcionalidade.
2. **Validar hipóteses de design**: confirmar o que “deveria acontecer” versus o que **acontece de fato**.
3. **Aprimorar qualidade e confiabilidade**: entender falhas intermitentes, *race conditions*, *deadlocks*, vazamentos, regressões.
4. **Apoiar engenharia contínua**: gerar *fixes* acompanhados de **testes de regressão** e melhorias de observabilidade.

---

## Como funciona (visão conceitual)

### 1) Arquitetura em camadas

* **Ambiente reprodutível**: cenário controlado (versões, dados, rede) para repetir o problema.
* **Camada de observação**: coleta de **traços, *stack traces*, métricas, logs, *dumps*, contadores**.
* **Camada de controle**: **breakpoints, *watchpoints*, *stepping***, alteração de estado, *record/replay*.
* **Camada analítica**: correlação temporal/semântica, eliminação de hipóteses, **bisseção** (binária) do espaço de causas.

### 2) Gatilhos comuns

* **Exceções/erros** (lançadas ou capturadas).
* **Condições temporais** (timeouts, *schedulers*, reentrâncias).
* **Eventos de E/S** (rede, disco, IPC).
* **Transições de estado** (inicialização, *shutdown*, *failover*).

### 3) Objetivos táticos

* **Reproduzir → isolar → explicar → corrigir → prevenir** (testes de regressão).
* **Reduzir escopo** com bisseção: mudar **uma variável por vez** até localizar a fronteira do defeito.
* **Tornar visível o invisível**: expor estados e sequências que o comportamento externo não revela.

---

## Taxonomia conceitual da depuração

### A) Observação orientada a evidências (caixa-preta)

* Foco em **logs, métricas, *stack traces*, *dumps*** e reprodução do sintoma sem alterar o binário.
* **Prós**: baixo acoplamento, alta fidelidade ao ambiente real.
* **Contras**: visibilidade limitada do estado interno e de decisões finas.

---

### B) Depuração instrumentada (semi-invasiva)

* **Tracing/hooking**, *profilers*, *event providers*, *record/replay*, amostragem de CPU/memória, *time series* detalhadas.
* **Prós**: maior **granularidade causal** (quem chamou o quê, com quais parâmetros e quando).
* **Contras**: **efeito observador** (overhead/timing), necessidade de símbolos/compilações específicas.

---

### C) Depuração interativa (controle ativo)

* **Breakpoints** (linha, função, condicional), **watchpoints** (dados), **step in/over/out**, inspeção/edição de **variáveis, registradores e memória**, *time-travel debugging*.
* **Prós**: poder explicativo máximo; permite **isolar e manipular** caminhos críticos.
* **Contras**: mais intrusiva; pode **mascarar heisenbugs** (erros sensíveis a tempo), exige habilidade e preparo de ambiente.

---

## Exemplos de **evidências** e artefatos úteis

* **Fluxo de execução:** *call stacks*, gráficos de chamadas, ordem de eventos.
* **Estado e dados:** *core/minidumps*, *heap snapshots*, valores de variáveis, *watch expressions*.
* **Concorrência:** estados de *threads*, *locks*, detecção de *deadlocks/livelocks*.
* **Desempenho:** perfis de CPU/IO/alocações, *hot paths*, contadores de espera/contensão.
* **Confiabilidade:** logs de erro/exceção com contexto (correlação entre serviços, IDs de requisição).

> A explicação se fortalece quando **múltiplas evidências convergem** para a mesma hipótese causal.

---

## Relação com teste, validação e engenharia

* **Teste ≠ depuração**: teste detecta o desvio; depuração **explica e corrige**.
* **Ciclo virtuoso**: cada conserto deve vir com **teste de regressão** e, idealmente, melhoria de observabilidade.
* **CI/CD e bisseção**: integração contínua e *git bisect* aceleram a localização da *change* culpada.
* **SRE/operabilidade**: incidentes alimentam **post-mortems**, *runbooks* e *dashboards* para reduzir MTTR.

---

## Limitações e *trade-offs*

* **Heisenbugs** e não determinismo: o ato de observar altera o comportamento.
* **Overhead/segurança**: instrumentação pode degradar performance e expor dados sensíveis.
* **Cobertura e custo**: ambientes distribuídos e heterogêneos tornam a reprodução cara/complexa.
* **Simbolização e versões**: falta de símbolos, *builds* otimizados e divergência de versões dificultam a análise.

---

## Considerações éticas e de pesquisa

* **Privacidade e sigilo**: evite registrar/inspecionar **credenciais, PII** e segredos; aplique **anonimização/redação**.
* **Produção com cautela**: depurar *in-prod* requer **limites claros**, *feature flags* e **controles de acesso**.
* **Reprodutibilidade**: documente versões, passos e artefatos; permita que terceiros **verifiquem** a correção.
