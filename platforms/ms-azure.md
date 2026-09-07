# Microsoft Azure

This defines the Microsoft Azure interface.

- `platform`: `https://{account}.blob.core.windows.net`
- `account`: The Microsoft account identifier
- `storage_class`: The blob access tier: `Hot`, `Cool`, `Cold`, or `Archive`.

The Access tier describes block blobs associated with the scheme. Tiering
availability depends on the storage account type, redundancy configuration,
blob type, encryption configuration, and Azure feature support.

## Lifecycle crosswalk

| Generic construct | Azure Blob Storage mapping | Conditions |
| --- | --- | --- |
| Source and target tier | Source and target `storage_class` | Tier mapping applies to eligible block blobs. |
| `datetime` | Application-managed evaluation | Azure lifecycle policies do not expose a general absolute per-blob datetime action. |
| `age` + transition to `Cool` | `tierToCool.daysAfterModificationGreaterThan` | `from` must identify a value guaranteed to equal the blob's last-modified time, and `after` must be an exact positive integral number of days. |
| `age` + transition to `Cold` | `tierToCold.daysAfterModificationGreaterThan` | Same timestamp and duration conditions. |
| `age` + transition to `Archive` | `tierToArchive.daysAfterModificationGreaterThan` | Same timestamp and duration conditions. |
| `age` + `expire` | `delete.daysAfterModificationGreaterThan` | Same timestamp and duration conditions. Soft-delete policy may retain deleted blobs. |
| Transition from `Archive` to `Hot`, `Cool`, or `Cold` | Application-managed rehydration with `Set Blob Tier` or `Copy Blob` | Azure lifecycle policies cannot rehydrate archived blobs to an online tier. |
| Access-driven `Cool` to `Hot` | Azure `enableAutoTierToHotFromCool` | This provider-specific optimization has no equivalent generic lifecycle trigger. |

Azure lifecycle management also supports provider-specific clocks such as last
access time and last tier-change time. They are not compiled from arbitrary STAC
JSON Pointers unless the implementation can prove timestamp equivalence.

### References

- [Azure lifecycle management](https://learn.microsoft.com/azure/storage/blobs/lifecycle-management-overview);
- [transition policies](https://learn.microsoft.com/azure/storage/blobs/lifecycle-management-policy-access-tiers);
- [deletion policies](https://learn.microsoft.com/azure/storage/blobs/lifecycle-management-policy-delete);
- [access tiers](https://learn.microsoft.com/azure/storage/blobs/access-tiers-overview).
