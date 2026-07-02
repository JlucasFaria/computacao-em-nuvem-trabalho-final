# Checklist de deploy + custos — TaskFlow AI (PREENCHIDO)

> Preenchido por **João Lucas** em 02/07/2026. Execução em **modo local** (Docker
> Compose: API FastAPI + PostgreSQL 16). **Nada foi provisionado na AWS**, então
> os blocos de nuvem (EKS/RDS/ELB/CDK) são marcados *"não se aplica"* com a
> explicação de como seriam em produção. Baseado em `deployment-checklist.md`.

---

## Antes do deploy

- [~] Credenciais AWS válidas (`aws sts get-caller-identity`): **não se aplica** —
      execução local, sem AWS.
- [~] Região correta (`us-east-1`): não se aplica (local).
- [~] Imagem no ECR: não se aplica. A imagem foi construída **localmente**
      (`docker compose build`, `cloudtask-api:dev`). Em produção iria para o ECR
      via `scripts/semana-04-ecr/build-push-ecr.sh`.
- [x] `.env` revisado: criado a partir de `.env.example`, sem placeholders quebrados.
- [x] Banco definido: **PostgreSQL 16 como container** (serviço `db` do Compose),
      ciente do trade-off vs. RDS (RDS traria backup gerenciado, HA, patch
      automático; o container é mais simples e sem custo, ideal para a demo).

## Durante (subir)

- [x] Aplicação sobe sem erro: `docker compose up --build -d` → `cloudtask-api` +
      `cloudtask-db` (este último `healthy`).
- [~] Cluster EKS `Ready` / metrics-server / `kubectl apply`: **não se aplica**
      (local). Manifests existem em `infra/k8s/aws/` para o caminho AWS.
- [x] Serviço acessível: `curl http://localhost:8000/health` → `200`;
      `/health/ready` → `{"status":"ready","db":"ok"}` (readiness checou o Postgres).

## Verificação funcional (demonstrar)

- [x] **Autenticação**: `POST /auth/login` (admin/admin#123) devolve JWT; rotas de
      dados exigem `Authorization: Bearer <token>`.
- [x] **CRUD**: criar/listar/atualizar/excluir tarefa (Swagger ou curl) — OK.
- [x] **Upload**: `POST /uploads` em **modo local** grava em `./local_uploads` e
      devolve nome + URL; download OK.
- [x] **Evento**: criar tarefa gera `task.created` (`GET /events`) — OK.
- [x] **Extra (meu)**: `GET /tasks/stats` devolve total + contagem por status e
      prioridade — OK.
- [~] **HPA** (gerar carga e ver réplicas): não se aplica (local, sem cluster).

## 🔥 Depois (destruir — OBRIGATÓRIO na nuvem)

- [~] Todos os itens (`kubectl delete`, `eksctl delete cluster`, apagar RDS/
      DynamoDB/S3, `cdk destroy`): **não se aplica** — nada foi criado na AWS,
      portanto **não há recurso cobrável para destruir**.
- [x] Ambiente local encerrado quando necessário: `docker compose down`
      (usar `down -v` zera o volume `pgdata`).

## Sweep final (tudo vazio = zero cobrança)

- [~] Comandos de varredura (EKS/EC2/ELB/NAT/EIP/RDS): **não se aplica** (local).
      Como nada foi provisionado, o resultado de todos seria vazio por definição.
- [x] **Custo do período: US$ 0,00** — execução 100% local, sem uso de AWS.

---

## Resumo de custos

| Recurso | Cobrou? | Observação |
| --- | --- | --- |
| Tudo (execução local) | **Não** | Docker na própria máquina; **US$ 0,00** |
| EKS / EC2 / RDS / ELB / S3 / DynamoDB | Não | Não provisionados — caminho AWS ficou como **conceito** |

**Legenda:** `[x]` feito · `[~]` não se aplica ao modo local (explicado).
