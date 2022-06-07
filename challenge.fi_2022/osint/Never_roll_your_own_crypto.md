# Never roll your own crypto?
```
A budding programmer thinks they have the start of a new age in encryption. Can you figure out why:
1. Once something is on the internet, it stays there
2. Never roll your own crypto

Start your investigation here: https://mastodon.social/@l33tpr0grammer

Flag format: GENZ{flag_goes_here}
```
Hint 1: `The git commit history holds all changes made in a project, for better or for worse`  

You can find a screenshot of a github repository named `3ncrypt0r` on the provided mastodon profile, revealing us the [source code](https://github.com/l33tpr0-grammer/3ncrypt0r) of a self-made crypto.

The README contains the most guarded secret, which most likely refers to the flag: `dnZ9bUpdA0NuQANoXQAAQ24HXU5cA0EETA==`, and we can find in the git history that the pin is `1337`, giving us the flag: `GENZ{n0t_s0_l33t_4nym0r3}`.