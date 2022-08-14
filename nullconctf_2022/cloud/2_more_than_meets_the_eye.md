# More than meets the eye
```
There's more to this than you think

Special flag format ENO-....

** The Cloud security challenges are provided by SEC Consult **
http://3.64.214.139/ 
```

If we investigate the calculator page further, we'll notice that the sponsor image is fetched from `https://nullcon-static-content-public-bucket.s3.eu-central-1.amazonaws.com/sec-consult.svg`. Visiting the bucket reveals that there is file listing enabled:
![](./images/public_bucket.png)

The `dummy_credentials` file has contents of `admin:5730df0sd8f4gsg`, though as the SSH of the machine only accepts public key we have nowhere to use these credentials. Though the `.boto` file happens to have another AWS credentials:
```
[Credentials]
aws_access_key_id = AKIA22D7J5LELFTREN7Z
aws_secret_access_key = 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
```

Checking what they are reveals us the flag:
```
└─$ AWS_ACCESS_KEY_ID=AKIA22D7J5LELFTREN7Z AWS_SECRET_ACCESS_KEY=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 aws sts get-caller-identity
{
    "UserId": "AIDA22D7J5LEH4OZQJF3U",
    "Account": "743296330440",
    "Arn": "arn:aws:iam::743296330440:user/ENO-W0W_sO_M4ny_Pu8l1c_F1leS"
}
```

Flag: `ENO-W0W_sO_M4ny_Pu8l1c_F1leS`