# flag-service (Python)

Este é o serviço de CRUD (Create, Read, Update, Delete) do projeto ToggleMaster. Ele é responsável por gerenciar as *definições* das feature flags.

**IMPORTANTE:** Este serviço é protegido e depende que o `auth-service` esteja rodando. Todas as requisições (exceto `/health`) exigem um header `Authorization: Bearer <sua-chave-api>`.

## 📦 Pré-requisitos (Local)

* [Python](https://www.python.org/) (versão 3.9 ou superior)
* [PostgreSQL](https://www.postgresql.org/download/) (rodando localmente ou em um contêiner Docker)
* O `auth-service` deve estar rodando (localmente na porta `8001`).

## 🚀 Rodando Localmente

1.  **Clone o repositório** e entre na pasta `flag-service`.

2.  **Prepare o Banco de Dados:**
    * Crie um banco de dados no seu PostgreSQL (ex: `flags_db`).
    * Execute o script `db/init.sql` para criar a tabela `flags`:
        ```bash
        psql -U seu_usuario -d flags_db -f db/init.sql
        ```

3.  **Configure as Variáveis de Ambiente:**
    Crie um arquivo chamado `.env` na raiz desta pasta (`flag-service/`) com o seguinte conteúdo:
    ```.env
    # String de conexão do seu banco de dados PostgreSQL
    DATABASE_URL="postgres://SEU_USUARIO:SUA_SENHA@localhost:5432/flags_db"
    
    # Porta que este serviço (flag-service) irá rodar
    PORT="8002"
    
    # URL do auth-service (que deve estar rodando na porta 8001)
    AUTH_SERVICE_URL="http://localhost:8001"
    ```

4.  **Instale as Dependências:**
    ```bash
    pip install -r requirements.txt
    ```

5.  **Inicie o Serviço:**
    ```bash
    gunicorn --bind 0.0.0.0:8002 app:app
    ```
    O servidor estará rodando em `http://localhost:8002`.

## 🔧 Variáveis de Ambiente

O serviço requer as seguintes variáveis de ambiente para funcionar:

### Obrigatórias
- **`DATABASE_URL`** - String de conexão completa com o PostgreSQL
  - Exemplo: `postgres://usuario:senha@localhost:5432/flags_db`
  - Alternativamente, pode ser montada a partir das variáveis individuais abaixo
- **`AUTH_SERVICE_URL`** - URL do Auth Service para validação de chaves
  - Exemplo: `http://localhost:8001`

### Variáveis Individuais (se DATABASE_URL não for definida)
- **`POSTGRES_USER`** - Usuário do PostgreSQL
- **`POSTGRES_PASSWORD`** - Senha do PostgreSQL  
- **`POSTGRES_HOST`** - Host do PostgreSQL (ex: `localhost`)
- **`POSTGRES_PORT`** - Porta do PostgreSQL (ex: `5432`)
- **`POSTGRES_DB`** - Nome do banco de dados

### Outras Variáveis
- **`PORT`** - Porta onde o serviço irá rodar (padrão: `8002`)

### Exemplo de arquivo .env
```env
# Opção 1: URL completa
DATABASE_URL="postgres://togglemaster:senha123@localhost:5432/flags_db"

# Opção 2: Variáveis individuais
POSTGRES_USER="togglemaster"
POSTGRES_PASSWORD="senha123"
POSTGRES_HOST="localhost"
POSTGRES_PORT="5432"
POSTGRES_DB="flags_db"

# Configurações do serviço
PORT="8002"
AUTH_SERVICE_URL="http://localhost:8001"
```

### Notas Importantes
- O **Auth Service** deve estar acessível na URL configurada
- O banco de dados PostgreSQL deve ter a tabela `flags` criada (use `db/init.sql`)
- Todas as requisições (exceto `/health`) exigem header `Authorization: Bearer <chave>`

## 🧪 Testando os Endpoints

**Primeiro, você precisa de uma chave de API válida!**

1.  Vá até o terminal do `auth-service` (que deve estar rodando) e crie uma chave:
    ```bash
    curl -X POST http://localhost:8001/admin/keys \
    -H "Content-Type: application/json" \
    -H "Authorization: Bearer admin-secreto-123" \
    -d '{"name": "admin-para-flag-service"}'
    ```
2.  Copie a chave retornada (ex: `tm_key_...`). Vamos chamá-la de `SUA_CHAVE_API` no resto dos exemplos.

---

**Agora, teste o `flag-service`:**

**1. Verifique a Saúde (Health Check):**
```bash
curl http://localhost:8002/health
```

Saída esperada: `{"status":"ok"}`

**2. Tente Acessar um Endpoint Protegido (Sem Chave):**
```bash
curl http://localhost:8002/flags
```

Saída esperada: `{"error":"Authorization header obrigatório"}`

**3. Crie uma nova Flag (Com a Chave Correta):**
```bash
curl -X POST http://localhost:8002/flags \
-H "Content-Type: application/json" \
-H "Authorization: Bearer SUA_CHAVE_API" \
-d '{
    "name": "enable-new-dashboard",
    "description": "Ativa o novo dashboard para usuários",
    "is_enabled": true
}'
```
Saída esperada: (Um JSON com os dados da flag criada).

**4. Liste todas as Flags:**
```bash
curl http://localhost:8002/flags \
-H "Authorization: Bearer SUA_CHAVE_API"
```
Saída esperada: (Uma lista `[]` contendo a flag que você criou).

**5. Desative a Flag (PUT):**
```bash
curl -X PUT http://localhost:8002/flags/enable-new-dashboard \
-H "Content-Type: application/json" \
-H "Authorization: Bearer SUA_CHAVE_API" \
-d '{"is_enabled": false}'
```
Saída esperada: (O JSON da flag atualizada, com `"is_enabled": false`).