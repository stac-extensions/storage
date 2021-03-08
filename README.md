# Storage Extension Specification

- **Title:** Storage
- **Identifier:** <https://stac-extensions.github.io/storage/v1.0.0/schema.json>
- **Field Name Prefix:** storage
- **Scope:** Item, Asset
- **Extension [Maturity Classification](https://github.com/radiantearth/stac-spec/tree/master/extensions/README.md#extension-maturity):** Proposal
- **Owner**: @davidraleigh

This document explains the Storage Extension to the [SpatioTemporal Asset Catalog](https://github.com/radiantearth/stac-spec) (STAC) specification.
This extension adds fields to STAC Item and Asset objects, allowing for details related to cloud storage access and costs to be associated
with a STAC Item.  This extension does not cover NFS solutions provided by PaaS cloud companies.

- Examples:
  - [Item example 1](examples/example-naip.json): Shows the basic usage of the extension in a STAC Item.
  - [Item example 2](examples/example-nsl.json): Another example of basic usage.
- [JSON Schema](json-schema/schema.json)
- [Changelog](./CHANGELOG.md)

## Item Fields
| Field Name  | Type   | Description |
| ----------- | ------ | ----------- |
| storage:min_tier_duration   | integer  | number of days for the shortest time tier restriction on access of an asset. |
| storage:max_tier_duration   | integer  | number of days for the longest time tier restrictions on access of an asset. |
| storage:archived            | bool    | descriptor for whether the data is "properly" archived according to implementers discretion |

## Asset Fields

| Field Name  | Type   | Description |
| ----------- | ------ | ----------- |
| storage:platform              | string    | (REQUIRED) The [cloud provider](#providers) where data is stored |
| storage:manager               | string    | The entity in charge of managing the data. |
| storage:region                | string    | (REQUIRED) The region where the data is stored. Relevant to speed of access and inter region egress costs (as defined by PaaS provider) |
| storage:bucket                | string    | The bucket for the asset, used along with object path |
| storage:object_path           | string    | The object_path for the asset, used along with bucket |
| storage:requester_pays        | bool      | (REQUIRED) Is the data requester pays or is it data manager/cloud provider pays |
| storage:tier                  | string    | The title for the tier type (as defined by PaaS provider) |
| storage:tier_duration         | integer   | Minimum storage duration (in days) required before additional fees |
| storage:date_stored           | string    | Date and time the corresponding asset placed into the current storage tier (relevant for tier_duration > 0). Format is RFC 3339. |
| storage:first_byte_latency    | string    | approximate time unit (milliseconds, minutes or hours) for accessing first byte of data |

### Additional Field Information

#### Providers
Currently this document is arranged to support object storage users of the following PaaS solutions:

- Alibaba
- AWS
- Azure
- Google Cloud Platform
- IBM
- Oracle

#### Cloud Provider Storage Tiers

| Duration      | Google Cloud  | AWS                   | Azure         | IBM           | Oracle    | Alibaba           |
| ------------- | ------------- | --------------------- | ------------- |-------------  | --------- | ---------         |
| 0 days        | STANDARD      | Standard              | Hot Tier      | Standard      | Standard  | Standard          |
| 30 days       | NEARLINE      | Standard-IA           | Cool Tier     | Vault         | N/A       | Infrequent Access |
| 60 days       | N/A           | N/A                   | N/A           | N/A           | N/A       | Archive           |
| 90 days       | COLDLINE      | Glacier               | N/A           | Cold Vault    | Archive   | N/A |
| 180 days      | N/A           | Glacier Deep Archive  | Archive Tier  | N/A           | N/A       | Cold Archive |
| 365 days      | ARCHIVE       | N/A                   | N/A           | N/A           | N/A       | N/A |

References for the above table:

IBM: <https://cloud.ibm.com/objectstorage/create#pricing>

Google Cloud: <https://cloud.google.com/storage/docs/storage-classes>

Microsoft: <https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-storage-tiers>

AWS: <https://aws.amazon.com/s3/storage-classes/>

Oracle: 
- <https://www.oracle.com/cloud/storage/pricing.html>
- <https://www.oracle.com/cloud/storage/archive-storage-faq.html>

Alibaba: 
- <https://www.alibabacloud.com/product/oss/pricing>
- <https://www.alibabacloud.com/help/doc-detail/51374.htm>

All timestamps MUST be formatted according to [RFC 3339, section 5.6](https://tools.ietf.org/html/rfc3339#section-5.6).
