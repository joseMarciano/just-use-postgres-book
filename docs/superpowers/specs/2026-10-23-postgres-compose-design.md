# PostgreSQL Compose local — design

## Objetivo

Adicionar ao repositório uma configuração simples para executar PostgreSQL localmente com Docker Compose. O container deve poder ser removido sem perder os dados armazenados no diretório local do projeto.

## Configuração proposta

- Criar `compose.yaml` na raiz com um único serviço PostgreSQL baseado em `postgres:17.2`.
- Definir usuário, senha e banco inicial como `local`.
- Publicar a porta do container `5432` como porta `5402` do host, para conexões em `localhost:5402`.
- Usar `network_mode: bridge`, conectando o serviço à rede bridge padrão do Docker (não host e não uma rede isolada criada pelo Compose).
- Montar `./shared-container-volume/` em `/var/lib/postgresql/data` como bind mount. Os dados permanecem no repositório quando o container é removido; apagar essa pasta também apagará os dados.
- Adicionar `shared-container-volume/` ao `.gitignore` para não versionar os arquivos de dados do banco.

## Fora de escopo

Não adicionar extensões, serviços auxiliares, credenciais via arquivo `.env`, rede personalizada, volume nomeado ou instruções adicionais ao README.

## Verificação e critérios de aceite

1. `docker compose config` valida o arquivo Compose.
2. Com Docker disponível e a porta `5402` livre, `docker compose up -d` inicia o PostgreSQL.
3. O serviço aceita conexão em `localhost:5402` usando banco/usuário/senha `local`.
4. Remover o container e iniciá-lo novamente preserva os dados em `./shared-container-volume/`.
5. O diretório de dados é ignorado pelo Git.
