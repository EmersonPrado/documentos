# Exemplos de utilização da API do Foreman

#### Índice

1. [Operações básicas](#operações-básicas)
	1. [Listar os domínios](#listar-os-domínios)
	1. [Listar um domínio específico](#listar-um-domínio-específico)
	1. [Listar os hosts](#listar-os-hosts)
	1. [Criar um host](#criar-um-host)
	1. [Alterar um host](#alterar-um-host)
	1. [Remover um host](#remover-um-host)
	1. [Procurar um host usando coringa](#procurar-um-host-usando-coringa)
	1. [Filtrar parâmetros da listagem de hosts](#filtrar-parâmetros-da-listagem-de-hosts)

## Operações básicas

### Listar os domínios

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" https://<Foreman>/api/domains | awk -F'[}{]' '/results/ { for ( ITEM = 1 ; ITEM < NF ; ITEM++ ) { print $ITEM } }' | grep id
Enter host password for user '<Usuário>':
...
"id":<ID>,"name":"<Nome>","fullname":null,"dns_id":null,"created_at":"<Criação>","updated_at":"<Atualização>"
"id":<ID>,"name":"<Nome>","fullname":null,"dns_id":null,"created_at":"<Criação>","updated_at":"<Atualização>"
"id":<ID>,"name":"<Nome>","fullname":null,"dns_id":null,"created_at":"<Criação>","updated_at":"<Atualização>"
...
```

### Listar um domínio específico

#### Especificando ID

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" https://<Foreman>/api/domains/<ID>
Enter host password for user '<Usuário>':
{"id":<ID>,"name":"<Nome>","fullname":null,"dns_id":null,"created_at":"<Criação>","updated_at":"<Atualização>","subnets":[],"parameters":[]}
```

#### Especificando nome

```
$ curl -ku <Usuário> -H "Accept: version=2,ation/json" https://<Foreman>/api/domains/<Nome>
Enter host password for user '<Usuário>':
{"id":<ID>,"name":"<Nome>","fullname":null,"dns_id":null,"created_at":"<Criação>","updated_at":"<Atualização>","subnets":[],"parameters":[]}
```

### Listar os hosts

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" https://<Foreman>/api/hosts
Enter host password for user '<Usuário>':
{
  "total": <Total hosts>,
  "subtotal": <Hosts filtrados>,
  "page": <Número da página>,
  "per_page": <Hosts por página>,
  "search": null,
  "sort": {
    "by": null,
    "order": null
  },
  "results": [{"name":"<FQDN>","id":<ID>,"ip":"<IP>","environment_id":<ID ambiente>,"environment_name":"<Nome ambiente>","last_report":<Último relatório>,...},{"name":"<FQDN>","id":<ID>,"ip":"<IP>","environment_id":<ID ambiente>,"environment_name":"<Nome ambiente>","last_report":<Último relatório>,...},...]
```

### Criar um host

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X POST -d '{ "name":"<FQDN>","environment_name":"<Ambiente>",... }' https://<Foreman>/api/hosts
```

> Verificar parâmetros obrigatórios para hosts

### Alterar um host

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X PUT -d '{ "<Parâmetro>":"<Valor>" }' https://<Foreman>/api/hosts/<Nome>
```

### Remover um host

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X DELETE https://<Foreman>/api/hosts/<Nome>
```

### Procurar um host usando coringa

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X GET -d '{ "search":"name~<Nome>%" }' https://<Foreman>/api/hosts/ | awk '/results/'
```

### Filtrar parâmetros da listagem de hosts

```
$ <Comando para mostrar, listar ou procurar hosts> | awk -F'[{},]+' '/results/ { for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) { if ( $CAMPO ~ /\"<Parâmetro 1>\"|\"<Parâmetro 2>\"|.../ ) { print $CAMPO } } }'
```

Exemplo:

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" -X GET -d '{ "search":"name~<String busca>%" }' https://<Foreman>/api/hosts/ | awk -F'[{},]+' '/results/ { for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) { if ( $CAMPO ~ /\"name\"|\"certname\"|\"hostgroup_name\"|\"environment_name/ ) { print $CAMPO } } }'
```
