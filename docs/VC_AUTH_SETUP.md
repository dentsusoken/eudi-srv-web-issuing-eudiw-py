# Changing VP Types Used for VP Authentication

Short guide for changing which credentials are **requested for authentication** when issuing credentials via VC authentication.

**[日本語 (Japanese)](VP_AUTH_SETUP.ja.md)**

## Terminology

- `credential_configurations_supported` (`credentialsSupported`)  
  -> **Catalog of credentials this Issuer can handle**
- `credential_auth_methods.PID_login`  
  -> **Credentials that may be issued when using VC authentication** (the name `PID_login` reflects EUDI’s assumption of PID-based authentication; it is kept for compatibility with EUDI.)

## Recommended order of steps

1. Define the credential IDs you want to request in `credentialsSupported` (skip if already defined)
2. Allow only the credential IDs you want to issue using **VC authentication** under `PID_login` (skip if already allowed)
3. Point the Verifier (`dynamic_presentation_url`) at your environment (see section 3)
4. Set `oid4vp_credentials_requested` via session or YAML
5. Restart `issuer_backend_local`

## 1) Define requestable credentials (`credentialsSupported`)

> [!NOTE]
> Run this step only when **adding a new type of credential**. If you only use credentials that are already defined, skip this step.

Add a credential definition JSON under the following path:

Where to add:

- `app/metadata_config/credentials_supported/*.json`

Minimal example:

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

## 2) Allow credentials issuable via VC authentication (`credential_auth_methods.PID_login`)

Under `credential_auth_methods.PID_login` in the following file, add the credentials that may be issued when using VC authentication.

`app/config_issuer_backend_local.yaml`:

```yaml
credential_auth_methods:
  PID_login:
    - eu.europa.ec.eudi.pid_mdoc
    - jp.ac.aaa-university.student_id
```

## 3) Verifier endpoint (`dynamic_presentation_url`)

> [!NOTE]
> By default, `dynamic_presentation_url` points at the Verifier Backend URL when running the Verifier locally in Docker.  
> Change it only when needed.

`/oid4vp` starts a presentation against the verifier’s `/ui/presentations`; set the base URL in `dynamic_presentation_url`.

Where to configure:

- `dynamic_presentation_url` in `app/config_issuer_backend_local.yaml`

Example (local development: verifier listens on host port **8080**):

```yaml
dynamic_presentation_url: "http://host.docker.internal:8080/ui/presentations"
```

- When the Issuer runs in Docker, `localhost` often refers to the container itself and connections fail; **`host.docker.internal`** is commonly used to reach the host’s published port.
- If you run the Issuer on the host directly, or the Issuer and Verifier share a Docker network with resolvable service names, use values such as `http://localhost:8080/...` or `http://verifier-backend-local:8080/...` as appropriate.
- For a cloud verifier, replace with its public URL (e.g. `https://verifier-backend.eudiw.dev/ui/presentations`).

Restart `issuer_backend_local` after changes.  
(`.env.local`’s `DYNAMIC_PRESENTATION_URL` is **not** read by the app; the YAML value is what takes effect.)

## 4) Specify VP types requested on `/oid4vp`

Specify which credential types are used for VC authentication.  
There are two ways: set them in the Credential Offer API request, or in YAML.

### A. Specify in the Credential Offer API request

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

- `oid4vp_credentials_requested` is an array of strings
- Highest precedence for `/oid4vp` in the same session
- Legacy key `credentials_requested` is still accepted (not recommended for new integrations)

### B. YAML defaults

`app/config_issuer_backend_local.yaml`:

```yaml
oid4vp_credentials_requested:
  - "jp.ac.aaa-university.student_id"
  - "eu.europa.ec.eudi.pid_mdoc"
```

## 5) Apply and verify

- Restart `issuer_backend_local` after JSON/YAML changes
- Ensure Verifier is running at `dynamic_presentation_url` and is reachable from the Issuer over the network
- After `/credential_offer2`, call `/oid4vp` in the **same session**
- Confirm requested IDs exist in `credential_configurations_supported`
- Confirm issuance targets are listed under `credential_auth_methods.PID_login`
