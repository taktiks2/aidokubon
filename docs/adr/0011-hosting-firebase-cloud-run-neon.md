---
status: accepted
date: 2026-09-27
---

# Firebase Hosting + Cloud Run + Neon で動かし、SPA と API を同じオリジンに置く

SPA は Firebase Hosting に置き、`/api/**` へのリクエストを Cloud Run の API に転送する（rewrite）。こうすると SPA と API が同じオリジンになり、CORS の設定も、Cookie の SameSite の考慮も要らなくなる。DB は Neon の PostgreSQL にする。API と DB は同じリージョン（シンガポール）に置く。1 回の API の呼び出しで SQL を複数回投げるので、DB までの往復時間が SQL の回数だけ積み重なるため。家族で使う規模では、どれも無料枠に収まる。

## Considered Options

| 案 | 見送った理由 |
|---|---|
| Cloud Run + Cloudflare Pages | 契約するサービスが 3 つになる。オリジンが別になり、CORS の設定が要る |
| AWS（CloudFront + S3 + Lambda Function URL） | 月 100 円前後で動き、IaC も最も成熟している。ただし Lambda は 1 つのインスタンスが 1 リクエストしかさばかないので、利用者が増えると DB への接続が増える。Lambda 向けの作りになり、コンテナより移しにくい |
| AWS（ECS Fargate） | 月 3,000 円以上かかる |
| Cloudflare Workers（Rust を Wasm にする） | sqlx が Wasm で動かないので、ADR-0005 の SQL のコンパイル時の検査を失う |
| Cloudflare Containers | 月 5 ドルからかかる。サービスが新しく、DB との距離を制御しにくい可能性がある |
| Cloud SQL / RDS | 無料枠がなく、最小の構成でも月に 10〜20 ドルほどかかる |
| Supabase（東京）+ Cloud Run（東京） | 速さでは上だが、無料枠だと 1 週間アクセスがないとプロジェクトが一時停止する |

## Consequences

- Neon は AWS 上で動いているので、Cloud Run からはクラウドをまたいだ通信になる。利用者が増えて遅延が問題になったら、DB を Cloud SQL に移し、API も DB と同じリージョンに置き直す（ADR-0013 の約束があれば、小さな作業で済む）。
- 実装する前に確認すること：Firebase Hosting からシンガポールの Cloud Run へ転送できるか、各サービスの無料枠の最新の条件。
