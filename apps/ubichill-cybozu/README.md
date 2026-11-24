# ubichill cybozu 環境（面接用）

`https://cybozu.ubichill.com`。サイボウズの面接で採用担当に触ってもらうための環境。

- Application: `argocd-apps/ubichill-cybozu.yaml`（Secret は `argocd-apps/ubichill-cybozu-secrets.yaml`）
- 構成: `values-dev.yaml`（メール確認なし）＋ DB の永続化。Go は 1 Pod（Redis の Directory・KEDA なし）
- イメージ: 本番と同じリリース版（`vX.Y.Z`。`apps/argocd/image-updater.yaml`）
- 連合: `WORLDS_FEDERATION_PEERS=https://ubichill.com`（本番の公式ワールドが一覧に出る）

## アカウント

| | ID | パスワード |
|---|---|---|
| 公式アカウント（管理者） | `ubichill` | Secret の `OFFICIAL_ACCOUNT_PASSWORD` |
| 採用担当 | `cybozu` | 手元の `ubichill-cybozu-recruiter.env`（Git には入れない） |

採用担当のアカウントは、デプロイ後に登録 API で作った（メール確認なしなのでそのまま使える）:

```sh
curl -sS https://cybozu.ubichill.com/api/v1/users/register -H 'Content-Type: application/json' \
  -d '{"email":"cybozu@cybozu.ubichill.com","password":"...","displayName":"cybozu","handle":"cybozu"}'
```

## Secret の更新

```sh
# クラスタの API に届かないときは ssh k3s-1 で公開鍵を取る（create-sealed-secret.sh と同じ）
kubectl create secret generic ubichill-cybozu-secrets -n ubichill-cybozu --from-env-file=ubichill-cybozu.env --dry-run=client -o yaml \
  | kubeseal --cert /tmp/sealed-secrets.crt --scope strict -o yaml > apps/ubichill-cybozu/ubichill-cybozu-secrets.enc.yaml
```

`POSTGRES_PASSWORD`・`REDIS_PASSWORD` は最初の起動で PVC に書かれるので、あとから変えるときは PVC も作り直す。

## 片付け

面接が終わったら `argocd-apps/ubichill-cybozu*.yaml`・`apps/ubichill-cybozu/`・Image Updater の `ubichill-cybozu` を消す（PVC は手で消す）。
