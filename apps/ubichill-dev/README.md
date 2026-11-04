# ubichill dev 環境

`https://dev.ubichill.com`。本番（`ubichill`）と同じ構成で、環境変数だけ development。負荷試験とスケーリングの評価に使う。

| | 本番 `ubichill` | dev `ubichill-dev` | プレビュー `preview-pr-<n>` |
|---|---|---|---|
| Application | `argocd-apps/ubichill.yaml` | `argocd-apps/ubichill-dev.yaml` | `argocd-apps/ubichill-preview.yaml`（ApplicationSet） |
| Secret | `apps/ubichill/`（SealedSecret） | `apps/ubichill-dev/`（SealedSecret、本番とは別の値） | 固定の開発用の値 |
| イメージ | `vX.Y.Z`（release） | `main-<sha>`（main の最新。`apps/argocd/image-updater.yaml`） | `sha-<PR の sha>` |
| DB・Redis | 永続（proxmox-data） | 永続（proxmox-data）。PR と違い消えない | 消える |
| `NODE_ENV` | production | development（メール確認なし・未署名の同梱ワールド可・本番と連合） | development |
| Go | Redis の Directory・KEDA（上限 3） | 同じ。生成器と別のノード（`ubichill.com/loadgen` の無いノード） | 2 Pod |
| Redis の指標 | なし | redis_exporter + ServiceMonitor | なし |

## 負荷試験

- 生成器は `ubichill.com/loadgen=true` のノード（今は `k3s-worker-1`。`kubectl label node k3s-worker-1 ubichill.com/loadgen=true`）に置く。dev の Go はこのノードに置かない（`instanceRuntime.affinity`）。
  本番の backend・frontend も `k3s-worker-1` に居るので、生成器には CPU の limit を付ける（chart の既定 1 コア/Pod）。
- 走らせ方・見方は ubichill の `docs/operations/load-testing.md`。結果は Grafana の `Ubichill / Load test`（`apps/grafana/dashboards/ubichill-loadtest.json`）。

## Secret の更新

```sh
kubeseal --fetch-cert --controller-namespace kube-system --controller-name sealed-secrets > /tmp/sealed-secrets.crt
kubectl create secret generic ubichill-dev-secrets -n ubichill-dev --dry-run=client -o yaml \
  --from-literal=BETTER_AUTH_SECRET=... --from-literal=INSTANCE_ADMIN_TOKEN=... \
  --from-literal=OFFICIAL_ACCOUNT_PASSWORD=... --from-literal=POSTGRES_PASSWORD=... \
  --from-literal=REDIS_PASSWORD=... --from-literal=RESEND_API_KEY=dev-resend-key \
  --from-file=SERVICE_TOKEN_SIGNING_KEY=<Ed25519 の PKCS8 PEM> \
  | kubeseal --cert /tmp/sealed-secrets.crt --scope strict -o yaml > apps/ubichill-dev/ubichill-dev-secrets.enc.yaml
```

`INSTANCE_ADMIN_TOKEN` は values-dev.yaml の `instanceRuntime.secretKey` がこの名前を指すので要る（本番は `BETTER_AUTH_SECRET` を兼用）。`POSTGRES_PASSWORD`・`REDIS_PASSWORD` は最初の起動で PVC に書かれるので、あとから変えるときは PVC も作り直す。
