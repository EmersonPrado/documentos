# Arquitetura

1. Banco (catálogo)
  1. CDB (para bancos _multitenant_)
  1. PDB (para bancos _multitenant_)
1. _Schema_ (_namespace_ para objetos do banco)

  > No caso do SGBD Oracle, cada usuário corresponde a um _schema_

1. _Tablespace_ (área lógica de armazenamento de objetos)
  1. _Tablespace_ temporária (para dados de consultas em andamento)
1. _Datafiles_ (arquivos contendo os objetos)

# Arquivos

1. _Control files_ - Contêm dados essenciais do banco e as localizações dos demais arquivos
1. _Datafiles_ - Contêm os objetos
1. _Redo logfiles_ - Contêm o histórico de alteração dos objetos
  1. _Online_ - Onde a instância escreve as alterações. Arquivos são reutilizados periodicamente.
  1. _Archived_ - Cópias permanentes dos arquivos de _redo online_

    > Os arquivos _online_ são copiados assim que ficam cheios  
    > Cada arquivo _online_ só pode ser reutilizado após arquivamento com sucesso

> Estes arquivos são organizados em grupos, cada um contendo cópias idênticas do arquivo, chamadas membros. Usar vários membros por grupo em discos diferentes, para garantir recuperação em caso de perda de um deles. Redundância via armazenamento (RAID e outros) não é suficiente - a falha em um membro pode ser propagada para outros.

# Tipos de objetos
