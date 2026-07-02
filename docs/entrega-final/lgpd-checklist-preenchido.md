# Checklist LGPD + segurança — TaskFlow AI (PREENCHIDO)

> Preenchido por **João Lucas** em 02/07/2026. Projeto executado em **modo local**
> (Docker, sem AWS). Itens específicos de nuvem são marcados como *"não se aplica
> — modo local"* com a explicação de **como seria em produção real**, mostrando
> entendimento do conceito. Baseado em `lgpd-checklist.md` (template do professor).

---

## 1. Dados pessoais — mapeamento

- [x] **Quais** dados pessoais a aplicação coleta: **praticamente nenhum**. As
      tarefas são texto livre (`title`/`description`); só haveria dado pessoal se
      o usuário digitasse um no texto. A autenticação usa **uma única conta
      administrativa** (`admin`), sem cadastro de titulares.
- [x] **Onde** cada dado é armazenado: tarefas no **PostgreSQL**; uploads no
      **disco local** (`./local_uploads`, modo local); eventos em **JSON local**
      (`./local_events/events.json`). Em produção: RDS + S3 + DynamoDB.
- [x] **Por quanto tempo / como apagar**: não há retenção automática. O titular
      apaga via `DELETE /tasks/{id}`. Sem política formal de expiração (seria
      definida em produção real).

## 2. Bases legais e finalidade (LGPD art. 6–11)

- [x] **Finalidade específica**: gerenciar tarefas (estudo/demonstração da
      disciplina). Documentada aqui e no README.
- [x] **Base legal**: projeto **didático** — a finalidade documentada cumpre o
      exercício. Em uso real, seria execução de contrato / legítimo interesse.

## 3. Segurança técnica (LGPD art. 46)

- [~] **Em trânsito**: em modo local roda em **HTTP** (`localhost`, dev). *Não se
      aplica TLS localmente.* Em produção real, o TLS termina na **borda** (Edge
      Caddy com cert ACME, ou ALB + ACM) — a API nunca administra certificado.
      Conceito coberto em `docs/conceitos/https-tls.md`.
- [~] **Em repouso**: disco local sem criptografia (dev). Em produção:
      S3 (`S3_MANAGED`), RDS (encryption at rest), DynamoDB (padrão) — todos com
      criptografia ativa.
- [x] **Segredos fora do código/git**: **verificado** — `.env` está no
      `.gitignore` e **não** é rastreado (`git ls-files` não lista `.env`); nenhuma
      chave AWS real (`AKIA…`) no repositório. A senha de demo `admin#123` é uma
      **simplificação didática** documentada (default em `config.py`); em produção
      viria de Secrets Manager / SSM e nunca seria fixa.
- [~] **Bucket S3 privado (Block Public Access)**: não se aplica (modo local). As
      stacks CDK (`infra/cdk/`) criam o bucket **privado e criptografado** —
      conceito coberto.
- [~] **Credenciais temporárias (roles)**: não se aplica localmente. Em AWS, o
      EC2/EKS usa **roles** (sem chave fixa) — ver `infra/cdk` e docs.
- [x] **Menor privilégio**: a aplicação acessa só o banco/armazenamento que
      precisa; rotas de dados exigem token JWT.

## 4. Direitos do titular (LGPD art. 18)

- [x] **Acessar** os dados: `GET /tasks` (listar) e `GET /tasks/{id}` (consultar).
- [x] **Excluir**: `DELETE /tasks/{id}` remove a tarefa. (Uploads podem ser
      removidos do armazenamento correspondente.)
- [x] **Logs sem dado sensível**: os eventos (`task.created/updated/deleted`)
      guardam só id, tipo e uma mensagem curta — **não** copiam conteúdo sensível.

## 5. Operação e incidentes

- [~] **Backups**: em modo local, os dados vivem no volume `pgdata` do Docker.
      Em produção: RDS com snapshot automático + S3 versionado (conceito coberto).
- [x] **Reação a vazamento**: rotacionar `SECRET_KEY` (invalida os JWT), trocar a
      senha admin, e — em produção — rotacionar segredos no Secrets Manager.
- [~] **Cost/uso monitorado (Budgets)**: não se aplica (sem gasto AWS). Conceito
      em `docs/conceitos/cost-explorer.md`.

## 6. Higiene de projeto

- [x] **Nenhuma conta AWS real ou segredo commitado**: verificado (seção 3).
- [x] **Recursos de teste destruídos**: não se aplica — nada foi provisionado na
      AWS (modo local). Sem recurso órfão cobrável.
- [x] **README/docs não expõem credenciais internas**: só a conta de **demo**
      documentada (`admin`/`admin#123`), intencional para a avaliação.

---

**Legenda:** `[x]` feito/verificado · `[~]` não se aplica ao modo local (explicado
como seria em produção).
