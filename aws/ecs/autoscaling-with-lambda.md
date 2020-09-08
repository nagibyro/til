# Autoscaling With Lambda
It appears to not be an uncommon practice to handle autoscaling via lambda instead
of AWS builtin Autoscaling service. This seems mainly for speed and flexibility reasons. When asking some of my coworkers why lambda. I got responses like:

> Main concern for us was speed to scaling event. Even with standard policies set to the minimum threshold (1 datapoint out of 1 minute) standard policies still take 4-5 minutes to actually trigger. using a lambda  cuts that down to 1:30-2 minutes which was a huge win for the minimal cost of lambda

> Same reasoning on my end, cloudwatch events are too slow, and in my case don't handle scaling multiple containers at once, think if 5 out of 20 spot containers were killed, we'd want to immediately scale up 5 containers, cloudwatch events would do 1 at a time for each event, much too slow for us

Basic scaling w/lambda setup would look like:
- scale-out lambda scheduled runs every minute
  - scale-out polls cloudwatch metrics and determines scaling (if needed)
  - updates ECS fargate count
  - turn on scale-in timer (`currentRunning > minRunning`)
- scale-in lamda scheduled every minute (disabled initially)
  - scale-in polls cloudwatch metrics and determines scaling in (if needed)
  - update ECS fargate count
  - if `currentRunning === minRunning` turn off scale-in lambda

Seems good if you need fast scaling or more complex scaling logic. The lambda could query any
data point to determine scaling needs if needed which could be interesting.