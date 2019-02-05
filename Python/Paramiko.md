# Paramiko

#### Índice

1. [Descrição](#descrição)
1. [Referências](#referências)
1. [Pré-requisitos](#pré-requisitos)
1. [Introdução](#introdução)
  1. [O protocolo SSH](#o-protocolo-ssh)
  1. [O módulo Paramiko](#o-módulo-paramiko)
1. [Preparação](#preparação)
  1. [Console Python](#console-python)
  1. [Módulos e variáveis](#módulos-e-variáveis)
1. [Conexão via objeto SSHClient](#conexão-via-objeto-sshclient)
  1. [Criar conexão](#conexão-sshclient)
  1. [Enviar comandos](#comandos-sshclient)
  1. [Encerrar conexão](#encerrar-sshclient)
1. [Conexão via objeto Transport](#conexão-via-objeto-transport)
  1. [Criar conexão](#conexão-transport)
  1. [Enviar comandos](#comandos-transport)
  1. [Encerrar conexão](#encerrar-transport)
1. [Invocação de _Shell_](#invocação-de-shell)
1. [Transferência de arquivos](#transferência-de-arquivos)

## Descrição

O Paramiko é uma API para programação com SSH no Python

## Referências

- [Documentação oficial](http://docs.paramiko.org/en/2.4/)
- [Creating multiple SSH connections at a time using Paramiko - Stack Overflow](http://stackoverflow.com/questions/3485428/creating-multiple-ssh-connections-at-a-time-using-paramiko)
- [Python and SSH: Paramiko shell](https://daanlenaerts.com/blog/2016/07/01/python-and-ssh-paramiko-shell/)
- [Secure Shell - Architecture - Wikipedia](https://en.wikipedia.org/wiki/Secure_Shell#Architecture)

## Pré-requisitos

- Python
- Paramiko
  - Debian - Instalar o pacote `python-paramiko` ou `python3-paramiko`, dependendo da versão do Python
  - RedHat - Instalar o pacote `python-paramiko`

## Introdução

### O protocolo SSH

O protocolo SSH estabelece um canal de comunicação criptografado com um _shell_ remoto, permitindo transferência segura de dados como comandos, resultados de execução e arquivos. Para que uma transferência via SSH aconteça, é necessário configurar 3 camadas distintas:

- Transporte - Gerencia a conexão entre os _hosts_ local e remoto, cuidando da autenticação de _hosts_, criptografia de dados e outras características da conexão de rede
- Autenticação - Gerencia a autenticação do cliente SSH, identificando credenciais de usuário e/ou chaves
- Conexão - Gerencia os **canais** estabelecidos para a transferência de dados

### O módulo Paramiko

O módulo Paramiko oferece as seguintes classes principais:
- [`paramiko.transport`](http://docs.paramiko.org/en/2.4/api/transport.html) - Gerencia transporte e autenticação, permitindo conectar a _hosts_ remotos e autenticar
- [`paramiko.channel`](http://docs.paramiko.org/en/2.4/api/channel.html) - Gerencia canais em conexões já estabelecidas entre _hosts_, permitindo enviar comandos, receber a saída destes, abrir _shells_ para execução interativa e transferir arquivos via SFTP
- [`paramiko.client`](http://docs.paramiko.org/en/2.4/api/client.html) - Classe _wrapper_ que simplifica o gerenciamento das três camadas, dando acesso a objetos das classes `paramiko.transport` e `paramiko.channel`

Criar uma conexão consiste em configurar as 3 camadas citadas acima. Para isso, há 3 caminhos distintos:

1. Criar um objeto da classe `paramiko.client.SSHClient`, configurá-lo e executar a conexão. Este modo é bem mais simples, mas também limitado.
1. Criar um objeto da classe `paramiko.transport.Transport`, configurá-lo, executar a conexão e abrir um canal. Este modo é mais trabalhoso, mas oferece mais controle sobre a sessão.
1. Criar um objeto da classe `paramiko.client.SSHClient`, configurá-lo, executar a conexão e invocar um _shell_. Este modo permite envio de vários comandos e interação com os processos na mesma sessão.

## Preparação

### Console Python

Para executar os comandos aqui descritos, é necessário criar _scripts_ Python ou entrar no **console** do Python:

```shell
$ python
<Informações de versão e ajuda>
>>>
```

Para sair do console do Python, usar a função `quit()`:

```python
>>> quit()
$
```

### Módulos e variáveis

O mínimo necessário para utilizar o módulo paramiko é incluí-lo:

```python
import paramiko
```

Além disso, é necessário ter o nome de usuário, a senha, e a lista de _hosts_. O _mapping_ `os.environ` e a função `getpass.getpass()` podem ajudar a coletar o usuário e a senha:

```python
from os import environ
from getpass import getpass
usuario = environ['USER']
senha = getpass('Favor digitar a senha do usuário {}: '.format(usuario))
host = '<host>'
```

## Conexão via objeto SSHClient

### <a name="conexão-sshclient"></a>Criar conexão

O objeto `paramiko.SSHClient` já configura o transporte, a autenticação e o canal. Criar e armazenar o objeto:

```python
canal = paramiko.SSHClient()
```

Como as conexões SSH normalmente verificam chaves, configure o cliente para carregar as chaves já existentes e auto-adicionar as faltantes:

```python
canal.load_system_host_keys()
canal.set_missing_host_key_policy(paramiko.AutoAddPolicy())
```

Com isto, está tudo preparado para abrirmos uma conexão:

```python
canal.connect(host, username=usuario, password=senha)
```

Neste ponto, já é possível enviar comandos, tratar a saída destes e fazer transferências via SFTP.

### <a name="comandos-sshclient"></a>Enviar comandos

O envio de comandos à sessão aberta se dá com a função `exec_command`, que abre uma sessão e retorna um _tuple_ com 3 objetos `file`: a entrada padrão, a saída padrão e a saída de erros. Atribur estes objetos a variáveis para controlar a comunicação com o processo:

```python
stdin, stdout, stderr = canal.exec_command('uname -n')
```

Para verificar a saída, basta ler a variável `stdout`:

> Caso o processo esteja esperando alguma entrada, envie os dados (ver abaixo) antes de verificar a saída - Caso contrário, a tentativa de ler `stdout` congela a sessão Python

```python
stdout.read()
'<host>\n'
stdout.read()
''
```

> Recomendável também formatar o resultado, usando funções como `split`, `strip` e `format`

> Verificar outros métodos de leitura do objeto `file` do Python, como `readline()` e `readlines()`

A conexão aberta permite o envio de quantos comandos forem necessários:

```python
>>> stdin, stdout, stderr = canal.exec_command('whoami')
>>> stdout.read()
'<usuário>\n'
>>> stdin, stdout, stderr = canal.exec_command('ls')
>>> stdout.read()
'<arquivo>\n<arquivo>\n...'
```

Caso algum comando exija alguma entrada, enviar strings à entrada padrão com o método `write()`:

```python
stdin, stdout, stderr = canal.exec_command('read VAR ; echo $VAR')
stdin.write('A\n')
stdin.flush()
stdout.read()
'A\n'
```

> Note que é **necessário** incluir a quebra de linha ao enviar a string - Caso contrário, qualquer tentativa de ler `stdout` congela a sessão Python

> Não confundir a quebra de linha enviada com a quebra de linha retornada em `stdout`

Importante: cada envio de comando é uma sessão distinta de _shell_, portanto não preserva resultados de execuções anteriores:

```python
stdin, stdout, stderr = canal.exec_command('read VAR')
stdin.write('A\n')
stdin.flush()
stdin, stdout, stderr = canal.exec_command('echo $VAR')
stdout.read()
'\n'
```

### <a name="encerrar-sshclient"></a>Encerrar conexão

Ao final, é necessário fechar o conjunto transporte/conexão/canal:

```python
canal.close()
```

## Conexão via objeto Transport

### <a name="conexão-transport"></a>Criar conexão

Para criar uma sessão com mais possibilidades de controle, o primeiro passo é criar o transporte - um objeto `paramiko.transport.Transport` e configurar a autenticação:

```python
transporte = paramiko.transport.Transport(host)
transporte.connect(username=usuario, password=senha)
```

Para confirmar que a conexão está ativa e a autenticação foi bem sucedida:

```python
transporte.is_active()
True
transporte.is_authenticated()
True
```

Com a conexão ativa, é necessário criar um canal - recomendável verificar o ID da sessão e confirmar a abertura:

```python
canal = transporte.open_session()
canal.get_id()
1
canal.closed
False
```

### <a name="comandos-transport"></a>Enviar comandos

Neste ponto, já é possível enviar um comando à sessão aberta:

```python
canal.exec_command('uname -n')
```

Para coletar os resultados, é necessário verificar a disponibilidade das saídas - saída padrão, saída de erro e código de retorno, através das funções `ready`:

```python
canal.recv_ready()
True
canal.recv_stderr_ready()
False
canal.exit_status_ready()
True
```

Pelas saídas acima, verificamos que há dados na saída padrão e um código de retorno, indicando que a execução foi concluída. Verificar estas saídas através das funções `recv`, até que não haja mais dados na saída padrão e a sessão seja automaticamente fechada:

```python
canal.recv(1024)
'<host>\n'
canal.recv_ready()
False
canal.recv_exit_status()
0
canal.closed
True
```

> A função `recv` pede o número máximo de bytes como parâmetro

> A função exit_status_ready() continua retornando `True` mesmo com a sessão fechada - o código de retorno da execução continua disponível

Como a sessão está fechada, caso seja necessário enviar mais comandos, é necessário abrir outra:

```python
canal.exec_command('whoami')
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python2.7/dist-packages/paramiko/channel.py", line 59, in _check
    raise SSHException('Channel is not open')
paramiko.ssh_exception.SSHException: Channel is not open
canal = transporte.open_session()
canal.get_id()
2
canal.closed
False
canal.exec_command('whoami')
canal.recv_ready(), canal.recv_stderr_ready(), canal.exit_status_ready()
(True, False, True)
canal.recv(1024)
'<usuário>\n'
canal.recv_ready()
False
canal.recv_exit_status()
0
canal.closed
True
```

> Note a forma compactada de execução das funções `canal.recv_ready()`, `canal.recv_stderr_ready()` e `canal.exit_status_ready()`

### <a name="encerrar-transport"></a>Encerrar conexão

Quando o trabalho estiver concluído, é necessário encerrar o canal e a conexão:

```python
if canal.closed == False:
...   canal.close()
transporte.is_active(), transporte.is_authenticated()
(True, True)
transporte.close()
transporte.is_active(), transporte.is_authenticated()
(False, False)
```

## Invocação de _Shell_

A criação de um canal com os procedimentos acima faz com que, para cada linha de comando enviada, uma nova sessão seja criada e encerrada. Isto impede a execução de comandos que exijam configuração de variáveis, usuários e outros estados. Para obter o pleno controle sobre a sessão, é necessário criar uma sessão de _shell_, através dos seguintes passos:

1. [Prepare o ambiente](#preparação) do Python
1. [Crie um canal](#conexão-sshclient) usando o objeto paramiko.SSHClient
1. Crie um _shell_ no objeto criado:

    ```python
    shell = canal.invoke_shell()
    ```

1. Caso haja conteúdo na saída padrão ou de erro (provavelmente _banners_ e _prompt_), é recomendável descartar

    ```python
    shell.recv_ready(), shell.recv_stderr_ready(), shell.exit_status_ready()
    (True, False, False)
    shell.recv(1024)
    '<Linha banner>\r\n<Linha banner>\r\n<Linha banner>\r\n...<Prompt>'
    ```

1. [Envie comandos](#comandos-transport) da forma descrita para o objeto Transport:

    ```python
    shell.send('read VAR\n')
    9
    shell.send('A\n')
    2
    shell.recv(1024) # Somente para descartar o conteúdo do _buffer_, facilitando a visualização do resultado do próximo comando
    'read VAR\r\nA\r\n<Prompt>'
    shell.send('echo $VAR\n')
    10
    shell.recv(1024)
    'echo $VAR\r\nA\r\n<Prompt>' # Veja que o valor da variável foi mantido
    ```

1. Ao final do trabalho, encerre o _shell_ e o canal:
    1. Fechar _shell_:

        ```python
        >>> shell.send('exit\n')
        5
        ```

    1. Verificar saída e códigos de execução:

        ```python
        >>> shell.recv_ready(), shell.recv_stderr_ready(), shell.exit_status_ready()
        (True, False, True)
        >>> shell.recv(1024)
        'exit\r\nlogout\r\n'
        >>> shell.recv_ready(), shell.recv_stderr_ready(), shell.exit_status_ready()
        (False, False, True)
        >>> shell.recv_exit_status()
        0
        ```

    1. Confirmar encerramento do _shell_:

        ```python
        >>> shell.closed
        True
        ```

    1. Fechar canal:

        ```python
        >>> canal.close()
        ```

### Formatação da saída

### Exemplo

## Transferência de arquivos
