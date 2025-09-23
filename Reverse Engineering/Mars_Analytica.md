# DuckWare Team - Mars Analytica (NorthSec 2018)
###### Solved by @vin1sss

> This is a CTF about Reverse Engineering

## Desafio: Mars Analytica (Engenharia Reversa)
#### Introdução

Este desafio originalmente foi aplicado no CTF presencial que aconteceu na [NorthSec conference](https://nsec.io/) em 2018. 

Neste desafio, recebemos um binário ELF de 64 bits para Linux chamado `tarefa1` (originalmente chamado `MarsAnalytica`).

Ao executá-lo utilizando o comando `./tarefa1`, o programa exibe um banner e solicita um `Citizen Access ID`:

[![Captura-de-tela-2025-09-18-155119.png](https://i.postimg.cc/jj49yqyV/Captura-de-tela-2025-09-18-155119.png)](https://postimg.cc/GB9qCCMz)

Qualquer entrada incorreta resulta no travamento e fim do pograma:

[![image.png](https://i.postimg.cc/kXgn1PW2/image.png)](https://postimg.cc/c6PyCj9W)

Isso sugere que o nosso objetivo é realizar [engenharia reversa](https://medium.com/ipnet-growth-partner/engenharia-reversa-e-o-seu-papel-dentro-da-seguran%C3%A7a-cibern%C3%A9tica-c70e1f84d4ec) neste executável, descobrir como o programa valida o `Citizen Acess ID` e encontrar o valor correto que revela a **flag**.

#### Reconhecimento Inicial do Binário

Antes de abrir o binário em um [Descompilador](https://medium.com/@raw.rewa10/decompilers-and-reverse-engineering-6b4acf3f76ff), vamos fazer uma análise inicial.

Executando o comando `file` no terminal, identificamos o tipo do arquivo e algumas características:

[![image.png](https://i.postimg.cc/RF8ZKPbP/image.png)](https://postimg.cc/ftcshvsX)

Executando o comando [`strings`](https://pt.wikipedia.org/wiki/Strings_(Unix)):

[![image.png](https://i.postimg.cc/rpNnPs6W/image.png)](https://postimg.cc/QBVJBXdx)

Obtemos algo, temos a string `UPX!` repetindo algumas vezes, além deste comentário informando que o arquivo está comprimido com [UPX](https://pt.quarkus.io/guides/upx):

[![image.png](https://i.postimg.cc/W36drRks/image.png)](https://postimg.cc/xJ8jD45Z)

Então antes de tudo, vamos descompactar utilizando o comando `upx -d tarefa1 -o tarefa1_unpacked
`:

[![image.png](https://i.postimg.cc/NGHnxx67/image.png)](https://postimg.cc/23CTkhXV)

Agora o tamanho do binário salta para quase 11 MB.

Fora isso, o comando `strings` não mostra nada útil, confirmando que as strings estão ofuscadas ou geradas em runtime.

#### Análise Estática com Binary Ninja 

Para fazer a [Análise Estática](https://www.checkpoint.com/pt/cyber-hub/cloud-security/what-is-static-code-analysis/) do binário utilizaremos o [Binary Ninja](https://en.wikipedia.org/wiki/Binary_Ninja).

[![image.png](https://i.postimg.cc/fRhrCLGn/image.png)](https://postimg.cc/Mvd5KWZd)

Vamos configurar a exibição para **modo gráfico** e **High Level IL**.

Ao analisar o gráfico nota-se a complexidade absurda e a grande quantidade de funções:

[![image.png](https://i.postimg.cc/PJfccfzr/image.png)](https://postimg.cc/r0bQRkPv)

Cada bloco desse é um novo conjunto de funções e chamadas: 

[![image.png](https://i.postimg.cc/90vd9JM5/image.png)](https://postimg.cc/K3D3S7BJ)

Para tentar entender de maneira mais simples, exportei o código linear do Binary Ninja e pedi ao [Manus](https://manus.im/pt-br/about) para analisar e me explicar o fluxo do código. Esta foi sua resposta:

---

**1. Estrutura**

* **Inicialização:** cópia de duas grandes tabelas constantes para memória local (`0xe551e0` e `0xe57d40`).
* **Registradores virtuais:** dezenas de variáveis locais manipuladas como estado interno.
* **Fluxo principal:** série de transformações aritméticas/bitwise em laços longos.
* **Lookups:** índices calculados via multiplicações e módulos usados para acessar tabelas.
* **Dispatch:** salto indireto controlado por valores de tabelas (`jump(*(arg1 - 0x5d328) + ...)`).
* **Saída:** sub-rotina (`sub_4009d7`) gera caracteres, enviados a `putchar`.

---

**2. Strings e chamadas externas**

* Strings claras: `ACCESS GRANTED`, `ACCESS DENIED`.
* Chamadas de libc: `puts`, `putchar`, `getchar`, `memcpy`, `malloc`, `time`, `srand`, `exit`, etc.

---

**3. Características**

* Computed jumps → máquina virtual/obfuscation.
* Uso intensivo de tabelas → validação dependente de lookups.
* Impressão caractere a caractere da saída.

---

**4. Interpretação funcional**

1. Carrega tabelas e inicializa estado.
2. Executa múltiplos rounds de transformação sobre input.
3. Usa índices derivados do estado para consultar tabelas.
4. Calcula próximo bloco via dispatch indireto.
5. Em condições de sucesso, gera saída: “ACCESS GRANTED” seguido do valor correto (*citizen id*).

---

**5. Estratégias de resolução**

* **Emulação em Python:** reproduzir lógica HLIL com dumps das tabelas → gera *citizen id*.
* **Execução monitorada:** rodar em GDB, dumpar tabelas e capturar saída de `putchar`.
* **Simbólica (angr):** possível, mas exigiria hooks para os dispatchs computados.

---

**6. Conclusão**

O executável implementa um verificador obfuscado de *citizen id*, baseado em máquina de estados com tabelas de lookup e saltos indiretos. O caminho de sucesso resulta em `ACCESS GRANTED` e exibe a sequência correta.

---

#### Solução

Infelizmente, apesar de múltiplas tentativas de reverter o complexo conjunto de transformações aritméticas, saltos computados e demais operações presentes no fluxo principal, não consegui alcançar um resultado conclusivo.

Porém, para não deixar esta documentação sem um resultado, encontrei um write-up desta questão que apresenta uma solução detalhada e aparentemente chega a um resultado válido:

* [NorthSec 2018 - Mars Analytica](https://re-dojo.github.io/posts/nsec2018/mars_analytica/)

**Resumo da solução do autor**:

---

**1. Reconhecimento inicial**

* O binário estava empacotado com **UPX**, e após descompressão aumentou para \~11 MB.
* Era um ELF 64-bit *stripped*, sem *strings* úteis.

**2. Identificação da máquina virtual**

* A `main` copiava **cinco tabelas grandes** para a stack.
* O fluxo consistia em **saltos indiretos (`push/ret`, `jmp rax`)**, típico de **virtualização/VM**.
* Cada “handler” calculava o próximo endereço → confirmado em depuração (GDB).

**3. Análise dos handlers**

* Implementação de instruções de máquina virtual em cima de **listas** (`malloc` → inserção, `free` → remoção).
* A VM era **stack-based**.
* Alguns handlers simples foram identificados por padrão de chamadas, ex.:

  * `pop_val + putchar + fflush` → instrução `putchar`.
  * `pop_val + pop_val + push_val` + operação → `xor`, `sub`, `mul`, etc.

**4. Extração do algoritmo de verificação**

* A entrada (`citizen id`) era armazenada em posições específicas do stack virtual com ordem embaralhada:

  ```
  [7, 8, 13, 15, 16, 26, 27, 22, 21, 4, 18, 28, 23, 29, 9, 1, 25, 30, 17]
  ```
* Após isso, o programa aplicava várias **equações aritméticas e bitwise** entre caracteres.
* Exemplo:

  ```
  (buf[14] * buf[6]) * ((buf[12] - buf[10]) ^ buf[13]) - 0x3fcf == 0
  ```

**5. Resolução com Z3**

* O autor modelou cada caractere como variável simbólica (`BitVec`).
* Impôs restrições: caracteres imprimíveis.
* Adicionou todas as equações extraídas.
* Rodou o solver **Z3**, que retornou a solução.

```
from z3 import *

solver = Solver()

# Flag order
order=[7,8,13,15,16,26,27,22,21,4,18,28,23,29,9,1,25,30,17]
flag={}
for i in order:
	flag[i] = BitVec("c_%d" % i,8)

#char constraints
for k in flag.keys():
	solver.add(flag[k] >= 32, flag[k] <= 126)
	
#equations
solver.add((flag[9] * flag[27])*((flag[23]-flag[18])^flag[29]) == 0x3fcf)
...
solution=''
if solver.check() == z3.sat:
    model=solver.model()
    for i in order:
        solution+=chr(model[flag[i]].as_signed_long())
print solution
```

**6. Resultado**

O `citizen id` válido obtido foi:

```
q4Eo-eyMq-1dd0-leKx
```

---

**Resumo (do resumo)**: o binário implementava uma máquina virtual ofuscada. O write-up evitou decodificar todos os handlers manualmente e explorou padrões repetidos para identificar instruções. Extraindo as equações de verificação, resolveu-as via **constraint solving (Z3)**, chegando ao `citizen id` correto.

#### Conclusão

Apenas para confirmar, ao executar o binário e inserir o `citizen id` gerado, a flag é retornada:

[![image.png](https://i.postimg.cc/KzhbGD39/image.png)](https://postimg.cc/rdj3gWJW)

Flag:
>`FLAG-l0rdoFb1Nq4EoeyMq1dd0leKx`

O resultado final ficou bem distante do que eu desejava, mas mesmo sem ter conseguido resolver o desafio por completo, a experiência foi extremamente enriquecedora. Durante o processo, pude explorar de forma prática diversas ferramentas de análise estática de código, entender melhor o fluxo de execução, mergulhar em detalhes de assembly e experimentar diferentes abordagens de engenharia reversa. Tudo isso trouxe um aprendizado valioso e ampliou bastante minha confiança para enfrentar desafios semelhantes no futuro.
