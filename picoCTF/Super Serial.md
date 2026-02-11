# Super Serial (picoCTF)
###### Solved by @biancardesouza

> This is a CTF about Web Exploitation

## Desafio: Super Serial (exploração web)
#### Introdução

Este é um desafio de nível médio disponibilizado na plataforma [picoCTF](https://picoctf.org), relacionado a exploração de arquivos e manipulação de [cookies](https://www.kaspersky.com.br/resource-center/definitions/cookies).

- [Página do desafio](https://play.picoctf.org/practice/challenge/180)

A questão consiste em explorar os arquivos de uma página e interpretar seus códigos, de forma a obter uma flag ao alterar o cookie de login.

#### Análise Inicial

O enunciado do desafio primeiramente nos mandar iniciar uma instância para acessarmos o site que exploraremos. Após isso, ele nos mostra o seguinte texto:

> *"Try to recover the flag stored on this website."*  
> *(Tente recuperar a flag armazenada neste site)*

E logo em seguida o link.

#### Solução

Ao abrirmos o site vemos apenas uma página simples de login.

[![image.png](https://i.postimg.cc/bwMfBZYL/image.png)](https://postimg.cc/WdMyDbkJ)

Inspecionando o site não é possível achar nada, então iremos acessar o [`robots.txt`](https://www.hostgator.com.br/blog/robots-txt-como-criar/?gad_source=1&gad_campaignid=22134387764&gclid=Cj0KCQiAnJHMBhDAARIsABr7b85FNrcVEjDQStvdlYyq40_8KNwN6NIxvOg3jz42UcGSFJMT3HL9CBsaAlhWEALw_wcB) para descobrirmos se existe algum arquivo escondido.

[![image.png](https://i.postimg.cc/tCTfG9M8/image.png)](https://postimg.cc/qgPQ8H0G)

Acessando o arquivos, achamos o arquivo `admin.phps`. PHPS é utilizado para exibir o código-fonte de um script PHP com destaque de sintaxe (colorido) no navegador, em vez de executá-lo. Nós não conseguimos abrí-lo pois não temos a permissão de administrador.

[![image.png](https://i.postimg.cc/Bbwmn1z0/image.png)](https://postimg.cc/rRx1hzcn)

Vamos tentar acessar o código fonte da página inicial colocando um `/index.phps` no final da URL, para que possamos ler seu código fonte.

[![image.png](https://i.postimg.cc/Y0VxHBvy/image.png)](https://postimg.cc/LgtPt7Bz)

Lendo esse código, podemos ver que existe mais um script chamado `authentication.phps`. Vamos trocar o "index" por "authentication" no link para acessá-lo.

[![image.png](https://i.postimg.cc/wBVcBmtV/image.png)](https://postimg.cc/jWDJgC4w)

Analisando os códigos que obtemos, é possível observar que o sistema utiliza a função [unserialize](https://www.php.net/manual/en/function.unserialize.php) diretamente em dados fornecidos pelo usuário, o que nos permite injetar objetos PHP. Vamos manipulat um objeto.

`O:10:"access_log":1:{s:8:"log_file";s:7:"../flag";}`

Depois precisamos codificar em [Base64](https://developer.mozilla.org/pt-BR/docs/Glossary/Base64). Para isso, vamos utilizar o [CyberChef](https://gchq.github.io/CyberChef/#recipe=To_Base64('A-Za-z0-9%2B/%3D')).

`TzoxMDoiYWNjZXNzX2xvZyI6MTp7czo4OiJsb2dfZmlsZSI7czo3OiIuLi9mbGFnIjt9`

Agora, pelo terminal, iremos criar um cookie chamado "login" e adicionar essa string como valor.

[![image.png](https://i.postimg.cc/zXzk9pV3/image.png)](https://postimg.cc/jC1NfH6r)

### Conclusão

O desafio Super Serial é importante porque evidencia os riscos da desserialização insegura em aplicações PHP antigas. Ele nos mostra como dados, aparentemente inofensivos, podem ser manipulados quando o servidor confia demais nas informações recebidas do cliente, reforçando a importância de validar e proteger dados no lado do servidor.

`Flag: picoCTF{1_c4nn0t_s33_y0u_2fba20fa}`