# Ejercicion

## Codex en el proyecto

```sh
mkdir -p .codex
nano .codex/config.toml
```

```toml
# Evita pedir aprobaciones
approval_policy = { granular = { sandbox_approval = false, rules = false, mcp_elicitations = false, request_permissions = false, skill_approval = false } }
sandbox_mode = "workspace-write"

[sandbox_workspace_write]
network_access = true

[mcp_servers.mysql]
command = "uvx"
args = ["--from", "mysql-mcp-server", "mysql_mcp_server"]

[mcp_servers.mysql.env]
MYSQL_HOST = "localhost"
MYSQL_PORT = "3306"
MYSQL_DATABASE = "fake"
MYSQL_USER = "read_user"
MYSQL_PASSWORD = "SECRET_PASSWORD"
```

## Claude en el proyecto

```sh
mkdir -p .claude
nano .claude/settings.json
```
settings.json
```json
{
  "permissions": {
    "allow": [
      "Write",
      "Edit",
      "MultiEdit",
      "Read",
      "Glob",
      "Grep",
      "LS",
      "Bash"
    ],
    "deny": [
      "Write(.env)",
      "Edit(.env)",
      "MultiEdit(.env)",
      "Write(agents/**)",
      "Edit(agents/**)",
      "MultiEdit(agents/**)"
    ]
  }
}
```


## Test por agente

### Test agente para analisis de la base de datos

1. Ejecuta el agente que analiza la base de datos

```sh
omni run agents/codex_mysql_model_reader
```

Escribe este prompt en la consola o en la interfaz web:

```md
Analiza la base de datos MySQL fake mediante el MCP configurado. Revisa las tablas customers, products, sales y shops. Genera los archivos outputs/data_model.md, outputs/join_strategy.md, outputs/medallion_flow.md y outputs/work_plan.md. No ejecutes operaciones INSERT, UPDATE, DELETE, DROP, TRUNCATE, ALTER ni CREATE.
```

Como resultado esperado deben generarse archivos en la carpeta outputs

### Test al agente de claude code

```sh
omni run agents/claude_pyspark_builder
```

Prompt

```md
Implementa el laboratorio.
```


## Ejecutar el agente orquestador

```sh
omni run agents/orchestrator
```

Prompt:
```md
Ejecuta el laboratorio completo: primero Codex para generar outputs y luego Claude Code para generar el código PySpark y el dashboard.
```