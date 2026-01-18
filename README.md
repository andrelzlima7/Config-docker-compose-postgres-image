# PostgreSQL com Docker Compose

Guia completo para configurar uma instância do PostgreSQL usando Docker Compose com volume para persistência de dados.

## Pré-requisitos

- Docker instalado
- Docker Compose instalado

### Instalação do Docker no Ubuntu/Debian

```bash
# Instalar Docker via snap
sudo snap install docker

# Ou via apt
sudo apt update
sudo apt install docker.io docker-compose
```

## Estrutura do Projeto

```
projeto/
├── docker-compose.yml
└── Documentos/Docker/Postgres  # Volume para persistência (criado automaticamente)
```

## Configuração

### 1. Criar o arquivo docker-compose.yml

Crie um arquivo `docker-compose.yml` com o seguinte conteúdo:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: meu_banco
    ports:
      - "5432:5432"
    volumes:
      - ~/Documentos/Docker/Postgres:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

### 2. Personalizar as variáveis de ambiente

Edite as seguintes variáveis conforme necessário:

- `POSTGRES_USER`: Nome do usuário do banco (padrão: postgres)
- `POSTGRES_PASSWORD`: Senha do banco ⚠️ **Altere em produção!**
- `POSTGRES_DB`: Nome do banco de dados inicial

### 3. Usar arquivo .env (Recomendado para segurança)

Crie um arquivo `.env` na mesma pasta:

```env
POSTGRES_USER=postgres
POSTGRES_PASSWORD=sua_senha_segura
POSTGRES_DB=meu_banco
POSTGRES_PORT=5432
```

E atualize o `docker-compose.yml`:

```yaml
services:
  postgres:
    image: postgres:16-alpine
    container_name: postgres-db
    restart: always
    environment:
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
      POSTGRES_DB: ${POSTGRES_DB}
    ports:
      - "${POSTGRES_PORT}:5432"
    volumes:
      - ./dados:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${POSTGRES_USER}"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## Comandos Úteis

### Iniciar o container

```bash
docker-compose up -d
```

### Parar o container

```bash
docker-compose down
```

### Ver logs

```bash
docker-compose logs -f postgres
```

### Verificar status

```bash
docker-compose ps
```

### Reiniciar o container

```bash
docker-compose restart
```

### Parar e remover volumes (⚠️ APAGA OS DADOS)

```bash
docker-compose down -v
```

## Acessar o PostgreSQL

### Via linha de comando (psql)

```bash
docker exec -it postgres-db psql -U postgres -d meu_banco
```

### Via cliente externo (DBeaver, pgAdmin, etc.)

**Configurações de conexão:**

- **Host:** localhost
- **Porta:** 5432
- **Usuário:** postgres (ou o definido no .env)
- **Senha:** postgres (ou a definida no .env)
- **Database:** meu_banco (ou o definido no .env)

### Porta 5432 já em uso

Se você já tem PostgreSQL instalado localmente:

```bash
# Parar o serviço local
sudo systemctl stop postgresql

# Ou mudar a porta no docker-compose.yml
ports:
  - "5433:5432"  # Usar porta 5433 externamente
```

### Permissões de volume

Se tiver problemas de permissão:

```bash
sudo chown -R 999:999 ./dados
```

### Container não inicia

Verificar logs:

```bash
docker-compose logs postgres
```

### Resetar completamente

```bash
docker-compose down -v
rm -rf dados/
docker-compose up -d
```

## Recursos Adicionais

- [Documentação oficial PostgreSQL](https://www.postgresql.org/docs/)
- [PostgreSQL no Docker Hub](https://hub.docker.com/_/postgres)
- [Docker Compose Documentation](https://docs.docker.com/compose/)

## Licença

Este guia é de domínio público. Use livremente!
