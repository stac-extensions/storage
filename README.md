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
  - [NAIP Item](examples/item-naip.json): Shows the usage of the extension in combination with the alternate asset extension.
  - [NSL Item](examples/item-nsl.json): Shows a mixture of storage providers, including custom S3 hosts.
  - [Catalog with Link](examples/catalog-link.json): Shows the usage of the extension on a link in a STAC Catalog.
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
| platform       | string  | **REQUIRED.** The [cloud provider](#platforms) where data is stored. |
| region         | string  | The region where the data is stored. Relevant to speed of access and inter region egress costs (as defined by PaaS provider) |
| requester_pays | boolean | Is the data requester pays or is it data manager/cloud provider pays. Defaults to `false` |
| tier           | string  | The title for the tier type (as defined by PaaS provider) |

The properties `title` and `description` as defined in Common Metadata can be used as well.

#### Platforms

The `platform` field identifies the cloud provider where the data is stored.

There are a couple of pre-defined values for common providers:

- Alibaba Cloud (Aliyun): `ALIBABA`
- Amazon AWS: `AWS`
- Microsoft Azure: `AZURE`
- Google Cloud Platform: `GCP`
- IBM Cloud: `IBM`
- Oracle Cloud: `ORACLE`

All other PaaS solutions must use a unique URL to the service.

In case an `href` contains a non-HTTP URL that is not directly resolvable,
the `platform` property must identify the host so that the URL can be resolved without further information.
This is especially useful to provide the endpoint URL for custom S3 providers.
In this case the `platform` is effectively the endpoint URL.

#### Tiers

Recommended values for the `tier` field:

| Minimum Duration | [Google Cloud Platform](https://cloud.google.com/storage/docs/storage-classes) | [Amazon AWS](https://aws.amazon.com/s3/storage-classes/) | [Microsoft Azure](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-storage-tiers) | [IBM Cloud](https://cloud.ibm.com/objectstorage/create#pricing)  | [Oracle Cloud](https://www.oracle.com/cloud/storage/pricing.html) | [Alibaba Cloud](https://www.alibabacloud.com/product/oss/pricing) |
| ------------- | --------- | ------------------------ | ------- |----------  | ----------------- | ----------------- |
| 0 (Auto-Tier) |           | Intelligent-Tiering      |         | Smart Tier |
| 0 days        | STANDARD  | Standard                 | hot     | Standard   | Standard          | Standard          |
| 30 days       | NEARLINE  | Standard-IA, One Zone-IA | cool    | Vault      | Infrequent Access | Infrequent Access |
| 60 days       |           |                          |         |            |                   | Archive           |
| 90 days       | COLDLINE  | Glacier                  |         | Cold Vault | Archive           | |
| 180 days      |           | Glacier Deep Archive     | archive |            |                   | Cold Archive |
| 365 days      | ARCHIVE   |                          |         |            |                   | |

## Contributing

All contributions are subject to the
[STAC Specification Code of Conduct](https://github.com/radiantearth/stac-spec/blob/master/CODE_OF_CONDUCT.md).
For contributions, please follow the
[STAC specification contributing guide](https://github.com/radiantearth/stac-spec/blob/master/CONTRIBUTING.md) Instructions
for running tests are copied here for convenience.

### Running tests

The same checks that run as checks on PR's are part of the repository and can be run locally to verify that changes are valid. 
To run tests locally, you'll need `npm`, which is a standard part of any [node.js installation](https://nodejs.org/en/download/).

First you'll need to install everything with npm once. Just navigate to the root of this repository and on 
your command line run:
```bash
npm install
```

Then to check markdown formatting and test the examples against the JSON schema, you can run:
```bash
npm test
```

This will spit out the same texts that you see online, and you can then go and fix your markdown or examples.

If the tests reveal formatting problems with the examples, you can fix them with:
```bash
npm run format-examples
```
