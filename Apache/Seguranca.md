# Dicas de segurança para configuração do servidor http Apache

#### Índice

1. [Introdução](#introdução)
1. [Usuário e grupo](#usuário-e-grupo)
  1. [Bloquear acesso ao _shell_](#bloquear-acesso-ao-shell)
  1. [Bloquear login local](#bloquear-login-local)
  1. [Bloquear login remoto](#bloquear-login-remoto)
  1. [_Softwares_ de _cache_ e _proxy_](#softwares-de-cache-e-proxy)
1. [Conteúdo](#conteúdo)

## Introdução

Servidores http executam processos com acesso ao sistema de arquivos e com porta(s) aberta(s) a _hosts_ remotos. Um invasor que comprometa um destes processos representa um risco crítico ao servidor http. Este artigo contém algumas dicas básicas para minimizar a vulnerabilidade a este tipo de problema.

## Usuário e grupo

Em uma instalação padrão, os processos do Apache rodam com o usuário `www-data` no Debian e `apache` no RedHat, com grupo do mesmo nome. Normalmente, este usuário não precisa fazer qualquer alteração no sistema (exceto no conteúdo de alguns serviços, como blogs).

Então, uma boa medida de segurança é restringir ao máximo as permissões deste usuário. Os procedimentos abaixo impedem que o usuário do Apache abra um _shell_ ou faça login, tanto remoto quanto local. Assim, um processo Apache corrompido dificilmente conseguirá adulterar o sistema.

### Bloquear acesso ao _shell_

No arquivo `/etc/passwd`, procurar o usuário do Apache e mudar o seu _shell_ padrão (último campo) para `/sbin/nologin` ou `/usr/sbin/nologin` (verificar a localização correta com `which nologin`, executado como `root`). Isto impede que o usuário possa abrir um _shell_.

> Na falta do `nologin`, é possível utilizar `/bin/false` ou até `/dev/null` como _shell_ padrão. Usar preferencialmente o `nologin`, pois este é feito pra isso.

### Bloquear login local

Nos arquivos `/etc/shadow` e `/etc/gshadow`, remover linhas que contenham este usuário/grupo. Isto desabilita o login do usuário.

### Bloquear login remoto

No arquivo `/etc/security/access.conf`, remover autorização de acesso a este usuário/grupo. Isto impede acesso remoto (como SSH).

### _Softwares_ de _cache_ e _proxy_

Algumas ferramentas de _cache_ e _proxy_, como o [Varnish](http://varnish-cache.org/) e o [Pound](http://www.apsis.ch/pound), executam seus processos com usuário e grupo extremamente restrito, como `nobody`, ou outro com acesso exclusivo aos arquivos da própria ferramenta. É imprescindível manter estas configurações desta forma.

Quando estas ferramentas são utilizadas, são elas que ficam com portas abertas para _hosts_ remotos. Em caso de comprometimento, a utilização deste usuário/grupo restrito minimiza a vulnerabilidade do servidor.

Caso seja inevitável alterar, utilizar usuário/grupo também fortemente restrito. Uma **péssima** ideia é utilizar o usuário/grupo do Apache - Um comprometimento da ferramenta torna **todo** o conteúdo Web visível - inclusive código fonte.

## Conteúdo

Com exceção de alguns serviços que permitem ao usuário alterar o conteúdo Web (blogs, por exemplo), o servidor http deve somente ler, e nunca alterar, os arquivos de conteúdo. Portanto, a recomendação é manter estes arquivos, e seus diretórios, com permissão apenas de leitura para o usuário/grupo do Apache, e nenhuma outra permissão para qualquer outro usuário, exceto `root`.

Uma opção é manter o caminho completo pertencente a `root:root` e as permissões de escrita apenas para o usuárion `root`:

```
$ ls -ld /{,var{,/www{,/html{,/*}}}}
drwxr-xr-x. 21 root   root      4096 Jan 15 18:20 /var
drwxr-xr-x   6 root   root      4096 Jan 15 18:20 /var/www
drwxr-xr-x   3 root   root      4096 Jan 15 18:20 /var/www/html
drwxr-xr-x   3 root   root      4096 Jan 15 18:20 /var/www/html/<vhost>
-rw-r--r--   5 root   root      4096 Jan 15 18:20 /var/www/html/<vhost>/index.html
...
```

Uma alternativa, mais restrita, é atribuir o conteúdo Web ao grupo do Apache e permitir a leitura apenas a este grupo. O acesso pelo processo do Apache continua o mesmo, mas outros usuários não podem ler o conteúdo Web (substituir `apache` pelo usuário padrão da distro):

```
$ ls -ld /{,var{,/www{,/html{,/*}}}}
drwxr-xr-x. 21 root   root      4096 Jan 15 18:20 /var
drwxr-xr-x   6 root   root      4096 Jan 15 18:20 /var/www
drwxr-x---   3 apache apache    4096 Jan 15 18:20 /var/www/html
drwxr-x---   3 apache apache    4096 Jan 15 18:20 /var/www/html/<vhost>
-rw-r-----   5 apache apache    4096 Jan 15 18:20 /var/www/html/<vhost>/index.html
...
```

> Em servidores Debian < 8, substituir `/var/www/html` por `/var/www`
