# Flag Service 🚩

Serviço de gerenciamento (CRUD) de feature flags do **ToggleMaster**. Este serviço é responsável por criar, ler, atualizar e deletar as definições das feature flags.

## 🎯 Descrição do Serviço

O Flag Service é o ponto central de controle para definições de feature flags. Ele:

1. Gerencia o lifecycle completo de feature flags (criar, listar, atualizar, desativar)
2. Armazena definições em PostgreSQL
3. Requer autenticação via chaves de API (valida com Auth Service)
4. Expõe endpoints para operações CRUD protegidas
5. Fornece endpoint `/health` para monitoramento

**Função crítica:** Nenhuma feature flag existe sem estar registrada neste serviço. É a "fonte da verdade" para todas as flags.

## 📦 Stack Técnico

- **Linguagem:** Python 3.9+
- **Framework:** Flask
- **Banco de Dados:** PostgreSQL
- **Autenticação:** Bearer Token (integração com Auth Service)
- **Dependências principais:** psycopg2, flask, python-dotenv

## 🚀 Como Usar

### Pré-requisitos Locais

- Python 3.9 ou superior
- PostgreSQL 12+ (instalado ou via Docker)
- Auth Service rodando (porta 8001)

### Setup Local

#### 1. Clone e Navegue para o Diretório
```bash
cd Flag-Service
```

#### 2. Prepare o Banco de Dados

Crie um banco de dados PostgreSQL:
```bash
createdb flags_db
```

Execute o script de inicialização:
```bash
psql -U seu_usuario -d flags_db -f db/init.sql
```

Este script cria a tabela `flags` com a seguinte estrutura:
```sql
CREATE TABLE flags (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) UNIQUE NOT NULL,
    description TEXT,
    is_enabled BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### 3. Configure as Variáveis de Ambiente
Crie um arquivo `.env` na raiz do serviço:

```env
# Banco de Dados PostgreSQL
DATABASE_URL=postgres://usuario:senha@localhost:5432/flags_db

# Ou configure individualmente:
POSTGRES_USER=togglemaster
POSTGRES_PASSWORD=seu_password_seguro
POSTGRES_HOST=localhost
POSTGRES_PORT=5432
POSTGRES_DB=flags_db

# Serviço
PORT=8002

# Auth Service (para validação de chaves)
AUTH_SERVICE_URL=http://localhost:8001

# Ambiente
ENVIRONMENT=development
```

#### 4. Instale as Dependências
```bash
pip install -r requirements.txt
```

#### 5. Inicie o Serviço
```bash
gunicorn --bind 0.0.0.0:8002 app:app
```

O servidor estará disponível em `http://localhost:8002`.

### Testando Localmente

#### 1. Crie uma Chave de API
Primeiro, gere uma chave via Auth Service (se não tiver):

```bash
curl -X POST http://localhost:8001/admin/keys \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer admin-secreto-123" \
  -d '{"name": "flag-service-client"}'

# Salve a chave retornada como SUA_CHAVE_API
```

#### 2. Health Check
```bash
curl http://localhost:8002/health
# Resposta esperada: {"status":"ok"}
```

#### 3. Tentar Acessar Sem Chave (Deve Falhar)
```bash
curl http://localhost:8002/flags
# Resposta esperada: {"error":"Authorization header obrigatório"}
```

#### 4. Criar uma Nova Flag
```bash
curl -X POST http://localhost:8002/flags \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_CHAVE_API" \
  -d '{
    "name": "enable-new-dashboard",
    "description": "Ativa o novo dashboard para usuários",
    "is_enabled": true
  }'

# Resposta esperada:
# {
#   "id": 1,
#   "name": "enable-new-dashboard",
#   "description": "Ativa o novo dashboard para usuários",
#   "is_enabled": true,
#   "created_at": "2025-05-17T10:30:00"
# }
```

#### 5. Listar Todas as Flags
```bash
curl http://localhost:8002/flags \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: lista JSON de todas as flags
```

#### 6. Atualizar uma Flag
```bash
curl -X PUT http://localhost:8002/flags/enable-new-dashboard \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer SUA_CHAVE_API" \
  -d '{"is_enabled": false}'

# Resposta esperada: flag atualizada com is_enabled=false
```

#### 7. Obter Flag Específica
```bash
curl http://localhost:8002/flags/enable-new-dashboard \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: JSON da flag
```

#### 8. Deletar uma Flag
```bash
curl -X DELETE http://localhost:8002/flags/enable-new-dashboard \
  -H "Authorization: Bearer SUA_CHAVE_API"

# Resposta esperada: {"message":"Flag deletada com sucesso"}
```

## 🔧 Variáveis de Ambiente

### Obrigatórias
| Variável | Descrição | Exemplo |
|----------|-----------|---------|
| `POSTGRES_USER` | Usuário PostgreSQL | `togglemaster` |
| `POSTGRES_PASSWORD` | Senha PostgreSQL | `senha_forte_123` |
| `POSTGRES_HOST` | Host PostgreSQL | `localhost` |
| `POSTGRES_PORT` | Porta PostgreSQL | `5432` |
| `POSTGRES_DB` | Nome do banco de dados | `flags_db` |
| `AUTH_SERVICE_URL` | URL do Auth Service | `http://localhost:8001` |

**Nota:** Alternativamente, use `DATABASE_URL` ao invés das variáveis individuais.

### Opcionais
| Variável | Descrição | Padrão |
|----------|-----------|--------|
| `PORT` | Porta do servidor | `8002` |
| `ENVIRONMENT` | Ambiente (development/production) | `development` |
| `LOG_LEVEL` | Nível de log | `INFO` |
| `MAX_CONNECTIONS` | Conexões máximas ao BD | `10` |

## 🔐 GitHub Secrets Necessários

Configure os seguintes secrets no GitHub para CI/CD:

```yaml
POSTGRES_USER
  Descrição: Usuário PostgreSQL
  Valor: togglemaster

POSTGRES_PASSWORD
  Descrição: Senha PostgreSQL
  Valor: <sua-senha-forte>

POSTGRES_HOST
  Descrição: Host PostgreSQL
  Valor: db.example.com

POSTGRES_PORT
  Descrição: Porta PostgreSQL
  Valor: 5432

POSTGRES_DB
  Descrição: Nome do banco de dados
  Valor: flags_db

DATABASE_URL
  Descrição: String de conexão completa
  Valor: postgres://user:password@host:5432/flags_db

AUTH_SERVICE_URL
  Descrição: URL do Auth Service
  Valor: http://auth-service:8001

DOCKERHUB_USERNAME
  Descrição: Docker Hub username
  Valor: <seu-username>

DOCKERHUB_TOKEN
  Descrição: Docker Hub personal access token
  Valor: <seu-token>

REGISTRY_URL
  Descrição: URL do registry de container
  Valor: docker.io

SONAR_TOKEN
  Descrição: Token SonarQube
  Valor: <seu-token>
```

## 📊 Endpoints da API

### Públicos (sem autenticação)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/health` | Verifica saúde do serviço |

### Protegidos (requer Bearer Token)
| Método | Endpoint | Descrição |
|--------|----------|-----------|
| GET | `/flags` | Lista todas as flags |
| POST | `/flags` | Cria uma nova flag |
| GET | `/flags/{name}` | Obtém flag específica |
| PUT | `/flags/{name}` | Atualiza uma flag |
| DELETE | `/flags/{name}` | Deleta uma flag |

### Corpo (POST/PUT)

```json
{
  "name": "enable-feature",
  "description": "Descrição da feature flag",
  "is_enabled": true
}
```

## 🏗️ Arquitetura

```
Request → Middleware Auth → Flask Route → Validator → Database → Response
                ↓
         Validação na Auth Service
```

## 📋 Exemplo de Fluxo Completo

1. **Criar Flag:**
```bash
curl -X POST http://localhost:8002/flags \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tm_key_..." \
  -d '{"name":"new-feature","is_enabled":true}'
```

2. **Listar Flags:**
```bash
curl http://localhost:8002/flags \
  -H "Authorization: Bearer tm_key_..."
```

3. **Desativar Flag:**
```bash
curl -X PUT http://localhost:8002/flags/new-feature \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer tm_key_..." \
  -d '{"is_enabled":false}'
```

4. **Deletar Flag:**
```bash
curl -X DELETE http://localhost:8002/flags/new-feature \
  -H "Authorization: Bearer tm_key_..."
```

## 🔐 Segurança

- ✅ Todas as requisições (exceto `/health`) exigem Bearer Token válido
- ✅ Tokens são validados via Auth Service
- ✅ Senhas de banco de dados devem estar em variáveis de ambiente
- ✅ Use HTTPS em produção
- ✅ Implemente rate limiting em produção

## 🐛 Troubleshooting

### Problema: "psycopg2.OperationalError: could not connect"
**Solução:** Verifique credenciais do PostgreSQL
```bash
psql -U seu_usuario -d flags_db -c "SELECT 1"
```

### Problema: "Authorization header obrigatório"
**Solução:** Adicione o header Authorization com uma chave válida
```bash
curl -H "Authorization: Bearer sua_chave" ...
```

### Problema: "Invalid token"
**Solução:** Verifique se a chave ainda é válida (pode ter sido revogada)

### Problema: "Flag already exists"
**Solução:** Nomes de flags devem ser únicos; tente outro nome

## 📊 Monitoramento

### Logs
```bash
docker logs flag-service
# ou localmente
tail -f logs/flag-service.log
```

### Métricas
- Número total de flags ativas
- Taxa de criação de flags
- Tempo de resposta dos endpoints

## 📚 Recursos Adicionais

- [Flask Documentation](https://flask.palletsprojects.com/)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)
- [ToggleMaster Architecture](../README.md)

## 👥 Suporte

Para dúvidas ou problemas, abra uma issue no repositório principal ou entre em contato com o time DevOps.
