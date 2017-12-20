# API Foreman

#### Índice

1. [Descrição](#descrição)
1. [Referências](#referências)
1. [Utilização](#utilização)
	1. [Sintaxe do comando `curl`](#sintaxe-do-comando-curl)

## Descrição

A API do Foreman permite ler e escrever no banco de dados do Foreman via linha de comando, através do envio de comandos HTTP

## Referências

[https://theforeman.org/api/1.13/index.html](Referência API v2)  
[https://theforeman.org/2012/01/advance-search-tips-in-foreman.html](Advance Search tips in Foreman)  
[http://www.theforeman.org/manuals/1.5/#5.1API](Manual API v1)  
[http://www.theforeman.org/manuals/1.10/index.html#5.1API](Manual API v2)

## Utilização

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
- URL - Servidor Foreman - `https://<Servidor Foreman>/<Campo>[?<Opções>|<Valor>]`
	- Campo - O tipo de registro que será lido ou escrito - `hosts`, `domains`, `hostgroups`, etc.
	- Opções - Parâmetros de busca ou exibição - `per_page=<Número>`, `page=<Número>`, etc.
	- Valor - ID de um registro (host, domínio, etc.) específico
		- Também é possível especificar o nome, mas isto é problemático - Ver bugs:
			- [API find_resource by name finds by ID instead when first character of name is a digit](http://projects.theforeman.org/issues/3876)
			- [Info commands must always use IDs instead of names in api calls](http://projects.theforeman.org/issues/3954)
