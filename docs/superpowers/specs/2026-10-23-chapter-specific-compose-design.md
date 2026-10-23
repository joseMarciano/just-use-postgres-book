# Compose services por capítulo — design

## Objetivo

Organizar em um único `compose.yaml` os serviços de banco dos capítulos, com nomes e armazenamento independentes, ativando somente o capítulo selecionado. Esta alteração implementa apenas o serviço do capítulo 4; serviços de outros capítulos ficam para solicitações futuras.

## Contexto observado

- O Compose atual tem um serviço `postgres`, com imagem `postgres:17.2`, role/banco `local`, senha `local`, porta do host `5402` e bind mount em `./shared-container-volume/`.
- Os comandos de `chapter04.md` foram adaptados para usar o Compose, mas referenciam o serviço `postgres`.
- O diretório atual `shared-container-volume/` contém o banco já carregado, inclusive as 10.000 linhas de `game.player_stats`.
- O container atual ocupa a porta `5402`.

## Desenho

- Renomear o identificador do serviço para `postgres-chapter-04` e atribuir o profile `chapter-04`.
- Não definir `container_name`: comandos Compose usam o identificador do serviço, enquanto o nome físico do container pode incluir o nome do projeto e um índice.
- Manter imagem `postgres:17.2`, `network_mode: bridge`, role/senha/banco `local` e porta `5402:5432`.
- Usar o novo bind mount `./shared-chapter-04-volume/` em `/var/lib/postgresql/data`.
- Preservar `shared-container-volume/` sem copiá-lo ou apagá-lo; ele permanece ignorado no Git como fallback dos dados existentes.
- Ignorar também diretórios `shared-chapter-*-volume/`, para que volumes de capítulos futuros não sejam versionados.
- Cada serviço futuro deverá ter um identificador, profile e diretório de dados próprios. Serviços de outros capítulos não fazem parte desta alteração.

## Uso do capítulo 4

O início passa a selecionar o profile e o serviço explicitamente:

```sh
docker-compose --profile chapter-04 up -d postgres-chapter-04
```

A seção “Preload sample dataset” de `chapter04.md` copiará e executará os arquivos através do serviço Compose, usando `-U local -d local`. `docker-compose up -d` sem profile não ativa os serviços dos capítulos; a seleção explícita evita iniciar todos quando o arquivo ganhar serviços futuros.

## Troca do serviço e dados

- Parar e remover somente o container antigo do serviço `postgres` para liberar a porta `5402`.
- Não remover nem alterar o diretório antigo `shared-container-volume/`.
- O novo volume começa com um banco vazio; executar o preload do capítulo 4 carrega os arquivos SQL nele.
- O preload é uma operação inicial; não deve ser repetido depois que o banco persistente já contém o schema e os registros.

## Verificação e critérios de aceite

1. `docker-compose config` valida a configuração e o profile do serviço.
2. `docker-compose --profile chapter-04 up -d postgres-chapter-04` inicia somente o serviço do capítulo 4.
3. O serviço atende em `localhost:5402`, usa `network_mode: bridge` e monta `./shared-chapter-04-volume/` em `/var/lib/postgresql/data`.
4. Os comandos atualizados de preload copiam e executam `gaming_ddl.sql` e `gaming_data.sql` com sucesso; `game.player_stats` contém 10.000 linhas.
5. `shared-container-volume/` continua intacto e ambos os diretórios de volume são ignorados pelo Git.

## Fora de escopo

Não adicionar serviços de outros capítulos, alterar os comandos dos demais capítulos, migrar os dados atuais para o novo volume, remover o diretório antigo ou definir `container_name` fixo.
