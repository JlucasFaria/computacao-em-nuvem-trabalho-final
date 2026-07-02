# Relatório final — TaskFlow AI (CloudTask AI SaaS)

> Disciplina **Computação em Nuvem** — UNINTER. Projeto executado em **modo local**
> (Docker), com os recursos de nuvem (EKS, RDS, S3, DynamoDB, CDK) estudados como
> **conceito + código versionado**. Preencha os campos `[ ... ]` da identificação.

---

## 1. Identificação

- **Aluno(a):** `[SEU NOME COMPLETO]`
- **RU / matrícula:** `[SEU RU]`
- **Disciplina:** Computação em Nuvem — UNINTER
- **Repositório:** https://github.com/JlucasFaria/computacao-em-nuvem-trabalho-final
  (branch `joaoLucas-trabalho-final`)
- **Data:** `[dd/mm/aaaa]`

## 2. Resumo do projeto

O **TaskFlow AI** é um mini-SaaS de **gerenciamento de tarefas** construído ao
longo da disciplina. Oferece uma **API REST** (FastAPI, com Swagger interativo)
para CRUD de tarefas, **autenticação por token JWT**, **upload de arquivos** e
registro de **eventos/logs** de auditoria. A stack é **Python + FastAPI +
PostgreSQL**, empacotada em **Docker**, com caminhos de nuvem (Kubernetes/EKS,
ECR, S3, DynamoDB) e **infraestrutura como código (AWS CDK)**. Todo recurso de
nuvem tem um **fallback local**, permitindo rodar o projeto inteiro sem AWS.

## 3. O que foi implementado (por semana)

> Executado e validado em **modo local** (Docker Compose: API + PostgreSQL 16).
> Evidências em `docs/entrega-final/evidencias/`.

| Semana | Entreguei | Evidência (comando / endpoint) |
| --- | --- | --- |
| 1 — FastAPI + Docker | API FastAPI com `/`, `/health`, Swagger; imagem Docker + Compose | `GET /health` → `{"status":"ok"}`; `docker compose up` |
| 2 — PostgreSQL + config | CRUD de tarefas no PostgreSQL; config via `.env`/pydantic-settings; readiness | `GET /health/ready` → `{"status":"ready","db":"ok"}`; `POST/GET/PUT/DELETE /tasks` |
| 3 — S3 + Kind | Upload com backend selecionável (modo **local** validado); manifests Kubernetes | `POST /uploads` → `{"storage_mode":"local"}`; `infra/k8s/` |
| 4 — ECR + EKS | Script de build/push para ECR e manifests EKS (caminho AWS, como conceito) | `scripts/semana-04-ecr/`; `infra/k8s/aws/` |
| 5 — HPA + DynamoDB | Eventos automáticos em create/update/delete (modo **local** JSON validado); HPA + teste de carga | `GET /events` → `task.created`; `infra/k8s/hpa.yaml` |
| 6 — CDK + entrega | 7 stacks CDK (S3, ECR, VPC, DynamoDB, etc.), auth JWT, docs finais | `POST /auth/login` → JWT; `infra/cdk/` |
| **Extra (meu toque pessoal)** | Rebranding **→ TaskFlow AI** + endpoint **`GET /tasks/stats`** (agregação) + 3 testes | `GET /tasks/stats` → total + contagem por status/prioridade |

**Testes automatizados:** `docker compose exec -T api pytest` → **73 passed**
(70 da base + 3 do endpoint novo).

## 4. Arquitetura

**O que EU subi (modo local):**

```text
              navegador / curl / Swagger (http://localhost:8000)
                                  │  HTTP
                          ┌───────▼────────┐
                          │  API FastAPI   │  (container cloudtask-api)
                          │  + Auth JWT     │
                          │  /tasks /uploads│
                          │  /events /stats │
                          └───┬───────┬─────┘
               uploads (disco)│       │ tarefas (SQL)
                     ./local_ │       ▼
                     uploads  │   ┌────────────┐
              eventos (JSON)  │   │ PostgreSQL │ (container cloudtask-db, vol. pgdata)
              ./local_events ─┘   └────────────┘
```

**Arquitetura-alvo em produção (conceito, coberto no código/CDK):** a mesma API
rodaria em **EKS** (imagem no **ECR**, HPA 2→5), atrás de **ALB + ACM** (HTTPS na
borda), com **RDS** (tarefas), **S3** (uploads) e **DynamoDB** (eventos) — tudo
descrito como código no **AWS CDK** (`infra/cdk/`). Ver `final-architecture.md`.

## 5. Como executar (reprodutível)

```bash
# pré-requisito: Docker Desktop rodando
cp .env.example .env
docker compose up --build -d          # sobe API + PostgreSQL

# healthcheck
curl http://localhost:8000/health          # {"status":"ok"}
curl http://localhost:8000/health/ready     # {"status":"ready","db":"ok"}

# autenticar (rotas de dados exigem token JWT)
TOKEN=$(curl -s -X POST http://localhost:8000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"username":"admin","password":"admin#123"}' | \
  python -c "import sys,json;print(json.load(sys.stdin)['access_token'])")

# usar a API
curl -X POST http://localhost:8000/tasks -H "Authorization: Bearer $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"title":"Estudar Cloud","priority":"high"}'
curl http://localhost:8000/tasks/stats -H "Authorization: Bearer $TOKEN"

# testes
docker compose exec -T api pytest       # 73 passed

# Swagger interativo: http://localhost:8000/docs
```

## 6. Decisões e trade-offs

- **PostgreSQL como container, não RDS:** sem custo e mais simples para a demo.
  Trade-off: perco backup gerenciado, HA e patch automático que o RDS traria.
- **Modo local para S3 e eventos (fallback):** permite rodar o projeto inteiro
  **sem AWS/credenciais**. Trocar para S3/DynamoDB é só mudar o `.env`.
- **Autenticação JWT no próprio backend:** simplificação didática. Em produção,
  um provedor de identidade dedicado (Cognito/OAuth) centralizaria o login.
- **HTTPS na borda (conceito):** TLS terminaria no ALB/Edge, nunca no app —
  evita loop de redirect nas health probes. Localmente rodo em HTTP puro.
- **Endpoint `/tasks/stats` agrega no banco (COUNT/GROUP BY):** mais eficiente
  que trazer todas as linhas para a memória da API. Declarado **antes** de
  `/tasks/{id}` para o FastAPI não tratar `"stats"` como um id.

## 7. Custos

- **Recursos que cobraram:** **nenhum**. Execução 100% local (Docker na própria
  máquina).
- **Estimativa do período:** **US$ 0,00**.
- **Confirmação de limpeza:** não se aplica — nada foi provisionado na AWS,
  portanto não há recurso cobrável para destruir. Sweep detalhado no
  `deployment-checklist-preenchido.md`.

## 8. LGPD e segurança

- A aplicação coleta **pouquíssimo dado pessoal** (tarefas são texto livre; uma
  única conta administrativa). Dados em PostgreSQL/disco local.
- **Segredos protegidos:** `.env` no `.gitignore` e **não** versionado
  (verificado); **nenhuma chave AWS real** no repositório. A senha `admin#123` é
  credencial de **demo** documentada.
- **Direitos do titular:** acesso via `GET /tasks`, exclusão via `DELETE /tasks`.
- **Em produção real ficaria pendente:** TLS na borda (ACM), criptografia em
  repouso (S3/RDS), segredos no Secrets Manager, roles IAM de menor privilégio.
- Detalhes no `lgpd-checklist-preenchido.md`.

## 9. Dificuldades e aprendizados

- **Autenticação inesperada:** ao rodar, descobri que as rotas de dados agora
  exigem **token JWT** (Semana 6). Aprendi a fazer login e usar o header
  `Authorization: Bearer`. *(Lição: sempre validar o fluxo real antes de assumir.)*
- **Ordem de rotas no FastAPI:** ao criar `/tasks/stats`, entendi que rotas
  específicas precisam vir **antes** das paramétricas (`/tasks/{id}`), senão o
  framework confunde `"stats"` com um id → erro 422.
- **Agregação no banco:** usar `COUNT` + `GROUP BY` em vez de contar em Python.
- **Fallback local:** compreendi na prática o valor de projetar com fallback —
  consegui rodar e demonstrar tudo **sem gastar 1 centavo** na AWS.
- **Cuidado com segredos:** verifiquei de fato (não "no chute") que nada sensível
  foi para o git antes de publicar o repositório.

## 10. Anexos

- [x] `lgpd-checklist-preenchido.md` preenchido
- [x] `deployment-checklist-preenchido.md` (custos/limpeza) preenchido
- [x] Evidências: `evidencias/01-smoke-test-local.md`, `evidencias/02-toque-pessoal.md`
- [ ] Prints do Swagger (`http://localhost:8000/docs`) — **anexar na hora de gerar o PDF**
