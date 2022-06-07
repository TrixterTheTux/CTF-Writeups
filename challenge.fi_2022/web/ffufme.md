# ffufme 1
```
We know exactly where the flag is. It is in https://ffufme1.challenge.fi/dir/dir/dir/... We also know exactly what the directories are. The provided file common.txt contains all the possibilities for the directory names.

Just try them all. You'll get there eventually..
https://ffufme1.challenge.fi/ 
```
Attachments: `common.txt`  

We're given a dictionary to use for bruteforcing the directory path to the flag. General hints to get this challenge solved is to try with and without a trailing slash, and try to compare all of the results and filter based on that, for example by first setting `-mc all` (match all codes), and then filtering per smaller condition to possibly notice some different behavior for single endpoint.

Let us start by doing simple ffufing:
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/FUZZ                               

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/FUZZ
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

wwwthreads              [Status: 200, Size: 0, Words: 1, Lines: 1]
:: Progress: [4686/4686] :: Job [1/1] :: 608 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

Then, all directories return `HTTP 200`, so we need to filter standard responses which we can do with `-fs 5` (filter size 5):
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/wwwthreads/FUZZ -fs 5

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/wwwthreads/FUZZ
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 5
________________________________________________

polls                   [Status: 200, Size: 10, Words: 1, Lines: 1]
:: Progress: [4686/4686] :: Job [1/1] :: 1315 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

Next, you won't find a result the same way unless you specifically add a trailing `/`:
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/wwwthreads/polls/FUZZ/ -fs 5

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/wwwthreads/polls/FUZZ/
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 5
________________________________________________

schedule                [Status: 200, Size: 20, Words: 5, Lines: 1]
:: Progress: [4686/4686] :: Job [1/1] :: 1108 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

Now, all directories return different sized responses, and if you visit e.g. `https://ffufme1.challenge.fi/wwwthreads/polls/schedule/test`, you'll notice the message changes to `test does not seem right`, where test is always the directory. Thus we'll filter that there isn't 5 words and remove the trailing slash or you won't find a result:
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/wwwthreads/polls/schedule/FUZZ -fw 5 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/wwwthreads/polls/schedule/FUZZ
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response words: 5
________________________________________________

_private                [Status: 200, Size: 22, Words: 4, Lines: 1]
:: Progress: [4686/4686] :: Job [1/1] :: 1268 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```

Matching all codes and filtering non-0 size:
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/FUZZ -mc all -fs 0

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/FUZZ
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
 :: Filter           : Response size: 0
________________________________________________

tartarus                [Status: 400, Size: 95, Words: 8, Lines: 5]
:: Progress: [4686/4686] :: Job [1/1] :: 357 req/sec :: Duration: [0:00:05] :: Errors: 0 ::
```

Adding a trailing slash will cause a single request to error, which we can find with `-debug-log [file]`:
```bash
└─$ ffuf -w ~/Downloads/common.txt -u https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/tartarus/FUZZ/ -debug-log ./ffuf.txt

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/tartarus/FUZZ/
 :: Wordlist         : FUZZ: /home/trixter/Downloads/common.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
________________________________________________

:: Progress: [4686/4686] :: Job [1/1] :: 793 req/sec :: Duration: [0:00:04] :: Errors: 1 ::

└─$ cat ffuf.txt
2022/06/07 15:33:23 Error while opening default config file: open /home/trixter/.ffufrc: no such file or directory
2022/06/07 15:33:24 Get "https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/tartarus/audit/": 303 response missing Location header
```

Checking the response headers of `https://ffufme1.challenge.fi/wwwthreads/polls/schedule/_private/tartarus/audit/` will get us the flag: `GENZ{just_ffuf_all_the_things!}`.

# ffufme 2
```
This one is simple. (But maybe not easier)

Just go to the url https://ffufme2.challenge.fi/FLAG, where the FLAG is the flag for this challenge in the format GENZ{text_her3}

Oh wait. You don't know the flag. Maybe guess it?
https://ffufme2.challenge.fi/
```

So the goal is to guess the flag somehow. Bruteforcing the whole flag at once isn't really viable due to the amount of requests, so maybe it would be possible to do it character by character. First we'll generate all printable ASCII characters as our dictionary:
```py
with open('charset.txt', 'wb') as fout:
    for i in range(33, 127):
        fout.write(bytes([i]) + b'\n')
```

And compare if we can somehow guess one character at a time:
```bash
└─$ ffuf -w charset.txt -u https://ffufme2.challenge.fi/GENZFUZZ -mc all

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme2.challenge.fi/GENZFUZZ
 :: Wordlist         : FUZZ: charset.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
________________________________________________

[...]
w                       [Status: 404, Size: 0, Words: 1, Lines: 1]
t                       [Status: 404, Size: 0, Words: 1, Lines: 1]
v                       [Status: 404, Size: 0, Words: 1, Lines: 1]
}                       [Status: 404, Size: 0, Words: 1, Lines: 1]
y                       [Status: 404, Size: 0, Words: 1, Lines: 1]
|                       [Status: 404, Size: 0, Words: 1, Lines: 1]
z                       [Status: 404, Size: 0, Words: 1, Lines: 1]
~                       [Status: 404, Size: 0, Words: 1, Lines: 1]
{                       [Status: 404, Size: 0, Words: 1, Lines: 1]
:: Progress: [94/94] :: Job [1/1] :: 18 req/sec :: Duration: [0:00:06] :: Errors: 1 ::
```
Note how the next character `{` responded last (though as these requests are sent in bulk, this isn't a reliable metric), so we'll also need to raise the threads to the size of the character set with `-t 94`. The more characters we know, the longer it takes so we'll also have to raise `-timeout` to a higher value than 10 seconds. This method isn't also really reliable so you may need to run same character multiple times to confirm that it actually is the next character.

Repeating this long enough, we can get character by character, until we get the flag: 
```bash
└─$ ffuf -w charset.txt -u https://ffufme2.challenge.fi/GENZ{keep_g0ing_TRY_Heftier_ok_take_flagFUZZ -mc all -t 94 -timeout 60

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : https://ffufme2.challenge.fi/GENZ{keep_g0ing_TRY_Heftier_ok_take_flagFUZZ
 :: Wordlist         : FUZZ: charset.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 60
 :: Threads          : 94
 :: Matcher          : Response status: all
________________________________________________

[...]
G                       [Status: 404, Size: 0, Words: 1, Lines: 1]
<                       [Status: 404, Size: 0, Words: 1, Lines: 1]
s                       [Status: 404, Size: 0, Words: 1, Lines: 1]
6                       [Status: 404, Size: 0, Words: 1, Lines: 1]
}                       [Status: 200, Size: 41, Words: 1, Lines: 1]
t                       [Status: 404, Size: 0, Words: 1, Lines: 1]
@                       [Status: 404, Size: 0, Words: 1, Lines: 1]
:: Progress: [94/94] :: Job [1/1] :: 4 req/sec :: Duration: [0:00:21] :: Errors: 1 ::
```

Flag: `GENZ{keep_g0ing_TRY_Heftier_ok_take_flag}`.