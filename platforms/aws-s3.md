# AWS S3

This defines the Amazon Web Services (AWS) S3 interface.

- `platform`: `https://{bucket}.s3.{region}.amazonaws.com`,
  which is the endpoint URL after replacing all variables in the URL.
- `bucket`: The bucket name.
- `region`: One of the S3 regions (lowercase).

**Note:** If the `s3` authentication scheme (i.e. "Simple S3 authentication") is referred to through `auth:refs`, you should disable signing requests,
e.g. using the AWS CLI parameter `--no-sign-request`.
