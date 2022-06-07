# Project Kyyber 2021: Spells and magic DLC PACK 1
```
Seems like there were some problems with the launch of the Project Kyyber 2021 and people got angry for little things like rampant cheating etc.

Well bad publicity is publicity and we got funding for a DLC pack. Lets roll this out and roll some dough.

There should now be some spells or something you can cast. One of them is super cool so I made them do it really hard to get. Next year maybe well implement micropayments for getting them easily.

Another feature is a scoreboard. You can track your progress and impress your friends!
```
Hint 1: `The achievement for flag 1 is called "Killcaster!"`  
Attachments: `release_v1.1.zip`  

After quickly glancing the source code, we're given a game that has a scoreboard, client and a server part. You can notice that there used to be flyhacks etc. due to the comments mentioning it but they're now fixed, so that's most likely not what we want to abuse (unless it was fixed badly!). Though we note an achievement that is given if your name is `Nixu nixu nixu`.

I decided to have fun and try to get that achievement even if it wasn't directly related to getting the flag. After some investigating, you can notice that the password is being used as a seed for generating the name. A quick python script bruteforcing possible seeds can be written, which allows us to get the achievement:
```py
import random

def id_to_name(id):
    namechooser = random.Random(id)
    name = ""
    name += namechooser.choice(
        [
            "Red",
            "Blue",
            "Orange",
            "Green",
            "Black",
            "White",
            "Yellow",
            "Cyan",
            "Brown",
            "Nixu",
        ]
    )
    if namechooser.randint(0, 10) == 0:
        name += " "
        name += namechooser.choice(["basher", "master", "kyyber", "nixu"])
    name += " "
    name += namechooser.choice(
        [
            "warrior",
            "thief",
            "monk",
            "sage",
            "hunter",
            "assassin",
            "spy",
            "knight",
            "fighter",
            "nixu",
        ]
    )
    return name

x = 0
while True:
    a = random.Random()
    a.seed(x)
    id = a.getrandbits(64)

    if id_to_name(id) == "Nixu nixu nixu":
        print(x)
        break
    
    x += 1
```

Then I happened to stumble across lootrng code in `common.py` of the game, where we notice there's a `1 in X chance` drop, that happens to be calculated in the following way:
```py
lootrng = player.random.randint(
    0, world.LOOTDIFFICULTY
)
```

And as we can control `player.random`, we can bruteforce the seed the same way until we get the seed `619820`:
```py
import random

def id_to_name(id):
    namechooser = random.Random(id)
    name = ""
    name += namechooser.choice(
        [
            "Red",
            "Blue",
            "Orange",
            "Green",
            "Black",
            "White",
            "Yellow",
            "Cyan",
            "Brown",
            "Nixu",
        ]
    )
    if namechooser.randint(0, 10) == 0:
        name += " "
        name += namechooser.choice(["basher", "master", "kyyber", "nixu"])
    name += " "
    name += namechooser.choice(
        [
            "warrior",
            "thief",
            "monk",
            "sage",
            "hunter",
            "assassin",
            "spy",
            "knight",
            "fighter",
            "nixu",
        ]
    )
    return name

x = 0
while True:
    a = random.Random()
    a.seed(x)
    id = a.getrandbits(64)

    consecutive = 0
    while True:
        lootrng = a.randint(
            0, 1000000
        )
        if lootrng == 1000000:
            consecutive += 1
        else:
            break
    
    if consecutive > 0:
        print(x, consecutive)

    x += 1
```

Then we can use that as a password and join the `challenge.fi` server, kill an enemy, pick the drop up and get the flag: `GENZ{seeded_rng_is_not_rng}`.

# Project Kyyber 2021: Spells and magic DLC PACK 2
```
Project Kyyber 2021: Spells and magic DLC PACK has multiple flags!
```
Hint 1: `The achievement for flag 2 is called "Millionaire!"`  

Next, the hint refers to an achievement named `Millionaire`. My guess would be referring to the score being a million, due to there not being anything else that seems abusable. Originally, I wanted to abuse spells even more (as you can spawn them anywhere) and slowly killing enemies (perhaps automatically?) but then realized only way to get scores is by using projectiles, which don't seem to be abusable due to distance checks and no infinite ammo.

And as we haven't touched the scoreboard part of the challenge yet, it'd seem likely we would have to abuse the scoreboard server somehow, which brings us to trying to abuse JWT due to there being so many different ways to get its implementation wrong. Initially, I tried to use the `none` algorithm but then the library errors about the key being passed. Then I noticed `RS256` and `HS256` algorithms both being enabled, which means we can either choose asymmetric or symmetric encryption. That means if we were to know the public key, we could trick the scoreboard server into doing symmetric encryption with it.

We navigate to https://jwt.io, choose HS256, set the payload data to
```
{"id":4474128141048952249,"score":1001337}
```

Then, we need to get the public key. Though this is trivial because connection to the scoreboard server greets you with the server's public key `ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIIdYJ0JG8+UoJgOaJaNYZQu+DDZlVZ2MilA1yZuut+SU`, which generates us the following JWT:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpZCI6IjQ0NzQxMjgxNDEwNDg5NTIyNDkiLCJzY29yZSI6MTAwMTMzN30.DZMErHtsnbkXPjherSM3SK__xncz5sjTUY36h-J1bbk
```
And then we can netcat to the server and paste the JWT to set our highscore. Though due to an oversight, nothing happened when joining the server with our new score. Then I realized my ID was too large to fit into 32 bits and had lost precision, causing me to have actually the wrong player ID.

Wrapping the id in string quotes produces us the flag `GENZ{oh_my_oh_day_old_is_new_again}`.