<!-- ============================================================= -->
<!-- Trabalho final — João Lucas Faria Filho (RU 4994592)          -->
<!-- Este projeto foi rebatizado como TaskFlow AI (toque pessoal): -->
<!-- rebranding do CloudTask AI SaaS + novo endpoint GET /tasks/stats. -->
<!-- Relatório e evidências em docs/entrega-final/ENTREGA-COMPLETA.md -->
<!-- ============================================================= -->

> 🎓 **TaskFlow AI** — entrega final da disciplina, baseada no projeto
> **CloudTask AI SaaS**. Rebranding + endpoint `GET /tasks/stats` (toque pessoal).
> Relatório completo em [`docs/entrega-final/ENTREGA-COMPLETA.md`](docs/entrega-final/ENTREGA-COMPLETA.md).

<!-- Área do Banner -->
<div align="center" style="background-color: white; max-width: 70%;">
  <img alt="BANNER do repositório CloudTask AI SaaS — disciplina Computação em Nuvem" title="Banner_CloudTask_AI_SaaS" src=".readme_docs/Banner_Github_NCPU.png" width="100%" />
</div>

<!-- Título e breve descrição do repositório -->
<div align="center">
  <h1>CloudTask AI SaaS — Semana 6 (Aulas 11 e 12) — final da disciplina</h1>
  <p><b>Branch <code>semana-06-cdk-final</code> — cobre as Aulas 11 e 12; consolida toda a jornada das 6 semanas.</b></p>
  <p>API FastAPI + PostgreSQL + CRUD com <b>uploads S3/local</b>, <b>Kubernetes (Kind→EKS)</b>, <b>HPA</b> e <b>eventos (DynamoDB)</b> — agora fechando com <b>Infraestrutura como Código (AWS CDK)</b> (Aula 11) e os <b>materiais de entrega final</b> (Aula 12).</p>
</div>

<p align="center">
  <a href="https://www.python.org/" title="Python"><img src="https://github.com/get-icon/geticon/raw/master/icons/python.svg" alt="Python" height="21px"></a>
  +
  <a href="https://fastapi.tiangolo.com/" title="FastAPI"><img src="https://icon.icepanel.io/Technology/svg/FastAPI.svg" alt="FastAPI" height="21px"></a>
  +
  <a href="https://www.docker.com/" title="Docker"><img src="https://github.com/get-icon/geticon/raw/master/icons/docker-icon.svg" alt="Docker" height="21px"></a>
  +
  <a href="https://www.postgresql.org/" title="PostgreSQL"><img src="https://github.com/get-icon/geticon/raw/master/icons/postgresql.svg" alt="PostgreSQL" height="21px"></a>
  +
  <a href="https://aws.amazon.com/s3/" title="Amazon S3">Amazon S3</a>
  +
  <a href="https://kubernetes.io/" title="Kubernetes">Kubernetes</a>
  +
  <a href="https://aws.amazon.com/ecr/" title="Amazon ECR">Amazon ECR</a>
  +
  <a href="https://aws.amazon.com/eks/" title="Amazon EKS">Amazon EKS</a>
  +
  <a href="https://aws.amazon.com/cdk/" title="AWS CDK">AWS CDK</a>
</p>

## O que foi feito nesta semana

> 🎓 **Última semana da disciplina.** Esta branch parte do estado da Semana 5
> (toda a base: CRUD, S3, Kind/EKS, HPA, DynamoDB) e fecha com **IaC** e a
> **entrega final**. Versão da API: **`0.6.0`**.

### Aula 11 — Infraestrutura como Código (AWS CDK)

- `infra/cdk/` — descreve parte da infra como **Python versionado**:
  - `app.py` + `cdk.json` — app CDK e as 3 stacks.
  - `stacks/storage_stack.py` — bucket **S3** privado (criptografado + versionado).
  - `stacks/ecr_stack.py` — repositório **ECR** `cloudtask-api` (scan + lifecycle).
  - `stacks/network_stack.py` — **VPC** 2 AZs (opcional, `nat_gateways=0` = sem custo).
- Fecha a evolução **console → CLI → script → IaC**.
- Doc completo: [`docs/praticas/18-cdk-iac.md`](docs/praticas/18-cdk-iac.md)
  (`cdk synth` grátis para a aula / Learner Lab; `deploy`/`destroy` em conta própria).

### Aula 12 — Entrega final

- `docs/entrega-final/` — materiais de consolidação:
  - `final-architecture.md` — arquitetura final (as 6 semanas em um diagrama).
  - `final-report-template.md` — template do relatório de entrega.
  - `lgpd-checklist.md` — checklist LGPD + segurança.
  - `deployment-checklist.md` — checklist de deploy + **limpeza de custos**.

---

### Base herdada das semanas 1–5 (resumo)

| Semana | Entregou |
| -----: | --- |
| 1 | FastAPI + Docker + devcontainer |
| 2 | PostgreSQL + CRUD + `.env` + HTTPS (conceito) |
| 3 | Uploads (S3/local) + Kubernetes local (Kind) |
| 4 | Imagem no **ECR** + deploy no **EKS** |
| 5 | **HPA** + custos + eventos (**DynamoDB**/JSON) |

Detalhes de cada entrega anterior nas práticas `docs/praticas/` (00–17) e no
[`docs/ROADMAP.md`](docs/ROADMAP.md).

<details>
<summary><b>Detalhe das aulas anteriores (Semanas 3–4)</b></summary>

#### Aula 5 (revisão) — Upload de arquivos (Amazon S3 + fallback local)

- `app/services/s3_service.py` — dois backends com a **mesma interface**:
  - `LocalStorage` (default): grava em `LOCAL_UPLOADS_DIR` no container.
  - `S3Storage`: envia para o bucket `S3_BUCKET_NAME` (boto3).
  - `get_storage()` escolhe um ou outro a partir de `STORAGE_MODE`.
- `app/api/routes_uploads.py`:
  - `POST /uploads` — recebe `multipart/form-data`, devolve nome + URL.
  - `GET /uploads/{filename}` — local serve do disco; S3 redireciona para URL pré-assinada.
- Limite de **10 MB** por arquivo (config didática).
- Nome de arquivo armazenado é **sanitizado** (sem `..`, com sufixo único) — evita path traversal.
- `app/schemas.py` ganhou `UploadResponse` com exemplos no Swagger.
- `app/core/config.py` ganhou `STORAGE_MODE`, `LOCAL_UPLOADS_DIR`, `AWS_REGION`, `S3_BUCKET_NAME`, `S3_ENDPOINT_URL` (opcional), `S3_PRESIGNED_URL_EXPIRES`.
- Testes (`tests/test_uploads.py`): fluxo feliz, 404, 413, 422, extensão preservada.
- `docs/conceitos/s3-efs-datalake.md` — guia didático S3 × EFS × Data Lake.

### Aula 6 (revisão) — Kubernetes local com Kind

- `infra/k8s/` — manifests:
  - `kind-config.yaml` — cluster Kind de 1 nó com porta `30080` mapeada para o host.
  - `namespace.yaml` — namespace `cloudtask`.
  - `configmap.yaml` — config não-sensível (hostname Postgres, STORAGE_MODE).
  - `secret.example.yaml` — TEMPLATE; copie para `secret.yaml` (gitignored) e preencha.
  - `postgres-deployment.yaml` + `postgres-service.yaml` — Postgres como Pod (sem volume — didático).
  - `api-deployment.yaml` — 2 réplicas da API, init container espera Postgres, probes HTTP.
  - `api-service.yaml` — NodePort `30080`.
  - `kustomization.yaml` — `kubectl apply -k infra/k8s/` aplica tudo.
- Roteiro passo a passo: [`docs/praticas/10-kubernetes-kind-local.md`](docs/praticas/10-kubernetes-kind-local.md).
- **Kind roda no HOST** (não no devcontainer). `kubectl` funciona dos dois lados.

Versão da API ao fim da semana: **`0.4.0`**.

### Base herdada das semanas anteriores
FastAPI + PostgreSQL + CRUD, config `.env`, HTTPS preparado, readiness probe,
testes (transação + savepoint), docker-compose dev/prod/test, devcontainer com
zsh + sticky scroll + transient prompt + AWS CLI, kubectl, eksctl, Node+CDK,
docker-outside-of-docker.

> Todo o código vem com **comentários didáticos** explicando motivo, impacto e
> risco de cada decisão.

</details>

> Todo o código vem com **comentários didáticos** explicando motivo, impacto e
> risco de cada decisão.

## Endpoints

| Método | Caminho               | Descrição |
| ------ | --------------------- | --------- |
| GET    | `/`                   | Metadados da aplicação. |
| GET    | `/health`             | Liveness probe. |
| GET    | `/health/ready`       | Readiness (checa o PostgreSQL). |
| POST   | `/tasks`              | Criar tarefa (201). |
| GET    | `/tasks`              | Listar (paginação `skip`/`limit`). |
| GET    | `/tasks/{task_id}`    | Obter por id (404). |
| PUT    | `/tasks/{task_id}`    | Atualizar parcial. |
| DELETE | `/tasks/{task_id}`    | Remover (204). |
| **POST** | **`/uploads`**          | **Enviar arquivo (multipart, 201)** |
| **GET**  | **`/uploads/{filename}`** | **Baixar (200) ou redirect S3 (307)** |
| GET    | `/docs`               | Swagger UI. |

## Como rodar

> ⚠️ **Ao mudar de semana (branch), faça REBUILD do devcontainer.**
> A imagem do container é um snapshot congelado das dependências da branch em que
> foi construída. Cada semana acrescenta libs novas em `requirements.txt` (ex.:
> `boto3` na Semana 3, `kubernetes` na Semana 6, etc.). Sem rebuild, o `uvicorn`
> vai quebrar com `ModuleNotFoundError` ao tentar importar uma lib que ainda não
> foi instalada e o Swagger sai do ar.
>
> No VS Code: `F1` → **Dev Containers: Rebuild and Reopen in Container**.
>
> Para saber se precisa rebuild antes de trocar de branch:
> ```bash
> git diff <branch-atual> <branch-destino> -- requirements.txt requirements-dev.txt requirements-test.txt Dockerfile docker-compose.yml
> ```
> Se mostrar diff → rebuild. Entre **aulas da mesma semana**, geralmente código
> apenas — não precisa rebuild.

### Devcontainer (recomendado)
`F1` → "Dev Containers: Reopen in Container". A API sobe sozinha em
`http://localhost:8000/docs`.

### Modo local (default — sem AWS)
```bash
# upload (qualquer arquivo)
curl -F "file=@README.md" http://localhost:8000/uploads
# resposta: {"filename":"abcd1234-...md","url":"/uploads/abcd...md","storage_mode":"local"}

# download
curl -O http://localhost:8000/uploads/abcd1234-...md
```

### Modo S3 (precisa de credenciais AWS)
```bash
# 1. criar bucket (uma vez)
aws s3 mb s3://cloudtask-ai-saas-uploads-SEU-NOME --region us-east-1

# 2. configurar .env
echo "STORAGE_MODE=s3" >> .env
echo "S3_BUCKET_NAME=cloudtask-ai-saas-uploads-SEU-NOME" >> .env

# 3. recriar container e testar
docker compose down && docker compose up -d
curl -F "file=@README.md" http://localhost:8000/uploads
# resposta agora traz URL pré-assinada do S3
```

## Testes

```bash
pytest -v
```
41 testes (5 novos de upload). Mode S3 não tem teste automatizado (depende de
credenciais reais ou LocalStack); validar manualmente.

## Como subir na AWS (resumo)
> ⚠️ **Ao mudar de semana (branch), faça REBUILD do devcontainer.**
> A imagem do container é um snapshot congelado das dependências da branch em
> que foi construída. Cada semana acrescenta libs novas em `requirements.txt`.
> Sem rebuild, o `uvicorn` quebra com `ModuleNotFoundError` ao importar uma lib
> que ainda não foi instalada e o Swagger sai do ar.
>
> No VS Code: `F1` → **Dev Containers: Rebuild and Reopen in Container**.
>
> Para saber se precisa rebuild antes de trocar de branch:
> ```bash
> git diff <branch-atual> <branch-destino> -- requirements.txt requirements-dev.txt requirements-test.txt Dockerfile docker-compose.yml
> ```
> Se mostrar diff → rebuild. Entre **aulas da mesma semana**, geralmente
> código apenas — não precisa rebuild.

**Nunca usou terminal, Docker ou AWS?** Comece pelo guia do absoluto zero —
instalação de todas as ferramentas e configuração do AWS Academy Learner Lab:
[`docs/praticas/00-setup-inicial-e-aws-academy.md`](docs/praticas/00-setup-inicial-e-aws-academy.md).

Resumo:

```bash
# 1. credenciais Learner Lab no host (~/.aws/credentials)
aws sts get-caller-identity

# 2. Aula 7 — publicar imagem no ECR
./scripts/semana-04-ecr/build-push-ecr.sh

# 3. Aula 8 — cluster EKS (~15 min) + deploy
eksctl create cluster --name cloudtask-eks --region us-east-1 \
  --node-type t3.small --nodes 2 --managed
kubectl apply -k infra/k8s/aws/
kubectl get svc -n cloudtask api -w   # esperar o ELB ficar pronto
```

> ⚠️ **Custo:** EKS cobra ~$0,10/h + 2 nós EC2 + ELB. **Sempre destrua ao fim:**
> ```bash
> kubectl delete -k infra/k8s/aws/
> eksctl delete cluster --name cloudtask-eks --region us-east-1
> ```

Roteiros mastigados: [`docs/praticas/11-ecr-push.md`](docs/praticas/11-ecr-push.md) → [`docs/praticas/12-eks-deploy.md`](docs/praticas/12-eks-deploy.md).

## Infra como Código (Aula 11) e entrega final (Aula 12)

```bash
# IaC com CDK (dentro de infra/cdk/) — synth é grátis (ótimo p/ aula)
cd infra/cdk && pip install -r requirements.txt
cdk synth                 # gera o CloudFormation sem criar nada
cdk deploy --all          # (conta própria) cria S3 + ECR + VPC
cdk destroy --all         # 🔥 apaga tudo
```
Cada branch `aula-XX-final` contém **somente o estado acumulado até aquela aula** e funciona de forma independente.

## Participantes

| [<img src="https://avatars3.githubusercontent.com/u/60905310?s=460&v=4" width="75px;"/>](https://github.com/guipatriota) |
| :------------------------------------------------------------------------------------------------------------------------: |
| [Prof. Guilherme Patriota](https://github.com/guipatriota) |

## Estrutura final do projeto (referência — aula 12)

```text
cloudtask-ai-saas/
├── app/
│   ├── main.py
│   ├── core/            # config, security
│   ├── api/             # routes_health, routes_tasks, routes_uploads, routes_events
│   ├── db/              # database, models, schemas
│   ├── services/        # s3_service, dynamodb_service, task_service
│   └── utils/           # logging
├── infra/
│   ├── docker/
│   ├── k8s/             # local + aws/
│   └── cdk/             # stacks S3, ECR, VPC
├── scripts/             # build-and-push-ecr.sh, load-test
├── tests/
├── docs/                # ROADMAP, HOW_TO_USE, arquitetura, LGPD, etc.
├── Dockerfile
├── docker-compose.yml
├── requirements.txt
├── .env.example
└── README.md
```

> Em branches anteriores à aula 12, só existem as pastas e arquivos correspondentes ao conteúdo já visto.

## Contribuições

Se você tiver alguma sugestão, correção de bugs ou melhorias para este projeto didático, sinta-se à vontade para abrir uma issue ou enviar uma pull request. Sua contribuição é muito bem-vinda!

Entrega final: preencha [`docs/entrega-final/final-report-template.md`](docs/entrega-final/final-report-template.md)
e rode os checklists de [LGPD](docs/entrega-final/lgpd-checklist.md) e
[deploy/custos](docs/entrega-final/deployment-checklist.md).

## Fim da disciplina 🎓

Esta é a **última semana**. A consolidação está em
[`docs/entrega-final/final-architecture.md`](docs/entrega-final/final-architecture.md)
(as 6 semanas em um diagrama) e no [`docs/ROADMAP.md`](docs/ROADMAP.md).

## Referências

- Issues da semana: [#11 — Aula 11 (CDK)](https://github.com/N-CPUninter/Computa-o-em-Nuvem---Projeto-exemplo-CloudTask-AI-SaaS/issues/11) · [#12 — Aula 12 (final)](https://github.com/N-CPUninter/Computa-o-em-Nuvem---Projeto-exemplo-CloudTask-AI-SaaS/issues/12)
- **CDK / IaC (Aula 11)**: [`docs/praticas/18-cdk-iac.md`](docs/praticas/18-cdk-iac.md) + stacks em `infra/cdk/`
- **Entrega final (Aula 12)**: [`docs/entrega-final/`](docs/entrega-final/README.md)
- 📜 **Scripts (mapa por semana)**: [`scripts/README.md`](scripts/README.md)
- Lista de tarefas: [`docs/TAREFAS.md`](docs/TAREFAS.md)
- Setup do zero: [`docs/praticas/00-setup-inicial-e-aws-academy.md`](docs/praticas/00-setup-inicial-e-aws-academy.md)
- **ECR**: [`docs/praticas/11-ecr-push.md`](docs/praticas/11-ecr-push.md) + `scripts/semana-04-ecr/build-push-ecr.sh` + `buildspec.yml`
- **EKS**: [`docs/praticas/12-eks-deploy.md`](docs/praticas/12-eks-deploy.md) + manifests em `infra/k8s/aws/`
- **Kubernetes Kind (Aula 6)**: [`docs/praticas/10-kubernetes-kind-local.md`](docs/praticas/10-kubernetes-kind-local.md) + manifests em `infra/k8s/`
- **S3 (Aula 5)**: [`docs/conceitos/s3-efs-datalake.md`](docs/conceitos/s3-efs-datalake.md)
- **Roteiro Aula 3+4 (semanas combinadas)**: [`docs/praticas/13-roteiro-aula-semanas-3-e-4.md`](docs/praticas/13-roteiro-aula-semanas-3-e-4.md)
- **Stack AWS por semana** (custos, Postgres container × RDS, ECS × EKS): [`docs/conceitos/infra-aws-minima-por-semana.md`](docs/conceitos/infra-aws-minima-por-semana.md)
- **Deploy manual AWS** (ECR, Fargate, EKS, RDS, Secrets Manager, CodeBuild): [`docs/praticas/09-deploy-manual-aws.md`](docs/praticas/09-deploy-manual-aws.md)
- Segurança: [`docs/conceitos/security-model.md`](docs/conceitos/security-model.md) · [`docs/conceitos/aws-networking.md`](docs/conceitos/aws-networking.md) · [`docs/conceitos/https-tls.md`](docs/conceitos/https-tls.md)
- Docker: [`docs/conceitos/docker-explained.md`](docs/conceitos/docker-explained.md)

## Licença

[GNU General Public License v3.0](LICENSE).
