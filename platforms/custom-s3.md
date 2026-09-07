# Generic S3 (non-AWS)

This defines the S3 interface for providers other than AWS (e.g. minio-based).

- `platform`: The API URL (template), must be the endpoint URL that can be used for the AWS CLI for example, e.g. `https://{bucket}.example.com` or `http://example.com:9000`.
- `bucket`: The bucket name, if applicable.
- `region`: The region, if applicable;
- `storage_class`: An opaque provider-defined storage-class identifier, if the
  provider exposes storage classes.

## Lifecycle crosswalk

`custom-s3` guarantees an S3-compatible data-access interface; it does not imply
support for the complete AWS S3 management API or identical lifecycle semantics.

| Generic construct | Mapping |
| --- | --- |
| `managed_by: application` | Use the endpoint's documented object, copy, restore, tier, and delete operations. |
| `managed_by: provider` | Allowed only when the endpoint explicitly documents compatible lifecycle-policy operations. |
| `datetime` trigger | Provider-specific; application-managed by default. |
| `age` trigger | Use the [`aws-s3` mapping](aws-s3.md) only when the provider explicitly guarantees the same clock, unit, filtering, and execution semantics. |
| `transition` action | Use `storage_class` as the provider-defined target class when the provider supports it; otherwise use an application-managed copy or move. |
| `expire` action | Provider lifecycle deletion only when explicitly supported; otherwise application-managed deletion. |

An implementation must use capability discovery or provider documentation; it
must not infer lifecycle support from `type: custom-s3` alone. Unknown
`storage_class` values are intentionally accepted because they are owned by the
custom provider.

## Mapping to S3 tooling

### GDAL (`/vsis3/`)

GDAL documentation: <https://gdal.org/en/latest/user/virtual_file_systems.html#vsis3-aws-s3-files>

- `platform`: Some options for S3 can be inferred from the given URL (template):
  - `AWS_HTTPS` can be retrieved by parsing the scheme part of the URL. `https` = `ON`, `http` = `OFF`.
  - `AWS_S3_ENDPOINT` is the authority part of the URL after replacing all variables in the URL,
     e.g. `us-west.mycloud.com` without `https://` or `s3://` as prefix.
  - `AWS_VIRTUAL_HOSTING` must be set to `FALSE` if there's no `{bucket}` placeholder in the URL template, otherwise `TRUE` (default value).
- The `region` property corresponds to the `AWS_REGION` option.
- The `requester_pays` property corresponds to the `AWS_REQUEST_PAYER` option. If `requester_pays` is `true`, set `AWS_REQUEST_PAYER` to `requester`.
- If the `s3` authentication scheme (i.e. "Simple S3 authentication") is referred to through `auth:refs`,
   you should set `AWS_NO_SIGN_REQUEST` to `NO`. Otherwise it should be `YES`.

### AWS CLI

AWS CLI documentation: <https://awscli.amazonaws.com/v2/documentation/api/latest/reference/index.html>

- `platform` corresponds to `--endpoint-url` after replacing all variables in the URL.
- `region` corresponds to `--region`.
- If `s3` is **missing** from `auth:refs`, you should use `--no-sign-request`.

### s3cmd

s3cmd documentation: <https://s3tools.org/usage>

- `platform` corresponds to `--host` after replacing all variables in the URL.
- `region` corresponds to `--region`.
- `requester_pays` corresponds to `--requester-pays`.
- If the `s3` authentication scheme (i.e. "Simple S3 authentication") is referred to through `auth:refs`,
   you should provide an secret access key and an access key id through environment variables, a profile or the `s3cmd sign` command.
