# SDE-Backend-Assignment-Solution

# Dependency
* AWS EC2
* AWS Lambda
* AWS API Gateway

# Basic Details
* Start / Stop the EC2 created instaces from REST APIs call.
* Show the list of EC2 instances with status.

### GET Request 
* Show all the EC2 instance status (Running/Stopped) 
  > [EC2 GET Request](https://nqi37ozzel.execute-api.us-east-2.amazonaws.com/testing/ec2/ "Exiting EC2 Instance List")
* The below code is used in AWS Lambda to get the EC2 GET Request

```python
import json
import boto3

def lambda_handler(event, context):
    
    ec2 = boto3.resource('ec2')
    
    instanceList = []
    
    for instance in ec2.instances.all():
        
        instanceDirectory = {
          "Id": instance.id,
          "Status": instance.state['Name'].capitalize()
        }
        instanceList.append(instanceDirectory)
    
    return {
        'Status': 200,
        'Body': "Getting EC2 Instance Status!",
        'Instances': instanceList
    }
```

### POST Request
* Start single stopped instance
* Start all stopped instances
* Stop single running instance
* Stop all running instances
> [EC2 POST Request](https://nqi37ozzel.execute-api.us-east-2.amazonaws.com/testing/ec2/ "Update Exiting EC2 Instance Status Start / Stop")
* The below code is used in AWS Lambda to Start / Stop the EC2 instaces via POST Request

```python
import json
import boto3

def lambda_handler(event, context):
    
    ec2_GET = boto3.resource('ec2')
    
    ec2 = boto3.client('ec2', region_name = 'us-east-2')

    stop_instance_id_list = []

    start_instance_id_list = []
    
    # Stop single running instance
    if event['action'] == "stop":
        
        stop_instance_id_list.append(event['instance_id'])
        
    # Stop all running instances
    elif event['action'] == "stop_all":
        
        for instance in ec2_GET.instances.all():
            if instance.state['Name'] == "running":
                stop_instance_id_list.append(instance.id)
    
    # Start single stopped instance
    elif event['action'] == "start":
        
        start_instance_id_list.append(event['instance_id'])
        
    # Start all stopped instances
    elif event['action'] == "start_all":
        
        for instance in ec2_GET.instances.all():
            if instance.state['Name'] != "running":
                start_instance_id_list.append(instance.id)
    
    if stop_instance_id_list:    
        ec2.stop_instances(InstanceIds = stop_instance_id_list)
        
    if start_instance_id_list:
        ec2.start_instances(InstanceIds = start_instance_id_list)

    # TODO implement
    return {
        'statusCode': 200,
        'body': "Instances status update!",
        'Stoped Instances': stop_instance_id_list,
        'Start Instances' : start_instance_id_list
    }

```
> The below Row Body is passed for Start / Stop the EC2 instances.
<table>
<tr>
<td> Action </td> <td> Row Body </td>
</tr>
<tr>
<td> Start Single Instance by ID </td>
<td>

```json
{
    "action": "start",
    "instance_id": "i-08856a5934ec4550d"
}
```

</td>
</tr>
<tr>
<td> Start All Stopped Instances </td>
<td>

```json
{
    "action": "start_all"
}
```

</td>
</tr>  
<tr>
<td> Stop Single Instance by ID</td>
<td>
  
```json
{
    "action": "stop",
    "instance_id": "i-08856a5934ec4550d"
}
```
  
</td>
</tr>
  
<tr>
<td> Stop All Running Instances </td>
<td>
  
```json
{
    "action": "stop_all"
}
```
  
</td>
</tr>
</table>
