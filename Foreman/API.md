# API Foreman

#### Índice

1. [Descrição](#descrição)
1. [Referências](#referências)
1. [Utilização](#utilização)
	1. [Sintaxe do comando `curl`](#sintaxe-do-comando-curl)
	1. [Saída do comando `curl`](#saída-do-comando-curl)
		1. [Consultas a registro específico](#consultas-a-registro-específico)
		1. [Consultas a registros múltiplos via busca](#consultas-a-registros-múltiplos-via-busca)
1. [Parâmetros para consultas e alterações](#parâmetros-para-consultas-e-alterações)
	1. [Tipos de registro](#tipos-de-registro)
	1. [Campos principais](#campos-principais)
1. [Comandos da API](#comandos-da-api)
	1. [Consultar registros](#consultar-registros)
		1. [Todos os registros](#todos-os-registros)
		1. [Registros filtrados](#registros-filtrados)
		1. [Registro específico](#registro-específico)
	1. [Criar registros](#criar-registros)
	1. [Alterar parâmetros de registros](#alterar-parâmetros-de-registros)
	1. [Remover registros](#remover-registros)
1. [Formatação dos resultados](#formatação-dos-resultados)
	1. [Filtragem e formatação com `tr`, `sed`, `grep`, etc.](#filtragem-e-formatação-com-tr,-sed,-grep,-etc.)

## Descrição

A API do Foreman permite ler e escrever no banco de dados do Foreman via linha de comando, através do envio de comandos HTTP

## Referências

[https://theforeman.org/api/1.13/index.html](Referência API v2)  
[https://theforeman.org/2012/01/advance-search-tips-in-foreman.html](Advance Search tips in Foreman)  
[http://www.theforeman.org/manuals/1.5/#5.1API](Manual API v1)  
[http://www.theforeman.org/manuals/1.10/index.html#5.1API](Manual API v2)

## Utilização

As consultas e alterações via API são feitas através de requisições ao contexto `/api` do Foreman.

O programa `curl`, presente em muitas instalações Linux/Unix, é uma ótima ferramenta para acesso à API do Foreman

### Sintaxe do comando `curl`

```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" [<Comando HTTP>] <URL>
```

Opções:
- `-k` - Permite conexões inseguras com SSL - Útil para servidores Foreman com certificado auto-assinado - **Não confere validade do certificado**
- `-u <Usuário>` - Especifica o usuário para conexão à API (normalmente o LDAP)
- `-H <Cabeçalho>` - Envia cabeçalhos ao servidor Foreman
- Comando HTTP - `[-X <Método> [-d <Parâmetros>]]`
	- `GET -d '{ "search":"<Campo>~<String>%" }'` - Lista registros com campo contendo string
	- `POST -d '{ "<Parâmetro>":"<Valor>","<Parâmetro>":"<Valor>"... }'` - Cria registro(s)
	- `PUT -d '{ "<Parâmetro>":"<Valor>","<Parâmetro>":"<Valor>"... }'` - Altera registro(s)
	- `DELETE` - Remove registro(s)
	- default: lista o registro indicado na URL (campo e valor obrigatórios):
		- `https://<Servidor Foreman>/api/<Campo>/<Valor>`
- URL - Servidor Foreman - `https://<Servidor Foreman>/api/<Campo>[?<Opções>|<Valor>]`
	- Campo - O tipo de registro que será lido ou escrito - `hosts`, `domains`, `hostgroups`, etc.
	- Opções - Parâmetros de busca ou exibição - `per_page=<Número>`, `page=<Número>`, etc.
	- Valor - ID de um registro (host, domínio, etc.) específico
		- Também é possível especificar o nome, mas isto é problemático - Ver bugs:
			- [API find_resource by name finds by ID instead when first character of name is a digit](http://projects.theforeman.org/issues/3876)
			- [Info commands must always use IDs instead of names in api calls](http://projects.theforeman.org/issues/3954)
		- Para nomes de _hosts_, é possível usar o FQDN neste campo, desde que o mesmo inicie por letra - Usar o FQDN, não o nome curto

### Saída do comando `curl`

A API retorna uma string JSON com os registros especificados ou encontrados, e seus parâmetros

#### Consultas a registro específico

Ao especificar o registro na URL (campo/valor), o comando retorna os resultados em uma linha:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://<Servidor Foreman>/api/hosts/<FQDN>
Enter host password for user '<Usuário>':
{"name":"<FQDN>","id":<ID>,"ip":"<IP primário>",...}
```

#### Consultas a registros múltiplos via busca

Ao especificar uma string de busca via HTTP GET, o comando retorna um registro JSON com as estatísticas da busca e outro com os resultados em si, também em uma linha:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d '{ "search":"name~<Parte do nome>%" }' \
	https://<Servidor Foreman>/api/hostgroups
Enter host password for user '<Usuário>':
{
  "total": 1388,	# Total de registros existentes no servidor
  "subtotal": 12, # Número de registros que correspondem à busca
  "page": 1,
  "per_page": 50,
  "search": "name~<Parte do nome>%",
  "sort": {
    "by": null,
    "order": null
  },
  "results": [{"id":<ID 1>,"name":"<Nome 1>","title":"<Nome completo 1>",...},{"id":<ID 2>,"name":"
<Nome 2>","title":"<Nome completo 2>",...},{"id":<ID 3>,"name":"<Nome 3>","title":"<Nome completo 3
>",...},...]
}
```

## Parâmetros para consultas e alterações

Segue abaixo uma lista, não exaustiva, dos recursos que podem ser consultados e alterados via API do Foreman

### Tipos de registro

- architectures
- domains
- environments
- hostgroups
- hosts
- operatingsystems
- realms
- reports
- puppetclasses

### Campos principais

- Comuns a todos os registros
	- id					# Índice do registro (referenciável na URL)
- Hosts
	- name				# FQDN do servidor (referenciável na URL)
	- environment_id		# ID do ambiente
	- environment_name	# Nome do ambiente
	- last_report			# Hora UTC do último relatório de execução do agente Puppet
	- domain_id			# ID do domínio
	- domain_name			# Nome do domínio
	- hostgroup_id		# ID do grupo
	- hostgroup_name		# Nome do grupo
	- puppet_ca_proxy_id	# ID do Puppet CA
	- puppet_proxy_id		# ID do Puppet Master
	- certname			# Nome retornado pelo agente Puppet - Deve ser idêntico ao campo "name"
	- all_puppetclasses	# Conjunto das classes Puppets aplicadas ao nó
	- parameters			# Parâmetros aplicados às classes Puppet
- Hostgroups
	- name				# Nome do grupo (somente o último nível)
	- title				# Nome completo do grupo (todos os níveis)
	- ancestry			# Ascendência do grupo - IDs de todos os ascendentes separados por - barra
	- puppet_proxy_id		# ID do Puppet Master
	- puppet_ca_proxy_id	# ID do Puppet CA
- Opções (não exaustivo):
	- per_page
	- page

## Comandos da API

### Consultar registros

A consulta a registros mostra todos os parâmetros de um conjunto de registros. Podemos visualizar todos os registros de um determinado tipo, selecionar um grupo de registros mediante busca, ou determinar um registro específico.

#### Todos os registros

Basta especificar o tipo de registro no final da URL:
```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://<Servidor Foreman>/api/<Tipo>[?<Opção>[&<Opção>...]]
```

#### Registros filtrados

Especificar o filtro no comando HTTP, mantendo a mesma URL:
```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d '{ "search":"<String busca>" }' \
	https://<Servidor Foreman>/api/<Tipo>[?<Opção>[&<Opção>...]]
```

A string de busca tem o formato `<Campo>=<Valor>` ou `<Campo>~<Parte do valor>`.

#### Registro específico

Especificar o ID do registro (ou o nome, caso não inicie com dígito e não tenha caracteres especiais, espaços, etc.) no final da URL:
```
https://<Servidor Foreman>/api/<Tipo>/<ID>[?<Opção>[&<Opção>...]]
```

### Criar registros

A criação de registros é feita através do método HTTP POST:
```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
-X POST -d '{ "<Parâmetro 1>":"<Valor 1>","<Parâmetro 2>":"<Valor 2>",... }' \
https://<Servidor Foreman>/api/<Tipo>
```

> Verificar os parâmetros obrigatórios do tipo de recurso a ser criado

### Alterar parâmetros de registros

A alteração de parâmetros é feita através do método HTTP PUT:
```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
-X PUT -d '{ "<Parâmetro 1>":"<Valor 1>","<Parâmetro 2>":"<Valor 2>",... }' \
https://<Servidor Foreman>/api/<Tipo>/<ID>
```

> Usar os [comandos de consulta](#registros-filtrados) para verificar o ID do recurso a ser alterado

### Remover registros

A remoção de registros é feita através do método HTTP DELETE:

```
curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
-X DELETE https://<Servidor Foreman>/api/<Tipo>/<ID>
```

> Usar os [comandos de consulta](#registros-filtrados) para verificar o ID do recurso a ser alterado

## Formatação de resultados

Como a API retorna uma string enorme com todos os resultados concatenados, é difícil extrair informações diretamente, especialmente em buscas que retornam vários registros. É necessário formatar a saída de forma mais conveniente.

A complexidade da tarefa de transformar uma linha de texto JSON em uma tabela compreensível exige um processador de textos poderoso. Uma solução simples é concatenar, via _pipes_, vários processadores, como `tr`, `sed` e `grep`. Para formatação mais completa, é possível usar o `awk`, que possui inúmeros recursos de tratamento de strings aliados a lógica de programação.

Em qualquer caso, é bom ter em mente como a saída da API é formatada:

- Registros e seus campos são sempre separados por vírgulas
- Grupos de registros e de campos aninhados ficam entre colchetes
- Cada registro individual e cada campo dentro de campos aninhados fica entre chaves
- Ao listar um único registro, a API retorna uma única linha com o registro (entre chaves)
- Ao listar vários registros, a API retorna uma tabela com estatísticas, depois uma única linha com todos os registros (entre colchetes)
- Cada campo tem o formato `"<Parâmetro>":"<Valor>"`

Abaixo um exemplo de saída com registro único. Note como o registro inteiro está entre chaves, e cada campo separado por vírgulas. No caso dos _hostgroups_, os campos `all_puppetclasses`, `puppetclasses` e `config_groups` contém vários campos aninhados - todos entre chaves e separados por vírgulas, e o conjunto inteiro entre colchetes. O campo `config_groups` também contém um campo aninhado - `puppetclasses` (que não é o mesmo `puppetclasses` anterior)- que, por sua vez, contém seus próprios campos aninhados, com a mesma formatação.

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

### Filtragem e formatação com `tr`, `sed`, `grep`, etc.

Uma necessidade natural é separar os campos com quebras de linhas, em vez de vírgulas. Isto pode ser feito facilmente com o comando `tr`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | tr , '\n'
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
Como cada campo está delimitado por chaves, podemos separar as linhas em cada conjunto de vírgulas e/ou chaves. Pra esta filtragem mais elaborada, podemos usar o comando `sed`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g'
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

Desta vez, os campos aninhados foram separados com clareza, ficando o campo externo em uma linha separada. Assim fica fácil filtrar campos específicos, usando `grep`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | grep 'environment_'
...
"environment_id":<ID ambiente>
"environment_name":<Nome ambiente>
```

Ou filtrar por trechos, usando `sed`:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | sed -n '/all_puppetclasses/,/]/p'
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

Um problema é que há campos com nomes repetidos dentro dos campos aninhados. Por exemplo, há campos `id` no registro, e também dentro dos campos `all_puppetclasses` e `puppetclasses`. Então a filtragem com `grep` retornaria várias informações, sendo só uma correta. A solução é primeiro filtrar o trecho que contém os campos de forma única, depois os campos específicos:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | sed '/all_puppetclasses/,$d' | grep -E '"(id|name|title)"'
...
"id":<ID>
"name":"<Nome>"
"title":"<Título>"
```

> O segundo comando `sed` remove tudo a partir de `all_puppetclasses`. Para os _hostgroups_, isso significa remover todos os campos aninhados.

> Note a inclusão das aspas duplas no comando `grep`. Isto é necessário para filtrar campos com os nomes exatos, e não qualquer coisa que contenha "id", "name" ou "title".

Outro exemplo:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" 'https://<Servidor Foreman>/api/hostgroups/<ID>' | sed 's/[,{}][,{}]*/\n/g' | sed -n '/all_puppetclasses/,/]/p' | grep '"name"'
...
"name":"<Nome da classe>"
"name":"<Nome da classe>"
"name":"<Nome da classe>"
...
```

Naturalmente, pode-se filtrar e formatar mais. Por exemplo: utilizar `cut` para remover o nome do parâmetro, deixando apenas o valor (caso a consulta seja de um único campo), outro `tr` para remover as aspas, etc. O limite é a criatividade (e o tempo).
