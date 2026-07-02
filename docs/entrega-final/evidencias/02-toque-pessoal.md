# Evidências — Passo 2: toque pessoal

> Customizações feitas por mim sobre a base, para o projeto não ser cópia
> idêntica e mostrar domínio do código. Data: 02/07/2026.

## 1. Rebranding: CloudTask AI SaaS → **TaskFlow AI**

Alterado o nome visível do serviço no título do Swagger, no endpoint raiz e na
configuração.

Arquivos: `app/main.py` (título FastAPI, endpoint `/`, descrição),
`app/core/config.py` (`app_name`), `tests/test_health.py` (asserção do nome).

```text
GET /  ->  {"name":"TaskFlow AI","version":"0.6.0","docs":"/docs"}
```

O título **TaskFlow AI** aparece no topo do Swagger UI (`/docs`).

## 2. Endpoint novo: GET /tasks/stats (não existia na base)

Retorna um **resumo agregado** das tarefas — total geral + contagem por `status`
e por `priority`. Feito com `COUNT` + `GROUP BY` no PostgreSQL (agregação no banco,
não em Python).

Arquivos: `app/api/routes_tasks.py` (endpoint + query),
`app/db/schemas.py` (schema `TaskStats`), `tests/test_tasks_crud.py` (3 testes).

Detalhe técnico: o endpoint é declarado **antes** de `/tasks/{task_id}` para o
FastAPI não interpretar `"stats"` como um id (evita erro 422).

```text
GET /tasks/stats
  -> {"total":3,
      "by_status":{"pending":1,"in_progress":1,"done":1},
      "by_priority":{"low":1,"medium":0,"high":2}}
```

## 3. Testes

Adicionei 3 testes automatizados para o novo endpoint (`TestStats`):
vazio, contagem por status/prioridade, e a não-colisão com a rota `/{task_id}`.

```bash
docker compose exec -T api pytest
# 73 passed in 2.01s     (70 da base + 3 novos)
```

Toda a suíte continua verde — a customização não quebrou nada da base.
