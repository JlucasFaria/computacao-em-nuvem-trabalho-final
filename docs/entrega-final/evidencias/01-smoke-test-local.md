# Evidências — Passo 1: execução local (smoke test)

> Ambiente: Windows 11 + Docker Desktop. Modo **local** (fallback sem AWS):
> `STORAGE_MODE=local`, eventos em JSON. Data: 02/07/2026.

## Como foi subido

```bash
docker compose up --build -d      # sobe API (FastAPI) + PostgreSQL 16
docker compose ps                 # cloudtask-api + cloudtask-db (healthy)
```

## 1. Metadados, liveness e readiness

```text
GET /              -> {"name":"CloudTask AI SaaS","version":"0.6.0","docs":"/docs"}
GET /health        -> {"status":"ok"}
GET /health/ready  -> {"status":"ready","db":"ok"}      # readiness checou o Postgres
```

## 2. Autenticação (JWT — Semana 6)

```text
POST /auth/login  {"username":"admin","password":"admin#123"}
  -> {"access_token":"eyJhbGciOiJIUzI1NiI...","token_type":"bearer","expires_in":28800}
```

As rotas de dados exigem `Authorization: Bearer <token>` (retornam 401 sem token).

## 3. CRUD de tarefas (Semana 2)

```text
POST /tasks {"title":"Minha primeira tarefa","description":"...","priority":"high"}
  -> 201 {"id":1,"status":"pending","priority":"high", ...}
GET /tasks
  -> [ {"id":1,"title":"Minha primeira tarefa", ...} ]
```

## 4. Upload de arquivo — modo local (Semana 3)

```text
POST /uploads  (multipart, file=@README.md)
  -> {"filename":"2d923ad12998f5a9-ddd81d19.md","url":"/uploads/...","storage_mode":"local"}
```

## 5. Eventos automáticos (Semana 5 / Aula 10)

```text
GET /events
  -> [ {"event_type":"task.created","task_id":1,
        "message":"Tarefa 1 criada: 'Minha primeira tarefa'.", ...} ]
```

O evento `task.created` foi gerado **automaticamente** ao criar a tarefa — integra
CRUD (Aula 3) + eventos (Aula 10).

## 6. Testes automatizados

```bash
docker compose exec -T api pytest
# ......................................................................  [100%]
# 70 passed in 1.89s
```

## Conclusão

Aplicação sobe e funciona **ponta a ponta em modo local**, sem depender da AWS,
conforme o princípio de *fallback local* definido pelo professor em `docs/TAREFAS.md`.
Todos os endpoints das Aulas 1–12 respondem; 70 testes automatizados passam.
