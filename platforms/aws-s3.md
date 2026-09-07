# AWS S3

This defines the Amazon Web Services (AWS) S3 interface.

- `platform`: `https://{bucket}.s3.{region}.amazonaws.com`,
  which is the endpoint URL after replacing all variables in the URL.
- `bucket`: The bucket name.
- `region`: One of the S3 regions (lowercase);
- `storage_class`: The AWS storage-class code associated with the objects, for
  example `STANDARD`, `STANDARD_IA`, `ONEZONE_IA`, `INTELLIGENT_TIERING`,
  `GLACIER_IR`, `GLACIER`, or `DEEP_ARCHIVE`.

The allowed class and transition combinations depend on bucket type, object
size, source class, target class, and AWS lifecycle constraints. Consumers must
not infer that every syntactically valid `storage_class` is available for every
bucket.

**Note:** If the `s3` authentication scheme (i.e. "Simple S3 authentication") is referred to through `auth:refs`, you should disable signing requests,
e.g. using the AWS CLI parameter `--no-sign-request`.

## Lifecycle crosswalk

| Generic construct | AWS S3 mapping | Conditions |
| --- | --- | --- |
| Source and target tier | Source and target `storage_class` | The target scheme must identify an AWS class supported by the bucket. |
| `datetime` + `transition` | Lifecycle `Transition.Date` | The resolved value must be usable as one rule date; AWS evaluates dates at midnight UTC. Directory buckets do not support date-based lifecycle transitions. |
| `datetime` + `expire` | Lifecycle `Expiration.Date` | Same scoping and midnight-UTC constraints; directory buckets do not support date-based expiration. |
| `age` + `transition` | Lifecycle `Transition.Days` | `from` must identify a timestamp guaranteed to equal S3 object creation, and `after` must be an exact positive number of days. |
| `age` + `expire` | Lifecycle `Expiration.Days` | Same object-creation and integral-day requirements. |
| Transition to `GLACIER` or `DEEP_ARCHIVE` | S3 Lifecycle transition | Subject to AWS's permitted-transition and minimum-duration rules. |
| Transition from `GLACIER` or `DEEP_ARCHIVE` to an online class | Application-managed restoration/copy | `RestoreObject` creates temporary access; a permanent class change requires an application-managed copy. It is not a reverse Lifecycle transition. |

AWS Lifecycle supports transition and expiration actions for objects selected by
bucket-rule filters.

If a rule is marked `managed_by: provider` but fails any condition above, a
compiler must reject it. Changing it silently to application-managed would make
the declared execution model inaccurate.

### References

- [Managing the lifecycle of objects](https://docs.aws.amazon.com/AmazonS3/latest/userguide/object-lifecycle-mgmt.html);
- [Lifecycle configuration elements](https://docs.aws.amazon.com/AmazonS3/latest/userguide/intro-lifecycle-rules.html);
- [Amazon S3 storage classes](https://docs.aws.amazon.com/AmazonS3/latest/userguide/storage-class-intro.html).
