---
status: accepted
date: 2026-09-27
---

# Rust のコードから OpenAPI を生成し、TypeScript のクライアントを生成して型の連鎖をつなぐ

言語の境界で型が途切れないように、次の連鎖を 1 本につなぐ。

1. sqlx で、DB から Rust まで
2. utoipa（utoipa-axum と一緒に使う）で、Rust から OpenAPI まで
3. openapi-typescript + openapi-fetch で、OpenAPI から TypeScript まで

Rust のコードを正とする。生成した OpenAPI の仕様はコミットし、CI で再生成したものとの差分と、TypeScript 側の型チェックを検査する。これで「Rust を変えたら、フロントエンドの壊れた箇所がコンパイルエラーになる」状態を保つ。

## Considered Options

| 案 | 見送った理由 |
|---|---|
| aide | ハンドラの型から仕様を作るので、注釈とのずれが少なく設計としてはきれい。ただし利用者と AI の学習データが少ない |
| ts-rs / specta | 型は共有できるが、URL とメソッドは手で合わせることになり、エンドポイントがずれうる |
| OpenAPI の仕様を先に手で書き、両側を生成する | Rust のサーバーのコードを生成するツールが弱く、実装が仕様からずれうる |
| Hey API / orval（クライアントの生成） | TanStack Query のフックまで生成できるが、生成されるコードが多く、AI が読む量と差分をレビューする量が増える。フックは薄いラッパーを自分で書く |
