# Fabric

#### Índice

1. [Descrição](#descrição)
1. [Referências](#referências)
  1. [Bugs](#bugs)

## Descrição

Fabric é uma extensão do Python desenvolvida para execução remota de comandos e scripts via SSH
Permite a execução de scripts Python simultaneamente em um conjunto de hosts remotos
Os scripts podem executar comandos ou scripts do shell nestes hosts
Permite execução como root usando sudo

## Referências

-	[Documentação](http://docs.fabfile.org/en/1.10/)
-	[Tutorial](http://docs.fabfile.org/en/1.10/tutorial.html)

### Bugs

- [Use env.password or env.prompts when using local() to run commands which require authentication](https://github.com/fabric/fabric/issues/1448)
- [Option for skipping auth failures in addition to connection ones](https://github.com/fabric/fabric/issues/533)
