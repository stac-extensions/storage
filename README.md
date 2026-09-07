# Storage Extension Specification

- **Title:** Storage
- **Identifier:** <https://stac-extensions.github.io/storage/v2.0.0/schema.json>
- **Field Name Prefix:** storage
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Pilot
- **Owner**: @matthewhanson @m-mohr

This document explains the Storage Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It allows adding details related to cloud object storage access and costs to be associated with STAC Assets.
This extension does not cover NFS solutions provided by PaaS cloud companies.

- Examples:
  - [NAIP Item with Alternate Assets](examples/item-naip.json): Shows a mixture of storage providers, including custom S3 hosts
    and the [alternate assets extension](https://github.com/stac-extensions/alternate-assets).
  - [Catalog with Link](examples/catalog-link.json): Shows the usage of the extension on a link in a STAC Catalog.
  - [Collection with Auth](examples/collection.json): Shows the usage of the extension in a STAC Collecion in combination with the
    [authentication extension](https://github.com/stac-extensions/authentication).
  - Lifecycle sequence: Four successive snapshots of the same Item as its result is
    [produced in hot storage](examples/lifecycle/01-produced-hot.json),
    [archived](examples/lifecycle/02-archived-cold.json),
    [restored](examples/lifecycle/03-restored-hot.json), and finally
    [expired](examples/lifecycle/04-expired-tombstone.json).
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Fields

The fields in the table below can be used in these parts of STAC documents:

- [x] Catalogs
- [x] Collections
- [x] Item Properties (incl. Summaries in Collections)
- [ ] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [ ] Links

| Field Name        | Type                                                         | Description |
| ----------------- | ------------------------------------------------------------ | ----------- |
| `storage:schemes` | Map<string, [Storage Scheme Object](#storage-scheme-object)> | **REQUIRED.** A property that contains all of the storage schemes used by Assets and Links in the STAC Item, Catalog or Collection. |

---

The fields in the table below can be used in these parts of STAC documents:

- [ ] Catalogs
- [ ] Collections
- [ ] Item Properties (incl. Summaries in Collections)
- [x] Assets (for both Collections and Items, incl. Item Asset Definitions in Collections)
- [x] Links
- [x] [Alternate Assets Object](https://github.com/stac-extensions/alternate-assets?tab=readme-ov-file#alternate-asset-object)

| Field Name     | Type       | Description |
| -------------- | ---------- | ----------- |
| `storage:refs` | \[string\] | A property that specifies which schemes in `storage:schemes` may be used to access an Asset or Link. Each value must be one of the keys defined in `storage:schemes`. |

### Storage Scheme Object

| Field Name     | Type    | Description |
| -------------- | ------- | ----------- |
| type           | string  | **REQUIRED.** Type identifier for the platform, see below. |
| platform       | string  | **REQUIRED.** The cloud provider where data is stored as URI or URI template to the API. |
| region         | string  | The region where the data is stored. Relevant to speed of access and inter region egress costs (as defined by PaaS provider). |
| requester_pays | boolean | Is the data "requester pays" (`true`) or is it "data manager/cloud provider pays" (`false`). Defaults to `false`. |
| storage_class  | string  | An opaque provider-defined storage-class identifier, if the provider exposes storage classes. |
| lifecycle      | [Storage Lifecycle Object](#storage-lifecycle-object) | Provider-neutral rules that describe transitions from this scheme or expiration of its stored objects. |
| ...            | ...     | Additional properties as defined in the URL template or in the platform specific documents. |

The properties `title` and `description` as defined in Common Metadata should be used as well.

#### platform

The `platform` field identifies the cloud provider where the data is stored as URI or URI template to the API of the service.

If a URI template is provided, all variables must be defined in the Storage Scheme Object as a property with the same name.
For example, the URI template `https://{bucket}.{region}.example.com` must have at least the properties
`bucket` and `region` defined:

```json
{
  "type": "example",
  "platform": "https://{bucket}.{region}.example.com",
  "region": "eu-fr",
  "bucket": "john-doe-stac",
  "requester_pays": true
}
```

In case an `href` contains a non-HTTP URL that is not directly resolvable,
the `platform` property must identify the host so that the URL can be resolved without further information.
For example, this is especially useful to provide the endpoint URL for custom S3 providers.
In this case the `platform` could effectively provide the endpoint URL.

#### type

We try to collect pre-defined templates and best pratices for as many providers as possible
in this repository, but be aware that these are not part of the official extension releases.
This extension just provides the framework, the provider best pratices
may change at any time without a new version of this extension being released.

The following providers have defined best pratices at this point:

| `type`      | Provider and Documentation |
| ----------- | -------------------------- |
| `aws-s3`    | [AWS S3](platforms/aws-s3.md) |
| `custom-s3` | [Generic S3 (non-AWS)](platforms/custom-s3.md) |
| `ms-azure`  | [Microsoft Azure](platforms/ms-azure.md) |

Feel encouraged to submit additional platform specifications via Pull Requests.

The `type` fields can be any value chosen by the implementor,
but the types defined in the table above should be used as defined in the best practices.
This ensures proper schema validation.

### Storage Lifecycle Object

The optional `lifecycle` property describes provider-neutral lifecycle rules for objects in the containing storage scheme.
The containing Storage Scheme Object is the source of every rule. A transition action identifies its destination by the key of another
Storage Scheme Object in the same `storage:schemes` map.

| Field Name | Type | Description |
| ---------- | ---- | ----------- |
| managed_by | string | The entity expected to evaluate and execute the rules: `provider` or `application`. |
| rules | Map<string, [Storage Lifecycle Rule Object](#storage-lifecycle-rule-object)> | **REQUIRED.** One or more lifecycle rules keyed by their identifiers. |

`managed_by` communicates operational responsibility; it does not change the meaning of a rule. For example, use `application` when an
application evaluates per-Asset timestamps that a storage provider cannot evaluate directly.

#### Storage Lifecycle Rule Object

Each rule is identified by its key in the containing `rules` map. Rule objects contain only `trigger` and `action`.

| Field Name | Type | Description |
| ---------- | ---- | ----------- |
| trigger | [Storage Lifecycle Trigger Object](#storage-lifecycle-trigger-object) | **REQUIRED.** The condition that makes the action eligible. |
| action | [Storage Lifecycle Action Object](#storage-lifecycle-action-object) | **REQUIRED.** The operation to perform when the trigger is eligible. |

#### Storage Lifecycle Trigger Object

Exactly one of the following trigger forms is used:

| `type` | Additional fields | Description |
| ------ | ----------------- | ----------- |
| `datetime` | `at` (string) | Makes the action eligible at the RFC 3339 timestamp identified by `at`. |
| `age` | `from` (string), `after` (string) | Makes the action eligible after an ISO 8601 duration has elapsed from the referenced timestamp. |

`at` and `from` are RFC 6901 JSON Pointers resolved against the STAC object containing `storage:schemes`. They must identify an RFC 3339
timestamp. `after` must be a positive ISO 8601 duration, for example `P30D` or `PT12H`.

#### Storage Lifecycle Action Object

Exactly one of the following action forms is used:

| `type` | Additional fields | Description |
| ------ | ----------------- | ----------- |
| `transition` | `target` (string) | Moves the stored object to the scheme named by `target` in the same `storage:schemes` map. |
| `expire` | none | Expires the stored object. This does not require deletion of the containing STAC entity. |

#### Lifecycle examples

This example transitions an Asset from a hot scheme to a cold scheme when the Asset-level `expires` timestamp is reached:

```json
{
  "storage:schemes": {
    "hot": {
      "type": "aws-s3",
      "platform": "https://{bucket}.s3.{region}.amazonaws.com",
      "bucket": "processing-results",
      "region": "eu-central-1",
      "lifecycle": {
        "managed_by": "application",
        "rules": {
          "archive-after-hot-retention": {
            "trigger": {
              "type": "datetime",
              "at": "/assets/result/expires"
            },
            "action": {
              "type": "transition",
              "target": "cold"
            }
          }
        }
      }
    },
    "cold": {
      "type": "aws-s3",
      "platform": "https://{bucket}.s3.{region}.amazonaws.com",
      "bucket": "processing-results",
      "region": "eu-central-1"
    }
  }
}
```

Age-based triggers can be expressed independently of any provider:

```json
{
  "rules": {
    "archive-after-30-days": {
      "trigger": {
        "type": "age",
        "from": "/properties/created",
        "after": "P30D"
      },
      "action": {
        "type": "transition",
        "target": "cold"
      }
    }
  }
}
```

The four linked lifecycle examples are snapshots of one catalog record changing over time, not four Items intended to coexist. After a
transition succeeds, the producer updates `storage:refs`, the Asset-level deadline, and the active lifecycle rule together. In the final
snapshot the stored bytes have expired, `assets` is empty, and the Item is retained as an unpublished metadata tombstone.

#### Lifecycle crosswalk

The [Lifecycle Crosswalk](./platforms/lifecycle-crosswalk.md) document defines how the provider-neutral
`lifecycle` member of a Storage Scheme Object maps to the providers documented by the Storage Extension.

## Contributing

See the [Contributor documentation](CONTRIBUTING.md) for details.
