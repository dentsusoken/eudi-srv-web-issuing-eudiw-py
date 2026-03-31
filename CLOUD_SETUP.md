# クラウド環境セットアップ

## 0. 前提条件

以下のツールが事前にインストールされていること。

| ツール | 用途 | インストール |
|---|---|---|
| Docker | コンテナ実行 | [docs.docker.com](https://docs.docker.com/engine/install/) |
| openssl | 鍵・証明書生成 | `brew install openssl` |
| keytool | JKS キーストア生成 | JDK に同梱 (`brew install openjdk`) |
| python3 | JWK 更新スクリプト実行 | `brew install python` |

また以下が用意されていること。

- Issuer、AS、フロントエンドそれぞれのドメイン
- ALB の設定と ACM 証明書（TLS は ALB で終端）

## 1. ディレクトリ構成

```
<作業ディレクトリ>/
├── eudi-srv-issuer-oidc-py/
│   └── script/
│       ├── setup-certs.sh
│       ├── generate_local.sh
│       ├── certs/                      ← setup-certs.sh で生成
│       │   ├── trusted_cas/            ← setup-certs.sh で生成
│       │   ├── privKey/                ← setup-certs.sh で生成
│       │   └── verifier/               ← setup-certs.sh で生成
│       └── patches/
├── eudi-srv-web-issuing-eudiw-py/
└── eudi-srv-web-issuing-frontend-eudiw-py/
```

## 2. `_local` ファイルの生成

```bash
bash eudi-srv-issuer-oidc-py/script/generate_local.sh
```

## 3. 証明書・鍵ファイルの生成

`--cloud` フラグを付けることで mkcert および iOS Simulator 関連の処理をスキップする。

```bash
bash eudi-srv-issuer-oidc-py/script/setup-certs.sh --cloud
```

以下が生成される：

- `certs/trusted_cas/` — IACA / DS 証明書
- `certs/privKey/` — 秘密鍵
- `certs/verifier/keystore.jks` — Verifier キーストア
- `metadata_config_local.json` の JWK が新しい公開鍵で自動更新される

## 4. 設定ファイルの編集

各リポジトリの `_local` ファイルをコピーし、以下のパラメータを書き換える。

### eudi-srv-issuer-oidc-py

**`config_local.json` を編集**

| パラメータ | 変更内容 |
|---|---|
| `domain` | AS のクラウドドメイン（例: `as.example.com`） |
| `base_url` | AS のクラウド URL（例: `https://as.example.com`） |
| `authorization_redirect_url` | Issuer のクラウド URL + `/auth_choice` |
| `op.server_info.add_ons.dpop.kwargs.allowed_htu` | Issuer と AS のクラウド URL |
| `webserver.server_cert` | 削除または `null` |
| `webserver.server_key` | 削除または `null` |

**`openid-configuration_local.json` を編集**

`localhost:6005` を AS のクラウド URL に一括置換する。

### eudi-srv-web-issuing-eudiw-py

**`app/.env.local` をコピーして編集**

| パラメータ | 変更内容 |
|---|---|
| `SERVICE_URL` | Issuer のクラウド URL |
| `DEFAULT_FRONTEND_URL` | フロントエンドのクラウド URL |
| `AUTH_SERVER_INTERNAL_URL` | AS のクラウド URL |
| `VERIFY_USER_ENDPOINT` | AS のクラウド URL + `/verify/user` |
| `SAML_SP_ENTITY_ID` | Issuer のクラウド URL |
| `SAML_SP_ACS_URL` | Issuer のクラウド URL + `/saml/acs` |
| `FLASK_TLS_OPTS` | 空文字（設定しない） |

**`app/metadata_config/openid-configuration_local.json` を編集**

`localhost:6005` を AS のクラウド URL に一括置換する。

> `metadata_config_local.json` のエンドポイント URL は `SERVICE_URL` の値で自動的に置換されるため手動変更は不要。

### eudi-srv-web-issuing-frontend-eudiw-py

**`.env.local` をコピーして編集**

| パラメータ | 変更内容 |
|---|---|
| `SERVICE_URL` | フロントエンドのクラウド URL |
| `ISSUER_URL` | Issuer のクラウド URL |
| `OAUTH_URL` | AS のクラウド URL |
| `FLASK_TLS_OPTS` | 空文字（設定しない） |

## 5. コンテナの起動

```bash
cd eudi-srv-web-issuing-eudiw-py
docker compose --profile local up -d
```

---

> **Note on order:** `generate_local.sh` は `setup-certs.sh` より先に実行すること。
> 逆順で実行すると `generate_local.sh` が `setup-certs.sh` で更新した JWK を上書きしてしまう。
