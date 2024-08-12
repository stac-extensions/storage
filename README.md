# Storage Extension Specification

- **Title:** Storage
- **Identifier:** <https://stac-extensions.github.io/storage/v2.0.0/schema.json>
- **Field Name Prefix:** storage
- **Scope:** Item, Catalog, Collection
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Pilot
- **Owner**: @davidraleigh @matthewhanson

This document explains the Storage Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
It allows adding details related to cloud object storage access and costs to be associated with STAC Assets.
This extension does not cover NFS solutions provided by PaaS cloud companies.

- Examples:
  - [NAIP Item with Alternate Assets](examples/item-naip.json): Shows a mixture of storage providers, including custom S3 hosts
    and the [alternate assets extension](https://github.com/stac-extensions/alternate-assets).
  - [Catalog with Link](examples/catalog-link.json): Shows the usage of the extension on a link in a STAC Catalog.
  - [Collection with Auth](examples/catalog-link.json): Shows the usage of the extension in a STAC Collecion in combination with the
    [authentication extension](https://github.com/stac-extensions/authentication).
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

| Field Name     | Type    | Description |
| -------------- | ------- | ----------- |
| `storage:refs` | string  | A property that specifies which schemes in `storage:schemes` may be used to access an Asset or Link. Each value must be one of the keys defined in `storage:schemes`. |

### Storage Scheme Object

| Field Name     | Type    | Description |
| -------------- | ------- | ----------- |
| platform       | string  | **REQUIRED.** The cloud provider where data is stored as URI or URI template to the API. |
| region         | string  | The region where the data is stored. Relevant to speed of access and inter region egress costs (as defined by PaaS provider). |
| requester_pays | boolean | Is the data "requester pays" (`true`) or is it "data manager/cloud provider pays" (`false`). Defaults to `false`. |
| ...            | ...     | Additional properties as defined in the URL template or in the platform specific documents. |

The properties `title` and `description` as defined in Common Metadata should be used as well.

#### platform

The `platform` field identifies the cloud provider where the data is stored as URI or URI template to the API of the service.

If a URI template is provided, all variables must be defined in the Storage Scheme Object as a property with the same name.
For example, the URI template `https://{bucket}.{region}.example.com` must have at least the properties 
`bucket` and `region` defined:

```json
{
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

We try to collect pre-defined templates and best pratices for as many providers as possible
in this repository, but be aware that these are not part of the official extension releases
and are not validated. This extension just provides the framework, the provider best pratices
may change at any time without a new version of this extension being released.

The following providers have defined best pratices at this point:

- [AWS S3](platforms/aws-s3.md)
- [Generic S3 (non-AWS)](platforms/s3.md)
- [Microsoft Azure](platforms/ms-azure.md)

Feel encouraged to submit additional platform specifications via Pull Requests.

## Contributing

See the [Contributor documentation](CONTRIBUTING.md) for details.
