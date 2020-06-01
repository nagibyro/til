# Defense in Depth
AWS blog has an excellent article about securing s3 buckets [in depth](https://aws.amazon.com/blogs/security/how-to-use-bucket-policies-and-apply-defense-in-depth-to-help-secure-your-amazon-s3-data/).
Essentially it says setting bucket ACL to private is a good start but doesn't protect you against
users accidentally or intentionally uploading objects publicly. The article essentially summarizes an IAM policy that will deny `putObject` requests that would set object acl's to public.

It also talks about setting policies for only accessing over HTTPS however this has implications
if you are using your bucket for website hosting and not using cloudfront. An example policy that
implements this defense in depth strategy in terraform would look like:

```hcl
data "aws_iam_policy_document" "default_policy" {
  statement {
    effect = "Deny"

    actions = [
      "s3:PutObject",
      "s3:PutObjectAcl",
    ]

    principals {
      type = "AWS"

      identifiers = [
        "*",
      ]
    }

    resources = [
      "${aws_s3_bucket.bucket.arn}/*",
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-acl"

      values = [
        "public-read",
        "public-read-write",
        "authenticated-read",
      ]
    }
  }

  statement {
    effect = "Deny"

    actions = [
      "s3:PutObject",
      "s3:PutObjectAcl",
    ]

    principals {
      type = "AWS"

      identifiers = [
        "*",
      ]
    }

    resources = [
      "${aws_s3_bucket.bucket.arn}/*",
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-grant-read"

      values = [
        "*http://acs.amazonaws.com/groups/global/AllUsers*",
        "*http://acs.amazonaws.com/groups/global/AuthenticatedUsers*",
      ]
    }
  }

  statement {
    effect = "Deny"

    actions = [
      "s3:PutBucketAcl",
    ]

    principals {
      type = "AWS"

      identifiers = [
        "*",
      ]
    }

    resources = [
      aws_s3_bucket.bucket.arn,
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-acl"

      values = [
        "public-read",
        "public-read-write",
        "authenticated-read",
      ]
    }
  }

  statement {
    effect = "Deny"

    actions = [
      "s3:PutBucketAcl",
    ]

    principals {
      type = "AWS"

      identifiers = [
        "*",
      ]
    }

    resources = [
      aws_s3_bucket.bucket.arn,
    ]

    condition {
      test     = "StringEquals"
      variable = "s3:x-amz-grant-read"

      values = [
        "*http://acs.amazonaws.com/groups/global/AllUsers*",
        "*http://acs.amazonaws.com/groups/global/AuthenticatedUsers*",
      ]
    }
  }

  statement {
    effect = "Deny"

    actions = [
      "s3:PutEncryptionConfiguration",
    ]

    principals {
      type = "AWS"

      identifiers = [
        "*",
      ]
    }

    resources = [
      aws_s3_bucket.bucket.arn,
    ]
  }
}
```

**Note** this policy doesn't implement everything from the blog post but is a good start.