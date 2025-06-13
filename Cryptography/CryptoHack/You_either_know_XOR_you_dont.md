# DuckWare Team - You either know, XOR you don't (CryptoHack)
###### Solved by @vin1sss

> This is a CTF about Cryptography

## Desafio: You either know, XOR you don't (Criptografia)
#### Introdução

Este é o último desafio da trilha de aprendizado ["Introduction to CryptoHack"](https://cryptohack.org/courses/intro/) da plataforma [CryptoHack](https://cryptohack.org/). A trilha é composta por uma série de 10 exercícios introdutórios voltados ao estudo da [criptografia](https://pt.wikipedia.org/wiki/Criptografia) e ao funcionamento da própria plataforma. Seu principal objetivo é proporcionar uma base sólida nos conceitos fundamentais da criptografia moderna, por meio da prática com desafios progressivos e acessíveis para iniciantes.

- [Página do desafio](https://cryptohack.org/courses/intro/xorkey1/)

Esta questão consiste em decriptar uma string em [hexadecimal](https://pt.wikipedia.org/wiki/Sistema_de_numera%C3%A7%C3%A3o_hexadecimal) para encontrar a flag.

#### Análise Inicial

Após ensinar o funcionamento do [algoritmo XOR](https://www.101computing.net/xor-encryption-algorithm/) nos desafios anteriores, agora é apresentado algo um pouco mais desafiante. O enunciado é simples, contendo apenas a frase:

> *"I've encrypted the flag with my secret key, you'll never be able to guess it."*  
> *(Eu criptografei a bandeira com minha chave secreta, você nunca será capaz de adivinhá-la.)*

Além disso, há uma dica:

> *"Remember the flag format and how it might help you in this challenge!"*  
> *(Lembre-se do formato da bandeira e como ele pode ajudar você neste desafio!)*

E também uma string em hexadecimal:

0e0b213f26041e480b26217f27342e175d0e070a3c5b103e2526217f27342e175d0e077e263451150104

Print do enunciado:
[![Captura-de-tela-2025-06-13-134751.png](https://i.postimg.cc/P5b2Kb0Z/Captura-de-tela-2025-06-13-134751.png)](https://postimg.cc/bZJxwnyN)

#### Interpretando a dica

A dica fornecida no enunciado nos direciona a uma única abordagem lógica. Sabemos que o algoritmo XOR opera sobre dois conjuntos de dados:

[![xor-xor.webp](https://i.postimg.cc/yNDhtwNM/xor-xor.webp)](https://postimg.cc/4KThh2VP)

Neste caso, uma das entradas é a string em hexadecimal apresentada no desafio. Assim, resta deduzirmos a segunda entrada, que deve ser algo que já conhecemos.

Como aprendemos ao longo da trilha, todas as flags seguem um formato padronizado e começam com `"crypto{"`. Podemos, portanto, usar esse trecho conhecido da flag como ponto de partida para tentar descobrir qual foi a chave secreta utilizada na cifra.

#### Solução

Para resolver este desafio, utilizarei o site [dcode.fr](https://www.dcode.fr/xor-cipher), que contém uma ferramenta prática para realizar operações de [XOR](https://www.101computing.net/xor-encryption-algorithm/) entre textos e/ou valores hexadecimais. Essa ferramenta facilita a aplicação do algoritmo e nos permite testar hipóteses sobre a chave ao comparar os resultados com o formato esperado da flag.

Então, inserindo a string em hexadecimal fornecida no enunciado diretamente na caixa de texto principal (que aceita entradas em hexadecimal, sem a necessidade de conversão prévia), e o trecho conhecido da flag `"crypto{"` na caixa de texto da opção marcada **"Use the ascii key"** (que permite entradas em texto padrão), realizamos a primeira análise:

[![Captura-de-tela-2025-06-13-141610.png](https://i.postimg.cc/6q2y50Cm/Captura-de-tela-2025-06-13-141610.png)](https://postimg.cc/YL7pdQn6)

A ferramenta aplicará o XOR entre os primeiros bytes da string e a palavra `"crypto{"`, revelando a parte inicial da chave secreta utilizada na criptografia:

[![Captura-de-tela-2025-06-13-142408.png](https://i.postimg.cc/xdpHPgy6/Captura-de-tela-2025-06-13-142408.png)](https://postimg.cc/QB5H8Qz7)

Agora que sabemos o começo da chave secreta, `"myXORkey"`, utilizamos essa informação para realizar uma nova operação de XOR. Desta vez, em vez de usar o trecho conhecido da flag, aplicamos diretamente a chave descoberta sobre toda a string em hexadecimal.

[![Captura-de-tela-2025-06-13-143321.png](https://i.postimg.cc/RhSBV12z/Captura-de-tela-2025-06-13-143321.png)](https://postimg.cc/dkxX5CqW)

A própria ferramenta do dcode se encarrega de repetir automaticamente a chave ao longo de toda a entrada — o que é essencial, já que a chave tem apenas 8 bytes, enquanto a string criptografada possui dezenas de bytes. Com isso, conseguimos aplicar corretamente a operação de XOR byte a byte, o que revela o conteúdo original criptografado: a flag completa.

[![Captura-de-tela-2025-06-13-143409.png](https://i.postimg.cc/Sx2psx1j/Captura-de-tela-2025-06-13-143409.png)](https://postimg.cc/21Dt9rDf)

#### Conclusão

Flag:
>`crypto{1f_y0u_Kn0w_En0uGH_y0u_Kn0w_1t_4ll}`

Este desafio foi uma excelente aplicação prática dos conceitos fundamentais de criptografia com XOR. Através de um pequeno trecho conhecido da flag, conseguimos deduzir parte da chave secreta utilizada na cifra. A partir disso, utilizamos uma ferramenta online para repetir essa chave ao longo de toda a mensagem criptografada, revertendo o processo de encriptação e revelando a flag completa.

Mais do que simplesmente encontrar a resposta, este exercício reforça a importância de padrões previsíveis em contextos criptográficos. Quando uma parte da mensagem é conhecida — ou pode ser adivinhada —, todo o sistema se torna vulnerável. Essa lição é essencial não só para resolver desafios em CTFs, mas também para compreender os riscos reais em sistemas mal projetados no mundo da segurança da informação.
