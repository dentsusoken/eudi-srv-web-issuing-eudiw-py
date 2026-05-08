# VC認証で利用するCredentialの変更方法

**[English](VP_AUTH_SETUP.md)**

VC認証を利用してクレデンシャルを発行する際に、認証用に要求するVCを変更するための最短手順です。

## 前提条件

- Issuerの[LOCAL_SETUP](../LOCAL_SETUP.md)を実施済みであること。
- Verifierの[LOCAL_SETUP](https://github.com/dentsusoken/eudi-srv-verifier-endpoint/blob/main/LOCAL_SETUP.md)を実施済みであること。（Verifierをローカルで起動する場合）

## 用語の整理

- `credential_configurations_supported` (`credentialsSupported`)  
  -> **当Issuerが扱うことの可能なCredentialの一覧**
- `credential_auth_methods.PID_login`  
  -> **VC認証を利用して発行可能なクレデンシャルの一覧**（EUDI側でPID固定での認証が想定されているため`PID_login`という名称になっています。EUDI側との互換性のため名称は変更していません。）

## 実施手順（推奨順）

1. 要求したいCredential IDを `credentialsSupported` に定義する（すでに定義済みの場合はスキップ）
2. VC認証を利用して発行させたいCredential IDだけ `PID_login` に許可する（すでに許可済みの場合はスキップ）
3. Verifier の向け先（`dynamic_presentation_url`）を環境に合わせる（後述のセクション 3）
4. `oid4vp_credentials_requested` をセッションまたはYAMLで指定する
5. `issuer_backend_local` を再起動する

## 1) 要求可能なCredentialを定義する（`credentialsSupported`）

> [!NOTE]
> 本手順は新しい種類のCredentialを追加する場合のみ実施します。すでに定義済みのCredentialを利用する場合はスキップしてください。

以下の追加先に、Credential定義のJSONを追加します。

追加先:

- `app/metadata_config/credentials_supported/*.json`

最小例:

```json
{
  "jp.ac.aaa-university.student_id": {
    "format": "dc+sd-jwt",
    "scope": "jp.ac.aaa-university.student_id",
    "vct": "https://jp.ac.example-university/vct/student-id",
    "credential_metadata": {
      "claims": [
        { "path": ["given_name"] },
        { "path": ["family_name"] }
      ]
    }
  }
}
```

## 2) VC認証を利用して発行可能なCredentialを許可する（`credential_auth_methods.PID_login`）

以下のファイルの`credential_auth_methods.PID_login`の設定項目に、VC認証を利用して発行可能なCredentialを追加します。

`app/config_issuer_backend_local.yaml`:

```yaml
credential_auth_methods:
  PID_login:
    - eu.europa.ec.eudi.pid_mdoc
    - jp.ac.aaa-university.student_id
```

## 3) Verifier の向け先（`dynamic_presentation_url`）

> [!NOTE]
> Verifier の向け先は、デフォルトでローカルのDockerで起動するVerifier BackendのURLを指定しています。
> 必要な場合のみ変更してください。

`/oid4vp` は verifier の `/ui/presentations` に対してプレゼンテーションを開始するため、`dynamic_presentation_url` でそのベースURLを指定します。

設定場所:

- `app/config_issuer_backend_local.yaml` の `dynamic_presentation_url`

例（ローカル開発で verifier がホストの **8080** で待ち受けている場合）:

```yaml
dynamic_presentation_url: "http://host.docker.internal:8080/ui/presentations"
```

- Issuer を Docker で動かしていると、`localhost` はコンテナ自身を指すことが多く接続に失敗します。その場合は **`host.docker.internal`** でホスト側のポートに届ける構成がよく使われます。
- ホストから直接 Issuer を動かす場合や、Verifier と同一 Docker ネットワークでサービス名解決できる場合は、`http://localhost:8080/...` や `http://verifier-backend-local:8080/...` など、環境に合わせて変更してください。
- クラウドの verifier を使う場合は、その公開URL（例: `https://verifier-backend.eudiw.dev/ui/presentations`）に差し替えます。

変更後は `issuer_backend_local` を再起動してください。  
（`.env.local` の `DYNAMIC_PRESENTATION_URL` は参照されず、実際に効くのはこの YAML の値です。）

## 4) VC認証で使用するVCの種類を指定する

VC認証で使用するVCの種類を指定します。
指定方法は、Credential Offer APIのリクエスト時に指定する方法とYAMLで指定する方法の2種類があります。

### A. Credential Offer APIのリクエスト時に指定

`POST /credential_offer2`:

```json
{
  "credential_configuration_id": "eu.europa.ec.eudi.pid_mdoc",
  "oid4vp_credentials_requested": [
    "jp.ac.aaa-university.student_id",
    "eu.europa.ec.eudi.pid_mdoc"
  ],
  "extra": {}
}
```

- `oid4vp_credentials_requested` は文字列配列
- 同セッションの `/oid4vp` で最優先
- 旧キー `credentials_requested` も互換受け付け（新規利用は非推奨）

### B. YAMLデフォルトで指定

`app/config_issuer_backend_local.yaml`:

```yaml
oid4vp_credentials_requested:
  - "jp.ac.aaa-university.student_id"
  - "eu.europa.ec.eudi.pid_mdoc"
```

## 5) 反映と確認

- JSON/YAML変更後は `issuer_backend_local` を再起動
- `dynamic_presentation_url` の先で Verifier が起動しており、ネットワーク的に Issuer から到達できること
- `/credential_offer2` 実行後、同セッションで `/oid4vp` を実行
- 指定IDが `credential_configurations_supported` に存在することを確認
- 発行対象IDが `credential_auth_methods.PID_login` に含まれていることを確認
