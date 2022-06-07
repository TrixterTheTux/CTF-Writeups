# iPhone Forensicator pt 1.
```
The user has shared a secret with a friend. Can you find it?
https://www.dropbox.com/s/xn90t87jgynv6lx/backup.tgz?dl=1 
```

We can use [iLEAPP](https://github.com/abrignoni/iLEAPP) for iPhone forensics making it easy to get an overall view of the provided dump:
```bash
└─$ python3 ~/Tools/iLEAPP/ileapp.py -t gz -i backup.tgz -o ./
```

The description refers to sending a message, and we can find that there's WhatsApp installed, where one of the messages is the flag: `GENZ{wh4ts_up_0r_wa5_1t_4pp}`.

# iPhone Forensicator pt 2.
```
As part 1 revealed, the user found a secure way to store secrets. How secure is it?
(Same file as in part 1) 
```

The description is referring to following messages:
```
Where would you store secrets like that?
I'd write a note and lock it
```

There used to be at some point dedicated tab for viewing notes in iLEAPP, but as I'm unable to find it anymore (as latest version seems to error when trying to extract them directly?) you can just directly check strings inside the SQLite database `NoteStore.sqlite-wal` which contains the flag: `GENZ{w3ll_d0ne_my_fr1end}`

# iPhone Forensicator pt 3.
```
One of the user's TikTok videos has a frame with the flag.
(Same file as in part 1) 
```

We can find easily all the videos in the dump:
```bash
└─$ find . | egrep "\.(mov|mp4)$"

[...]
./private/var/mobile/Media/PhotoData/Thumbnails/V2/DCIM/100APPLE/IMG_0004.mp4
./private/var/mobile/Media/PhotoData/Mutations/DCIM/100APPLE/IMG_0001/Adjustments/FullSizeRender.mov
./private/var/mobile/Media/DCIM/100APPLE/IMG_0004.mp4
./private/var/mobile/Media/DCIM/100APPLE/._IMG_0004.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Library/Application Support/gurd_cache/365b20e8f6c343df1eff65214a0e3e74/fe_tiktok_end_of_year/end-of-year-hub/video/EOY_BG_video_low.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Library/Application Support/gurd_cache/365b20e8f6c343df1eff65214a0e3e74/fe_tiktok_end_of_year/end-of-year-hub/video/EOY_BG_video.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Documents/kAWEPublishLocalVideoStorageFolder/publish_video_local_7034498052782050566.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Documents/kAWEPublishLocalVideoStorageFolder/publish_video_local_7034159208035552518.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Documents/kAWEPublishLocalVideoStorageFolder/publish_video_local_7034483537734634757.mp4
./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Documents/welcomeScreenVideo.mp4
[...]
```

We can ignore most of the prebundled videos as it's highly unlikely that's what the challenge wants to find us, and instead focus on the `private/var/mobile/` prefixed videos.

Watching `./private/var/mobile/Containers/Data/Application/A331F95B-8433-47F7-8874-048D4D148295/Documents/kAWEPublishLocalVideoStorageFolder/publish_video_local_7034159208035552518.mp4` will flash the flag quickly, which is `GENZ{Y0u_ju5t_n33d_to_pay_att3nt10n}`.