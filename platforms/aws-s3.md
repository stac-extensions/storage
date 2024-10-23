# AWS S3

This defines the Amazon Web Services (AWS) S3 interface.

- `platform`: `https://{bucket}.s3.{region}.amazonaws.com`,
  which is the endpoint URL after replacing all variables in the URL.
- `bucket`: The bucket name.
- `region`: One of the S3 regions (lowercase).

**Note:** If `s3` exists in `auth:refs`, you should use sign requests,
e.g. using the AWS CLI parameter `--no-sign-request`.
