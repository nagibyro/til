# Object Locks
S3 [Object Locks](https://docs.aws.amazon.com/AmazonS3/latest/dev/object-lock-overview.html)
Allow you to add additional compliance security around the deletion of s3 objects.
Which is super useful for meeting compliance requirements. Object locks
allow you to set retention modes & a retention period. The mode basically states
what IAM permissions you require to delete an object. The strongest mode doesn't
even allow the account root user to delete the object.

Retention period gives the date to which you cannot delete the object. This can
either be set as a default for the whole bucket or individual objects.

**Note** seems that you have to set some specific IAM policies to not allow
`PutObject` requests to set their own retention period. If you want to force all
objects to have the same retention


The last bit is `Legal Holds` which essentially are retention periods with out an
end date.


Looks like object locks just stop deletions but don't actually delete the data
after the retention period so you still need a lifecycle policy.