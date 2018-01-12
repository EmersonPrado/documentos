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
1. [Alterações múltiplas](#alterações-múltiplas)
	1. [Configurar vários grupos no mesmo host](configurar-vários-grupos-no-mesmo-host)
1. [Operações avançadas](#operações-avançadas)
	1. [Verificar tempo (em segundos) da última sincronização de um host com o Foreman](#verificar-tempo-em-segundos-da-última-sincronização-de-um-host-com-o-foreman)

## Operações básicas

### Listar os domínios

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" https://<Foreman>/api/domains | \
	awk -F'[}{]' '/results/ { for ( ITEM = 1 ; ITEM < NF ; ITEM++ ) { print $ITEM } }' | grep id
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
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X POST -d '{ "name":"<FQDN>","environment_name":"<Ambiente>",... }' https://<Foreman>/api/hosts
```

> Verificar parâmetros obrigatórios para hosts

### Alterar um host

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X PUT -d '{ "<Parâmetro>":"<Valor>" }' https://<Foreman>/api/hosts/<Nome>
```

### Remover um host

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X DELETE https://<Foreman>/api/hosts/<Nome>
```

### Procurar um host usando coringa

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d '{ "search":"name~<Nome>%" }' https://<Foreman>/api/hosts/ | awk '/results/'
```

### Filtrar parâmetros da listagem de hosts

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X GET -d '{ "search":"name~<Nome>%" }' https://<Foreman>/api/hosts/ | \
	awk -F'[{},]+' '/results/ { for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) {
		if ( $CAMPO ~ /\"name\"|\"certname\"|\"hostgroup_name\"|\"environment_name/ ) { print $CAMPO }
	} }'
```

## Alterações múltiplas

### Configurar vários grupos no mesmo host

> http://serverfault.com/questions/629369/foreman-assign-mutiple-hosts-to-a-config-group

```
$ curl -ku <Usuário> -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	-X PUT -d '{"<Host>":{"config_group_names":["<Grupo>","<Grupo>"...]}}' \
	https://<Foreman>/api/hosts/<Host>
```

## Operações avançadas

### Verificar tempo (em segundos) da última sincronização de um host com o Foreman

```
$ { \
	date +"%s %z" | sed 's/\([+-]*\)\(..\)\(..\)$/\1\2 \1\3/'; \
	curl -ku emerson.prado -H "Accept: version=2,application/json" -H "Content-Type: application/json" \
	https://foreman.prj.configdtp/api/hosts/<FQDN> | \
		awk -F, '{ for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) { if ( $CAMPO ~ /last_report/ ) {
			gsub("\"last_report\":", "", $CAMPO) ; system("date +%s -d "$CAMPO"")
		} } }';\
	} | awk 'NF == 3 { printf "%s - %s * 3600 - %s * 60 - ", $1, $2, $3 } ; NF == 1 { print $0 }' | bc
```

Destrinchando:

```
$ {
    date +"%s %z" | \                                         	# Mostra a data em segundos desde o "epoch" (%s) e o fuso horário em minutos (%z)
      sed 's/\([+-]*\)\(..\)\(..\)$/\1\2 \1\3/'; \            	# Reformata o fuso horário - Separa horas de minutos e repete o sinal das horas nos minutos
    curl \
        -ku emerson.prado \
        -H "Accept: version=2,application/json" \
        -H "Content-Type: application/json" \
        https://foreman.prj.configdtp/api/hosts/<FQDN> | \    	# Lista os parâmetros do host especificado
      awk -F, '{																						  	# Separa parâmetros por vírgula
			awk -F, '{ for ( CAMPO = 1 ; CAMPO <= NF ; CAMPO++ ) {  	# Processa um campo por vez
        if ( $CAMPO ~ /last_report/ ) {                       	# Encontra o parâmetro "last_report" (horário da última sincronização)
          gsub("\"last_report\":", "", $CAMPO)                	# Remove o nome do parâmetro (deixa apenas o valor)
          system("date +%s -d "$CAMPO"") }                    	# Retorna o valor, em segundos desde o "epoch"
        }
      }';
	} | \
    awk '
      NF == 3 { printf "%s - %s * 3600 - %s * 60 - ", $1, $2, $3 }    # Corrige o fuso horário da primeira linha (horário atual) para UTC
      NF == 1 { print $0 }                                            # Subtrai a segunda linha (horário da última sincronização)
    ' | \
    bc                                                              	# Processa fórmula
```
