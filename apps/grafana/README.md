# Grafana ダッシュボード

`dashboards/*.json` を Git で管理し、Kustomize で ConfigMap に変換する。
Argo CD の `grafana` Application が適用し、Grafana の dashboard sidecar が
`monitoring` 内の `grafana_dashboard: "1"` ラベル付き ConfigMap を読み込む。
ConfigMap の `grafana_folder` アノテーションで表示先のフォルダを指定し、
JSON の変更も自動で反映される。

## Ubichill / Instance

- ファイル: `dashboards/ubichill-instance.json`
- フォルダ: `Ubichill`
- URL: https://grafana.youkan.uk/d/ubichill-instance/ubichill-instance
- 対象: `namespace="ubichill", service="ubichill-instance"`
- 表示: 部屋数の合計、Pod ごとの部屋数と推移、受け入れ可能な部屋数、WebSocket 接続数、メトリクス収集状態
- 更新間隔: 15 秒。Pod セレクタで対象を絞り込める。

既存の `Prometheus` データソースを datasource 変数で参照する。
データソースの UID を変更せず、既存のダッシュボードでも引き続き利用できる。
Prometheus 接続は `argocd-apps/grafana.yaml`、収集は
`argocd-apps/ubichill.yaml` の `instanceRuntime.metrics.serviceMonitor.enabled` で管理する。

`ubichill_instance_active_rooms` は各 Go instance Pod が所有する部屋数。
Pod ラベルは ServiceMonitor による収集時に付与される。
`ubichill_instance_websocket_connections` は中継元・所有先の両方で接続を数えるため、
ユニークな利用者数とは異なる。データ未取得を 0 部屋に置き換えず、
収集失敗は `up` のパネルで確認できる。

## 電気代

- ファイル: `dashboards/electricity.json`
- フォルダ: `Home`
- UID: `yotc6lm`（既存ダッシュボードと同じ UID で更新する）
- データソース: 既存の InfluxDB（UID `P951FEA4DE68E13C5`）
- 表示: 一日あたりのサーバー電気代、月サーバー電気代予測、電力の推移

提供された JSON のクエリ・単価（40 円/kWh）・レイアウトを使用する。
データベース内で採番される `id` のみ `null` にし、ダッシュボードの UID と
データソースの UID は維持している。

## 変更方法

JSON を編集してコミット・push すると、Argo CD の同期後に反映される。
Grafana の画面からの保存は無効。画面で編集した内容を採用する場合は、
ダッシュボードの JSON をエクスポートして対象ファイルを更新する。
各ダッシュボードの UID を維持すると URL も維持される。

追加する場合は JSON を配置し、`kustomization.yaml` の `configMapGenerator` に登録する。
`options.annotations.grafana_folder` に `/tmp/dashboards/<フォルダ名>` を指定する。
ConfigMap を削除すると、対応するダッシュボードも削除される。
