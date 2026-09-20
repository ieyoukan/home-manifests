# MediaMTX for VRChat

OBSからMediaMTXへRTMPで入力し、PC版VRChatにはRTSP over TCP、Quest/AndroidにはHTTPS HLSで配信する。

## OBS

- サービス: カスタム
- サーバー: `rtmp://192.168.0.221/vrchat`
- ストリームキー: `youkan`
- 映像: H.264
- 音声: AAC
- キーフレーム間隔: 1秒
- Bフレーム: 0（設定できる場合）

Bearer Tokenやユーザー認証は不要。`live.youkan.uk`はCloudflare経由のHLS視聴用なので、RTMP入力先には使わない。

## VRChat

- PC低遅延: `rtspt://rtsp.live.youkan.uk/vrchat/youkan`
- Quest/Android: `https://live.youkan.uk/vrchat/youkan/index.m3u8`

ワールドでは`VRCAVProVideoPlayer`を使い、`Low Latency`を有効にする。独自ドメインを使うため、視聴者側の`Allow Untrusted URLs`と、Public/Group Publicインスタンスではワールド設定の`Video Player Allowed Domains`も必要になる。

## ネットワーク

- ルーター: WAN TCP/554を`192.168.0.221:554`へ転送する。Serviceがコンテナ内のTCP/8554へ変換する。
- DNS: `live.youkan.uk`はCloudflare経由のHLS用として維持する。
- DNS: `rtsp.live.youkan.uk`を追加し、Cloudflareのプロキシを無効（DNS only）にして自宅のグローバルIPへ向ける。
- TCP/1935はLAN内のOBS入力専用とし、WANには公開しない。
- HLSは既存のCloudflare Tunnel/Ingress経由で公開する。

RTMP publish認証は無効になっている。TCP/1935をWANへ公開すると第三者が配信できるため、必ずLAN内に限定する。

公開RTSPは暗号化されない。配信内容や秘匿性が重要な用途では、自宅から直接公開せず、VRChat向け配信サービスまたは中継VPSを使う。
