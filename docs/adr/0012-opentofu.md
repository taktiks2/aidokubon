---
status: accepted
date: 2026-09-27
---

# インフラを OpenTofu で管理する

GCP（Cloud Run、Artifact Registry、Secret Manager、IAM、Workload Identity Federation、Firebase Hosting のサイト）と Neon を、コードで管理する。ツールは OpenTofu にする。書き方とプロバイダーが Terraform と共通なので、AI の学習データの多さを活かせる。ライセンスの心配もない。状態は GCS のバケットに置き、このバケットだけは最初に手で作る。

## Considered Options

| 案 | 見送った理由 |
|---|---|
| Terraform | 機能はほぼ同じだが、ライセンスが BSL。個人で使う分には問題ないが、選ぶ利点もない |
| Pulumi（TypeScript） | 型の検査が効くのは魅力だが、Neon を扱うには Terraform のプロバイダーを変換する一手間が要る。IaC の規模は小さく変更も少ないので、`tofu plan` の差分をレビューすれば十分に守れる |

## Consequences

- 秘密の値（楽天のアクセスキー、OAuth のクライアントシークレットなど）は、state ファイルに平文で残さないように、Secret Manager の入れ物だけを OpenTofu で作る。値は別の手段で入れる。
- Neon のプロバイダーはコミュニティ製（`kislerdm/neon`）なので、採用する前に保守の状況を確認する。
- IaC で管理できないもの（ドメインの取得、楽天のアプリ登録、Google の OAuth クライアントと同意画面）は、手順を README に残す。
