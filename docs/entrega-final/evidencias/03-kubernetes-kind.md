# Evidências — Kubernetes funcionando (Kind local)

> Cluster **Kubernetes real** rodando localmente com **Kind** (Kubernetes-in-Docker),
> sem custo de nuvem. Atende ao entregável "Kubernetes funcionando" do AVA.
> Data: 04/07/2026. Kind v0.32.0, node image kindest/node:v1.31.4.

## Como foi subido

```bash
kind create cluster --name cloudtask --config infra/k8s/kind-config.yaml \
     --image kindest/node:v1.31.4
docker build --target prod -t cloudtask-api:prod .
kind load docker-image cloudtask-api:prod --name cloudtask
kubectl apply -k infra/k8s/          # namespace, configmap, secret, postgres, api
```

## 1. Pods, deployments e services (Running)

```text
$ kubectl get pods -n cloudtask
NAME                        READY   STATUS    RESTARTS   AGE
api-547df9d8ff-gvgf4        1/1     Running   0          39s
api-547df9d8ff-ptwzt        1/1     Running   1          3m17s
postgres-79f9c9475c-6j6wc   1/1     Running   0          3m17s

$ kubectl get deployments -n cloudtask
NAME       READY   UP-TO-DATE   AVAILABLE
api        2/2     2            2
postgres   1/1     1            1

$ kubectl get svc -n cloudtask
NAME       TYPE        CLUSTER-IP     PORT(S)
api        NodePort    10.96.84.107   8000:30080/TCP
postgres   ClusterIP   10.96.11.134   5432/TCP
```

A API roda com **2 réplicas** (alta disponibilidade) e o Postgres como Pod.

## 2. Aplicação respondendo PELO cluster (NodePort 30080)

```text
GET http://localhost:30080/              -> {"name":"TaskFlow AI","version":"0.6.0","docs":"/docs"}
GET http://localhost:30080/health        -> {"status":"ok"}
GET http://localhost:30080/health/ready  -> {"status":"ready","db":"ok"}
```

O NodePort `30080` do Service é mapeado para o host pelo `kind-config.yaml`,
então o Swagger fica acessível em `http://localhost:30080/docs`.

## 3. Auto-healing (o Kubernetes recria pods sozinho)

```text
$ kubectl delete pod api-547df9d8ff-lj868 -n cloudtask
pod "api-547df9d8ff-lj868" deleted

$ kubectl get pods -n cloudtask -l app=api   # logo em seguida
NAME                   READY   STATUS     AGE
api-547df9d8ff-gvgf4   0/1     Init:0/1   1s     <- NOVO pod criado automaticamente
api-547df9d8ff-ptwzt   1/1     Running    2m39s

# segundos depois: de volta a 2/2 Running (auto-healing concluído)
```

Ao deletar um pod, o Deployment **recria automaticamente** para manter as 2
réplicas — demonstração de **self-healing** e **alta disponibilidade** do Kubernetes.

## Conceitos demonstrados (Slide 12 do AVA)

- **Containers** (Docker) orquestrados por **Kubernetes**.
- **Alta disponibilidade** (2 réplicas) e **auto-healing** (recriação de pods).
- **Escalabilidade** (basta `kubectl scale`/HPA para mais réplicas).
- Separação **config** (ConfigMap) × **segredos** (Secret).

> Observação de honestidade: este é Kubernetes rodando **localmente (Kind)**, que é
> Kubernetes real. O deploy do MESMO conjunto em **EKS (nuvem)** está descrito nos
> manifests `infra/k8s/aws/` e nas stacks CDK; a evidência de nuvem é tratada
> separadamente (deploy em conta AWS).
