# ubichill demo 環境

`https://demo.ubichill.com`。外部の人に触ってもらうためのデモ環境。

- Application: `argocd-apps/ubichill-demo.yaml`（Secret は `argocd-apps/ubichill-demo-secrets.yaml`）
- 構成: `values-dev.yaml`（メール確認なし）＋ DB の永続化。Go は 1 Pod（Redis の Directory・KEDA なし）
- イメージ: 本番と同じリリース版（`vX.Y.Z`。`apps/argocd/image-updater.yaml`）
- 連合: `WORLDS_FEDERATION_PEERS=https://ubichill.com`（本番の公式ワールドが一覧に出る）

## アカウント

| | ID | パスワード |
|---|---|---|
| 公式アカウント（管理者） | `ubichill` | PR のプレビュー・dev と同じ（`official-account.yaml`。`ubichill-demo-secrets` の値より優先） |
| デモ用 | `demo` | 手元の `ubichill-demo-account.env`（Git には入れない） |

デモ用のアカウントは、デプロイ後に登録 API で作った（メール確認なしなのでそのまま使える）:

```sh
curl -sS https://demo.ubichill.com/api/v1/users/register -H 'Content-Type: application/json' \
  -d '{"email":"demo@demo.ubichill.com","password":"...","displayName":"ゲスト","handle":"demo"}'
```

## Secret の更新

```sh
# クラスタの API に届かないときは ssh k3s-1 で公開鍵を取る（create-sealed-secret.sh と同じ）
kubectl create secret generic ubichill-demo-secrets -n ubichill-demo --from-env-file=ubichill-demo.env --dry-run=client -o yaml \
  | kubeseal --cert /tmp/sealed-secrets.crt --scope strict -o yaml > apps/ubichill-demo/ubichill-demo-secrets.enc.yaml
```

`POSTGRES_PASSWORD`・`REDIS_PASSWORD` は最初の起動で PVC に書かれるので、あとから変えるときは PVC も作り直す。

## 片付け

要らなくなったら `argocd-apps/ubichill-demo*.yaml`・`apps/ubichill-demo/`・Image Updater の `ubichill-demo` を消す。
子の Application に resources-finalizer が無いので、Namespace `ubichill-demo`（PVC ごと）は手で消す。
