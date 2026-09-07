# Storage Lifecycle Provider Crosswalk

This document defines how the provider-neutral `lifecycle` member of a Storage
Scheme Object maps to the providers documented by the Storage Extension.

The crosswalk is normative for implementations that translate STAC lifecycle
rules into provider operations. It does not assert that a provider has already
been configured with the corresponding native policy.

## Translation rules

An implementation translates a Lifecycle Rule Object as follows:

1. Treat the containing Storage Scheme Object as the source scheme.
2. For a `transition` action, resolve `target` against the keys in the same
   `storage:schemes` map.
3. Inspect the source and target provider-specific class or tier fields.
4. Compile the rule to a provider-managed policy only when the mapping is exact.
5. Reject a provider-managed rule that cannot be represented exactly. An
   implementation must not silently approximate its trigger or action.
6. Execute all other supported rules through an application-managed workflow.
7. Update an Asset's or Link's `storage:refs`, and its `href` when necessary,
   only after the destination is usable.

JSON Schema validates the shape of the rule. Referential integrity, provider
eligibility, duration conversion, and equivalence between STAC timestamps and
provider timestamps require semantic validation.

## Summary

| Generic lifecycle construct | `aws-s3` | `custom-s3` | `ms-azure` |
| --- | --- | --- | --- |
| Current class or tier | `storage_class` | `storage_class` | `storage_class` |
| `managed_by: application` | AWS object APIs | Provider-specific S3-compatible APIs | Azure Blob APIs |
| `managed_by: provider` | S3 Lifecycle configuration when exactly representable | Only when the endpoint explicitly documents compatible lifecycle operations | Azure lifecycle management policy when exactly representable |
| `datetime` trigger | S3 `Date` in the restricted cases below; otherwise application | Provider-specific; application by default | Application only |
| `age` trigger | S3 `Days` only when based on S3 object creation and expressed as integral days | AWS mapping only when explicitly supported; otherwise application | `daysAfterModificationGreaterThan` for base blobs when timestamps are equivalent and the duration is integral days |
| `transition` action | S3 `Transition`; archive restoration requires application logic | Provider-specific; application by default | `tierToCool`, `tierToCold`, or `tierToArchive`; rehydration requires application logic |
| `expire` action | S3 `Expiration` | Provider-specific; application by default | Azure lifecycle `delete` for age rules; application for absolute datetimes |

## Common limitations

### Per-object timestamps

The generic `datetime.at` and `age.from` fields are JSON Pointers into a STAC
object. Cloud lifecycle configurations are generally bucket- or account-level
policies. A provider policy can be generated only if the resolved STAC value can
be represented by one provider rule and the provider rule is scoped to exactly
the intended objects, for example through a prefix or object tag managed outside
this extension.

### Durations

The generic model uses ISO 8601 durations. AWS and Azure lifecycle policies use
whole days for the mappings documented here. A provider compiler may translate
only durations that are an exact, positive integral number of days. Durations
containing years, months, weeks, fractional values, or time components must be
application-managed unless the provider guarantees an equivalent interpretation.

### Copying between locations

A lifecycle transition usually changes a storage class or access tier inside a
provider boundary. A transition that changes endpoint, account, bucket,
container, or provider is an application-managed copy or move. The application
must verify the destination before changing `storage:refs` or deleting a source
object.

### Expiration

`expire` applies to stored bytes. It does not imply deletion of the STAC Item,
Collection, or Catalog. Soft-delete, version-retention, legal-hold, and object-lock
behavior remain provider concerns.

## Provider details

- [`aws-s3`](./aws-s3.md)
- [`custom-s3`](./custom-s3.md)
- [`ms-azure`](./ms-azure.md)
