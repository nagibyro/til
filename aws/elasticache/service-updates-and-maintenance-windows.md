# Service Updates and Maintenance Windows
When creating ElastiCache clusters you provide a time to allow for maintenance windows to apply.
Maintenance windows allow for underlying AWS processes to apply to your ElastiCache cluster.
This seems to include (but not limited to):
- Service Instance Type resizing
- Migrating underlying AWS hosts
- Parameter group applies
- ect...

However it does not automatically apply security patches these are run through a
separate update mechanism called Service Updates. Service updates are not applied
automatically but must be applied yourself which if you don't have an automatic
system (i.e. something that kicks it off via the api) then it'll will be a manuel process.
