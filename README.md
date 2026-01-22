# Gwan Moodle

Projeto de configuração Docker Compose para Moodle LMS com PostgreSQL e Redis.

## 📋 Descrição

Este projeto fornece uma configuração completa do Moodle usando Docker Compose, incluindo:
- **Moodle LMS** - Sistema de gerenciamento de aprendizado
- **PostgreSQL** - Banco de dados
- **Redis** - Cache e sessões

## 🚀 Pré-requisitos

- Docker Engine 20.10+
- Docker Compose 2.0+
- Git (para clonar o repositório)

## 📁 Estrutura do Projeto

```
gwan-moodle/
├── docker-compose.yml          # Configuração para produção (com Traefik)
├── docker-compose.local.yml     # Configuração para desenvolvimento local
├── env.example                  # Exemplo de variáveis de ambiente (produção)
├── env.local.example            # Exemplo de variáveis de ambiente (local)
├── .gitignore                   # Arquivos ignorados pelo Git
└── README.md                    # Este arquivo
```

## 🔧 Configuração

### Ambiente Local

1. **Copie o arquivo de exemplo de variáveis de ambiente:**
   ```bash
   cp env.local.example .env.local
   ```

2. **Edite o arquivo `.env.local` com suas configurações:**
   ```bash
   # Ajuste as senhas e configurações conforme necessário
   MOODLE_HOST=localhost
   MOODLE_SITE_NAME=Demo EAD - Prefeituras (Local)
   
   MOODLE_ADMIN_USER=admin
   MOODLE_ADMIN_PASSWORD=sua_senha_forte
   MOODLE_ADMIN_EMAIL=admin@localhost
   
   # PostgreSQL externo compartilhado (produção)
   POSTGRES_HOST=postgres.gwan.com.br
   POSTGRES_PORT=5432
   POSTGRES_DATABASE=moodle
   POSTGRES_USER=moodle
   POSTGRES_PASSWORD=senha_db_forte
   
   REDIS_PASSWORD=senha_redis_forte
   ```

### Ambiente de Produção

1. **Copie o arquivo de exemplo:**
   ```bash
   cp env.example .env
   ```

2. **Edite o arquivo `.env` com suas configurações de produção:**
   ```bash
   MOODLE_HOST=seu-dominio.com.br
   MOODLE_SITE_NAME=Seu Nome do Site
   # ... outras configurações
   ```

## 🏃 Como Executar

### Ambiente Local

1. **Inicie os serviços:**
   ```bash
   docker-compose -f docker-compose.local.yml --env-file .env.local up -d
   ```

2. **Aguarde o clone do Moodle terminar (primeira execução):**
   ```bash
   docker logs -f moodle_init_local
   ```
   Aguarde a mensagem "Clone finalizado."

3. **Acesse o Moodle:**
   - URL: http://localhost:8080
   - Siga o assistente de instalação

4. **Durante a instalação, use estas informações do banco:**
   - **Tipo de banco:** `PostgreSQL` (ou `pgsql`)
   - **Host do banco de dados:** `postgres` (não use `localhost`)
   - **Nome do banco de dados:** `moodle` (ou o valor de `POSTGRES_DATABASE`)
   - **Usuário do banco de dados:** `moodle` (ou o valor de `POSTGRES_USER`)
   - **Senha do banco de dados:** (valor de `POSTGRES_PASSWORD` do seu `.env.local`)
   - **Porta:** `5432`
   - **Prefixo das tabelas:** `mdl_` (padrão)

### Ambiente de Produção

1. **Certifique-se de que a rede externa `gwan` existe:**
   ```bash
   docker network create gwan
   ```

2. **Configure as variáveis de ambiente** no arquivo `.env` ou `.env.production`:
   ```bash
   # PostgreSQL externo compartilhado
   POSTGRES_HOST=postgres.gwan.com.br
   POSTGRES_PORT=5432
   POSTGRES_DATABASE=moodle
   POSTGRES_USER=moodle
   POSTGRES_PASSWORD=sua_senha
   ```

3. **Inicie os serviços:**
   ```bash
   docker-compose up -d
   ```

4. **O Traefik está configurado automaticamente** através das labels do Docker Compose:
   - O Moodle será acessível via HTTPS em `https://moodle.gwan.com.br` (ou o valor de `MOODLE_HOST`)
   - Certificado SSL automático via Let's Encrypt
   - Apenas HTTPS é habilitado (sem redirect HTTP)
   - O Traefik deve estar configurado com os entrypoints `websecure` e o certresolver `letsencrypt`

**Nota:** Em produção, o PostgreSQL é uma instância externa compartilhada. Certifique-se de que o container do Moodle tem acesso de rede ao host especificado em `POSTGRES_HOST`.

## 📊 Serviços e Portas

### Ambiente Local

| Serviço | Container | Porta | Descrição |
|---------|-----------|-------|-----------|
| Moodle Web | `moodle_web_local` | 8080 | Interface web do Moodle |
| PostgreSQL | `moodle_postgres_local` | 5432 | Banco de dados |
| Redis | `moodle_redis_local` | 6379 | Cache e sessões |
| Moodle Cron | `moodle_cron_local` | - | Tarefas agendadas |
| Moodle Init | `moodle_init_local` | - | Inicialização (clone do código) |

### Ambiente de Produção

| Serviço | Container | Porta | Descrição |
|---------|-----------|-------|-----------|
| Moodle | `moodle_app` | 80 (interno) | Interface web (via Traefik) |
| PostgreSQL | **Externo** | Configurável | Banco de dados compartilhado (via `POSTGRES_HOST`) |
| Redis | `moodle_redis` | 6379 | Cache e sessões |
| Moodle Cron | `moodle_cron` | - | Tarefas agendadas |
| Moodle Init | `moodle_init` | - | Inicialização (clone do código) |

## 🛠️ Comandos Úteis

### Ver logs dos serviços
```bash
# Local
docker-compose -f docker-compose.local.yml logs -f

# Produção
docker-compose logs -f
```

### Ver logs de um serviço específico
```bash
# Local
docker logs moodle_web_local
docker logs moodle_postgres_local

# Produção
docker logs moodle_app
```

### Parar os serviços
```bash
# Local
docker-compose -f docker-compose.local.yml down

# Produção
docker-compose down
```

### Parar e remover volumes (⚠️ apaga dados)
```bash
# Local
docker-compose -f docker-compose.local.yml down -v

# Produção
docker-compose down -v
```

### Reiniciar um serviço específico
```bash
# Local
docker-compose -f docker-compose.local.yml restart moodle-web

# Produção
docker-compose restart moodle
```

### Ver status dos containers
```bash
# Local
docker-compose -f docker-compose.local.yml ps

# Produção
docker-compose ps
```

### Acessar o shell do container
```bash
# Local
docker exec -it moodle_web_local bash

# Produção
docker exec -it moodle_app bash
```

## 🔐 Variáveis de Ambiente

### Principais Variáveis

| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `MOODLE_HOST` | Domínio ou host do Moodle | `localhost` ou `moodle.exemplo.com` |
| `MOODLE_SITE_NAME` | Nome do site | `Demo EAD - Prefeituras` |
| `MOODLE_ADMIN_USER` | Usuário administrador | `admin` |
| `MOODLE_ADMIN_PASSWORD` | Senha do administrador | `senha_forte` |
| `MOODLE_ADMIN_EMAIL` | Email do administrador | `admin@exemplo.com` |
| `POSTGRES_HOST` | Host do PostgreSQL externo (produção) | `postgres.gwan.com.br` |
| `POSTGRES_PORT` | Porta do PostgreSQL externo | `5432` ou `5433` |
| `POSTGRES_DATABASE` | Nome do banco de dados | `moodle` ou `gwan_moodle` |
| `POSTGRES_USER` | Usuário do banco | `moodle` |
| `POSTGRES_PASSWORD` | Senha do banco | `senha_db` |
| `REDIS_PASSWORD` | Senha do Redis | `senha_redis` |

## 🔄 Atualização do Moodle

### Ambiente Local

O Moodle é clonado do repositório oficial na primeira execução. Para atualizar:

1. **Pare os serviços:**
   ```bash
   docker-compose -f docker-compose.local.yml down
   ```

2. **Remova o volume do Moodle (⚠️ backup antes se necessário):**
   ```bash
   docker volume rm gwan-moodle_moodle_www_local
   ```
   
   **Nota:** Se precisar remover o volume do PostgreSQL também:
   ```bash
   docker volume rm gwan-moodle_postgres_data_local
   ```

3. **Inicie novamente:**
   ```bash
   docker-compose -f docker-compose.local.yml --env-file .env.local up -d
   ```

### Ambiente de Produção

Siga a [documentação oficial do Moodle](https://docs.moodle.org/) para atualizações em produção.

## 🐛 Solução de Problemas

### Problema: "Versão do Moodle incorreta"

**Solução:** Aguarde o clone do repositório terminar. Verifique com:
```bash
docker logs moodle_init_local
```

### Problema: "Não consigo conectar ao banco de dados"

**Solução:** Use `postgres` como host (não `localhost`) quando estiver dentro do Docker Compose. Certifique-se de que o tipo de banco está configurado como `PostgreSQL` ou `pgsql`.

### Problema: "Porta já em uso"

**Solução:** Altere as portas no `docker-compose.local.yml` ou pare o serviço que está usando a porta.

### Problema: "Container não inicia"

**Solução:** Verifique os logs:
```bash
docker logs nome_do_container
```

## 📚 Recursos Adicionais

- [Documentação do Moodle](https://docs.moodle.org/)
- [Docker Compose Documentation](https://docs.docker.com/compose/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [Redis Documentation](https://redis.io/docs/)

## 📝 Notas

- ⚠️ **Nunca commite arquivos `.env` ou `.env.local`** - eles contêm senhas e informações sensíveis
- 🔒 Use senhas fortes em produção
- 📦 Os volumes Docker persistem os dados mesmo após parar os containers
- 🔄 O container `moodle-cron` executa tarefas agendadas a cada minuto

## 👥 Contribuindo

1. Faça um fork do projeto
2. Crie uma branch para sua feature (`git checkout -b feature/MinhaFeature`)
3. Commit suas mudanças (`git commit -m 'Adiciona MinhaFeature'`)
4. Push para a branch (`git push origin feature/MinhaFeature`)
5. Abra um Pull Request

## 📄 Licença

Este projeto está sob a licença do Moodle (GPL v3 ou posterior).

## 🆘 Suporte

Para problemas ou dúvidas, abra uma issue no repositório.
