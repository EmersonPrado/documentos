# Formatação de resultados de consultas à API do Foreman

#### Índice

1. [Introdução](#introdução)
1. [Filtragem e formatação com `tr`, `sed`, `grep`, etc.](#filtragem-e-formatação-com-tr-sed-grep-etc)
  1. [Separar campos em linhas](#separar-campos-em-linhas)
  1. [Separar campos, inclusive aninhados](#separar-campos,-inclusive-aninhados)
  1. [Filtrar campos](#filtrar-campos)
  1. [Filtrar trechos](#filtrar-trechos)
  1. [Filtrar campos em trechos](#filtrar-campos-em-trechos)
  1. [Registros múltiplos](#registros-múltiplos)
  1. [Mais](#mais)
1. [Filtragem e formatação com `awk`](#filtragem-e-formatação-com-awk)
  1. [Separar campos em linhas](#separar-campos-em-linhas)
  1. [Selecionar campos](#selecionar-campos)
  1. [Remover nome do parâmetro](#remover-nome-do-parâmetro)
  1. [Registros múltiplos](#registros-múltiplos)

## Introdução

Como a API retorna uma string enorme com todos os resultados concatenados, é difícil extrair informações diretamente, especialmente em buscas que retornam vários registros. É necessário formatar a saída de forma mais conveniente.

A complexidade da tarefa de transformar uma linha de texto JSON em uma tabela compreensível exige um processador de textos poderoso. Uma solução simples é concatenar, via _pipes_, vários processadores, como `tr`, `sed` e `grep`. Para formatação mais completa, é possível usar o `awk`, que possui inúmeros recursos de tratamento de strings aliados a lógica de programação.

Em qualquer caso, é bom ter em mente como a saída da API é formatada:

- Registros e seus campos são sempre separados por vírgulas
- Grupos de registros e de campos aninhados ficam entre colchetes
- Cada registro individual e cada campo dentro de campos aninhados fica entre chaves
- Ao listar um único registro, a API retorna uma única linha com o registro (entre chaves)
- Ao listar vários registros, a API retorna uma tabela com estatísticas, depois uma única linha com todos os registros (entre colchetes)
- Cada campo tem o formato `"<Parâmetro>":"<Valor>"`

Abaixo um exemplo de saída com registro único. Note como o registro inteiro está entre chaves, e cada campo separado por vírgulas. No caso dos _hostgroups_, os campos `all_puppetclasses`, `puppetclasses` e `config_groups` contém vários campos aninhados - todos entre chaves e separados por vírgulas, e o conjunto inteiro entre colchetes. O campo `config_groups` também contém um campo aninhado - `puppetclasses` (que não é o mesmo `puppetclasses` anterior) - que, por sua vez, contém seus próprios campos aninhados, com a mesma formatação.

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>'
...
{"id":<ID>,"name":"<Nome>","title":"<Título>",...,"all_puppetclasses":[{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},...],"parameters":[],"template_combinations":[],"puppetclasses":[{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},...}],"config_groups":[{"id":<ID>,"name":"<Nome>","created_at":"<Criação>","updated_at":"<Atualização>","puppetclasses":[{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},{"id":<ID>,"name":"<Nome>","module_name":"<Módulo>"},...]}]}
```

Outro exemplo, agora com registros múltiplos. Note como cada registro está entre chaves, e todos, separados por vírgulas, agrupados entre colchetes.

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X GET -d '{ "search":"title~<Padrão>" }' 'https://<Servidor Foreman>/api/hostgroups'
...
{
  "total": <Quantidade total de registros>,
  "subtotal": <Quantidade de registros filtrados>,
  "page": <Página sendo mostrada>,
  "per_page": <Quantidade de registros por página>,
  "search": "<Expressão de busca>",
  "sort": {
    "by": null,
    "order": null
  },
  "results": [{"id":<ID>,"name":"<Nome>","title":"<Título>",...},{"id":<ID>,"name":"<Nome>","title":"<Título>",...},{"id":<ID>,"name":"<Nome>","title":"<Título>",...},...]
```

## Filtragem e formatação com `tr`, `sed`, `grep`, etc.

### Separar campos em linhas

Uma necessidade natural é separar os campos com quebras de linhas, em vez de vírgulas. Isto pode ser feito facilmente com o comando `tr`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | tr , '\n'
...
{"id":<ID>
"name":"<Nome>"
"title":"<Título>"
...
"all_puppetclasses":[{"id":<ID>
"name":"<Nome>"
"module_name":"<Módulo>"}
...]
...}
```

Convenientemente, cada campo ficou em sua própria linha, permitindo filtragens adicionais com `sed` ou `grep`. Exceto nos campos aninhados (`all_puppetclasses`), pois o campo externo não é separado dos internos por vírgula, então o primeiro campo interno apareceu junto com o externo.

### Separar campos, inclusive aninhados

Como cada campo está delimitado por chaves, podemos separar as linhas em cada conjunto de vírgulas e/ou chaves. Pra esta filtragem mais elaborada, podemos usar o comando `sed s` (substitute):

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g'
...
"id":275
"name":"TREINAMENTO"
"title":"SP/TREINAMENTO"
...
"all_puppetclasses":[
"id":<ID>
"name":"<Nome>"
"module_name":"<Módulo>"
...
]
...
]
```

> A expressão regular `[,{}]` encontra uma vírgula **ou** uma chave. Aqui, queremos uma sequência de qualquer tamanho, mas o `sed` não reconhece o multiplicador `+`, das expressões regulares. Então contornamos com `<Expressão><Expressão>*`.

Desta vez, os campos aninhados foram separados com clareza, ficando o campo externo em uma linha separada.

### Filtrar campos

O comando `grep` permite a seleção de campos específicos:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | grep 'environment_'
...
"environment_id":<ID ambiente>
"environment_name":<Nome ambiente>
```

### Filtrar trechos

Com o comando `sed p` (print), podemos selecionar trechos da saída (opção `-n` evita retornar outras linhas):

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | sed -n '/all_puppetclasses/,/]/p'
...
"all_puppetclasses":[
"id":<ID>
"name":"<Nome>"
"module_name":"<Módulo>"
"id":<ID>
"name":"<Nome>"
"module_name":"<Módulo>"
...
]
```

### Filtrar campos em trechos

Um problema é que há campos com nomes repetidos dentro dos campos aninhados. Por exemplo, há campos `id` no registro, e também dentro dos campos `all_puppetclasses` e `puppetclasses`. Então a filtragem com `grep` retornaria várias informações, sendo só uma correta. A solução é primeiro filtrar o trecho que contém os campos de forma única, depois os campos específicos

Neste exemplo, removemos tudo a partir de `all_puppetclasses`, usando o comando `sed d`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | \
	sed '/all_puppetclasses/,$d' | grep -E '"(id|name|title)"'
...
"id":<ID>
"name":"<Nome>"
"title":"<Título>"
```

> O segundo comando `sed` remove tudo a partir de `all_puppetclasses`. Para os _hostgroups_, isso significa remover todos os campos aninhados.

> Note a inclusão das aspas duplas no comando `grep`. Isto é necessário para filtrar campos com os nomes exatos, e não qualquer coisa que contenha "id", "name" ou "title".

Neste outro exemplo, selecionamos o campo aninhado `all_puppetclasses`, usando o comando `sed p`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | \
	sed -n '/all_puppetclasses/,/]/p' | grep '"name"'
...
"name":"<Nome da classe>"
"name":"<Nome da classe>"
"name":"<Nome da classe>"
...
```

### Registros múltiplos

Ao filtrar saídas com múltiplos registros, é interessante processar apenas a linha contendo `"results":`, com um simples `grep`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d "{ "search":"<Filtro>" }" 'https://<Servidor Foreman>/api/hostgroups/' | \
	grep '"results":' | sed ...
```

### Mais

Naturalmente, pode-se filtrar e formatar mais. Por exemplo: utilizar `cut` para remover o nome do parâmetro, deixando apenas o valor (caso a consulta seja de um único campo), outro `tr` para remover as aspas, etc. O limite é a criatividade (e o tempo).

## Filtragem e formatação com `awk`

Por embutir uma linguagem de programação razoavelmente decente e uma boa quantidade de opções, o `awk` permite uma variedade quase ilimitada de opções para processamento da saída. Recomendo a [documentação GNU](https://www.gnu.org/software/gawk/manual/gawk.html) para o `awk` e um pouco de estudo sobre [expressões regulares](http://aurelio.net/regex/).

### Separar campos em linhas

Para separar os campos por vírgulas e chaves, usamos a opção do delimitador de campos (`-F`), depois retornamos (`print`) os campos:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://<Servidor Foreman>/api/hostgroups/<ID> | \
	awk -F'[{},]+' '{
		for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) {
			print $CAMPO
		}
	}'
...
"id":<ID>
"name":"<Nome>"
"title":"<Título>"
...
"all_puppetclasses":[
"id":<ID>
"name":"<Classe>"
"module_name":"<Módulo>"
...
]
...
"puppetclasses":[
"id":<ID>
"name":"<Classe>"
"module_name":"<Módulo>"
...
]
"config_groups":[]
```

### Selecionar campos

Para selecionar campos específicos, basta usar o `if`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://<Servidor Foreman>/api/hostgroups/<ID> | \
	awk -F'[{},]+' '{
		for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) {
			if ( $CAMPO ~ /"(title|environment_name)"/ ) {
				print $CAMPO
			}
		}
	}'
...
"title":"<Título>"
"environment_name":<Ambiente>
```

> Importante incluir as aspas na expressão regular - `"(title|environment_name)"` - para não retornar parâmetros com nomes parecidos

### Remover nome do parâmetro

Se só estivermos interessados em um campo, podemos excluir a parte `"<Parâmetro>":`, e até as aspas, usando o substituidor `gsub`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://<Servidor Foreman>/api/hostgroups/<ID> | \
	awk -F'[{},]+' '{
		for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) {
			if ( $CAMPO ~ /"title"/ ) {
				gsub("^[^:]+:|\"", "", $CAMPO)
				print $CAMPO
			}
		}
	}'
...
<Título>
```

### Registros múltiplos

Assim como na filtragem sem `awk`, ao filtrar saídas com múltiplos registros, pode ser interessante processar apenas a linha contendo `"results":`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d "{ "search":"<Filtro>" }" https://<Servidor Foreman>/api/hostgroups/ | \
	awk -F'[{},]+' '/"results":/ { for ... }'
```
