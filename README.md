# Course Catalog - Deploy

Repositório de deploy da aplicação [Course Catalog](https://github.com/4linux/simplePythonFlask), no modelo **GitOps**.

Aqui está descrito o estado desejado de cada ambiente. O **Argo CD** observa este repositório e aplica no cluster Kubernetes o que estiver na branch `main`.

## Estrutura

```
.
├── base/                   # manifests comuns a todos os ambientes
│   ├── kustomization.yaml
│   ├── mariadb.yaml        # Secret, PVC, Deployment e Service do banco
│   └── web.yaml            # Secret, Deployment, Service e Ingress da aplicação
└── overlays/
    ├── homolog/            # namespace homolog, 1 réplica
    └── production/         # namespace production, 2 réplicas
```

## Ambientes

|Ambiente|Namespace|Endereço|
|---|---|---|
|Homologação|`homolog`|http://homolog.192.168.88.30.nip.io|
|Produção|`production`|http://production.192.168.88.30.nip.io|

## Como uma versão chega a um ambiente

A versão da aplicação em cada ambiente é a linha `newTag` do arquivo `kustomization.yaml` do overlay:

```yaml
images:
  - name: course-catalog
    newName: 192.168.88.20:8082/course_catalog
    newTag: "0.1"
```

- **Homologação**: o pipeline de CI altera o `newTag` e envia o commit direto para a `main`.
- **Produção**: o pipeline abre um Pull Request alterando o `newTag`. A versão só entra após a aprovação.

Para voltar uma versão, reverta o commit que a promoveu.

## Validando localmente

Os comandos abaixo mostram os manifests finais de cada ambiente, sem aplicar nada no cluster:

```sh
kubectl kustomize overlays/homolog
kubectl kustomize overlays/production
```
