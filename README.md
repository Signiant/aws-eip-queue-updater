# AWS SQS Keepalive Service 

We use SQS in one application as a simple lock for assigning Elastic IPs to instances (since the EIP can be 'stolen').  To do this, we have a queue with a single message in there - if the instance can read the message, it has obtained the lock.

The problem is that SQS has a maxium message duration of 14 days, after which messages will be purged.  To workaround this, we run a cron task on our EC2 instances under Elastic Beanstalk that read and replace the message once a day.  This works fine...until we want to 'pause' the eb environment.

This task then checks for SQS queues matching a pattern and will read the single message and re-post it.

## Variables

- VERBOSE - enable more logging.  Default: False
- FREQUENCY - How often to check the SQS queues (in seconds).  Default: 86400
- SQS_NAME_PATTERN - Process queues with this pattern in their name.  Default: EIPQueue
- EXCLUDE_REGIONS - Comma seperated list of SQS regions to ignore.  Default: cn-north-1,us-gov-west-1

## Example Docker run

````
docker run -d -e "FREQUENCY=90" \
	      -e "SQS_NAME_PATTERN=foo \
              --name sqs-keepalive \
              signiant/aws-sqs-keepalive
````

## Running under the EC2 Container Service (ECS)

A sample task definition is included under the ecs folder.
