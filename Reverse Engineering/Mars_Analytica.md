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

#### Análise Estática com Ghidra



#### Conclusão

Flag:
>`crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`

Este desafio foi uma excelente aplicação prática dos conceitos fundamentais de criptografia com XOR. Através de um pequeno trecho conhecido da flag, conseguimos deduzir parte da chave secreta utilizada na cifra. A partir disso, utilizamos uma ferramenta online para repetir essa chave ao longo de toda a mensagem criptografada, revertendo o processo de encriptação e revelando a flag completa.

Mais do que simplesmente encontrar a resposta, este exercício reforça a importância de padrões previsíveis em contextos criptográficos. Quando uma parte da mensagem é conhecida — ou pode ser adivinhada —, todo o sistema se torna vulnerável. Essa lição é essencial não só para resolver desafios em CTFs, mas também para compreender os riscos reais em sistemas mal projetados no mundo da segurança da informação.

