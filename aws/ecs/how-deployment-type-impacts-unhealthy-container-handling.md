# How Deployment Type Impacts Unhealthy Container Handling

TIL -- For AWS ECS Fargate the deployment type you choose not only effects how deployments are made but also how the
ECS service behaves when a container goes unhealthy. If you use the Rolling Deployment strategy (which I believe is the
default or at least what I've almost always used in the past) When a container goes unhealthy ECS stops the unhealthy
container before starting a replacement. However this doesn't respect keeping a minimum number of containers around. So
if you have a web service that gets an influx of traffic and containers fail from high cpu or high request count that
outage can cascade and bring your containers down one by one and will actually leave you with 0 running containers. For
us it also didn't auto recover because the traffic would keep thrashing each container as it came up 1 by 1.

However if you don't use rolling deployment type so you use code deploy instead then the AWS docs say:

> If a task is marked unhealthy, the service scheduler will first start a replacement task. If the replacement task has a
health status of HEALTHY, the service scheduler stops the unhealthy task. If the replacement task has a health status
of UNHEALTHY, the scheduler will stop either the unhealthy replacement task or the existing unhealthy task to get the
total task count to equal `desiredCount`. 

Which combined with ALB's failing open aka continue to route running traffic to
unhealthy target hosts gives the chance for the system to recover from a traffic spike. I also believe because the
hosts are killed any metrics that might trigger AWS Autoscaling get interrupted and so Autoscaling policies usually
fail to kick in to increase the number of hosts. (If you're using something like CPU Utilization or memory as your
scaling metric).
