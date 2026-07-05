# tasks.md — o que falta para finalizar a entrega

> Mapa de finalização do trabalho final (**TaskFlow AI / CloudTask AI SaaS**),
> cruzando o que já fizemos com as **instruções do AVA** (Slides do professor).
> Legenda: ✅ feito · 🟡 parcial/fácil · ❌ falta · ❓ decisão pendente.
> Atualizado: 04/07/2026. Prazo: **estendido pelo professor** (data original 29/06/2026).

---

## A. Entregáveis obrigatórios — imagens no PDF (Slide 13)

O relatório PDF final **precisa conter imagens de**:

- [x] **Aplicação funcionando** — ✅ 9 prints do Swagger (Anexo C do `ENTREGA-COMPLETA.md`).
- [x] **Deploy em cloud** — ✅ FEITO! App rodando em **EC2 t3.micro** (Amazon Linux 2023,
      São Paulo), acessível em `http://15.229.42.56:8000/docs`. Prints no Anexo E.3.
      ⚠️ DESTRUIR a instância EC2 + bucket S3 após conferir (ver teardown).
- [x] **Containers Docker** — ✅ PRINT salvo (`prints/docker-containers.png`, Anexo D).
- [x] **Kubernetes funcionando** — ✅ RODANDO via **Kind** (2 réplicas + auto-healing);
      prints `prints/k8s-swagger.png` e `prints/k8s-pods.png` (Anexo D);
      evidência em `evidencias/03-kubernetes-kind.md`.
- [x] **Link GitHub do projeto** — ✅ https://github.com/JlucasFaria/computacao-em-nuvem-trabalho-final

## B. Funcionalidades mínimas do sistema (Slide 11)

- [x] Login / autenticação simples — ✅ JWT (`POST /auth/login`, admin/admin#123).
- [ ] **Cadastro de usuários** — ❓ VERIFICAR. A base só tem 1 conta admin fixa (sem registro de usuários). Decidir: (a) considerar "login simples" suficiente, ou (b) adicionar um endpoint simples de cadastro de usuário como toque extra.
- [x] CRUD de tarefas — ✅.
- [x] Status e prioridades — ✅ (`pending/in_progress/done`, `low/medium/high`).
- [x] Upload de arquivos — ✅ (modo local validado).
- [x] API REST funcionando — ✅ (Swagger).
- [x] Persistência em banco SQL — ✅ (PostgreSQL).

## C. Tecnologias/conceitos OBRIGATÓRIOS (Slide 12)

- [x] Python + FastAPI — ✅
- [x] Docker — ✅
- [ ] **Kubernetes** — ❌ falta rodar (Kind local resolve para a evidência).
- [ ] **AWS Academy** — ❓ depende de acesso (seção D).
- [ ] **PostgreSQL/RDS** — 🟡 Postgres ✅ local; **RDS** falta (parte do deploy cloud).
- [x] **S3** — ✅ FEITO na AWS real (bucket `taskflow-ai-uploads-jlucas` + upload;
      Anexo E). Conta `7303-3524-6337`, região sa-east-1. Relógio estava dessincronizado
      (causa de todos os erros "Acesso negado/Signature expired") — corrigido.
- [x] GitHub — ✅
- Conceitos (SaaS, elasticidade, escalabilidade, alta disp., containers, EKS,
  segurança, IAM, MFA, criptografia, custos, backup, Data Lake): documentados
  nos `docs/conceitos/` e no relatório — 🟡 revisar se todos aparecem no relatório.

## D. ❓ BLOQUEIO PRINCIPAL — acesso ao AWS Academy

- [ ] **Confirmar acesso ao AWS Academy / Learner Lab** (credenciais ativas).
  - Se **SIM** → plano cloud: subir imagem no **ECR**, criar **RDS**, **S3**,
    cluster **EKS**, aplicar `infra/k8s/aws/`, tirar prints ("Deploy em cloud"),
    e **destruir tudo** depois (evitar custos). Cobre S3 + RDS + EKS + IAM de uma vez.
  - Se **NÃO** → fazer **Kubernetes local (Kind)** para a evidência de K8s e
    documentar o deploy cloud como arquitetura + código (CDK/manifests), sendo
    transparente sobre a limitação no relatório.

## E. Tecnologias OPCIONAIS / bônus (Slide 14) — já temos!

- [x] AWS CDK — ✅ (7 stacks em `infra/cdk/`) — pode virar print de `cdk synth`.
- [x] DynamoDB / NoSQL — ✅ (eventos; modo local, real na AWS se houver acesso).
- [ ] GitHub Actions — ❌ (opcional; não faremos — foge do escopo "sem CI/CD profundo").
- [ ] CloudWatch — ❌ (opcional; só se fizermos o caminho cloud completo).
- [ ] Auto Scaling / HPA — 🟡 (manifest `hpa.yaml` existe; demonstrar exige cluster).

## F. Fechamento (depois de resolver A–D)

- [ ] Inserir os novos prints (Docker, Kubernetes, cloud) no `ENTREGA-COMPLETA.md`.
- [ ] Revisar o relatório para citar os conceitos obrigatórios do Slide 12.
- [ ] Commit + push de tudo no GitHub.
- [ ] Converter `ENTREGA-COMPLETA.md` → **PDF** e enviar no portal.

---

## Resumo — o caminho crítico

1. **Decidir D** (tem AWS Academy?) — destrava "Deploy em cloud", S3, RDS.
2. **Kubernetes funcionando** (Kind local) — pode fazer já, sem custo.
3. **Print dos containers Docker** — trivial.
4. Inserir prints no relatório → PDF → entregar.
