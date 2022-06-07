# Catching Cyber Terrorists 1
```
Hello recruit,

We need Your help in catching a nefarious Cyber Terrorist. However as this is a perilous task, we cannot give out information to just anyone, so you need to pass the challenge to gain access to the assignment. All the required information can be found from the attached document.

You may approach us when you find out how. Bear in mind that we are working for the government and reply times may vary. Only correct message in correct media will be replied to with further instructions.

The flag will be provided to you in proper channels. Flag is form: GENZ{xxx}
```
Hint 1: `Read the instructions.`  
Hint 2: `No. I mean REALLY read the instructions.`  
Attachments: `Application.pdf`  

We're given a blank PDF with a QR code, which resolves to https://tinyurl.com/yeyefbzx
Viewing the document's properties we find the following base64 encoded strings:
```
Title: aHR0cHM6Ly90aW55dXJsLmNvbS95ZXllZmJ6eA==
Keywords: KzM1ODQ2NTQxOTQ3NQ== 
```

The title decodes to the same tinyurl as above, while the keyword decodes to the following phone number: `+358[...]` (censored to avoid scraping bots). Running the phone number through maltego doesn't give anything useful.

After a lot of staring and being confused, I happened to press the "read aloud" button when viewing the PDF, and it happens to read invisible text which contain the next instructions:
```
Attention recruit,
you are about to undertake a battle of wits against the most 
mischievous minds in the Cyber Domain.
To know that you have what it takes, you must first reach Us through 
WhatsApp using the key phrase of the name of the pet bot of the 
Gen Z Discord server, this is how we know you are one of Us.
Good luck, hope you don’t need it.
The Operator
```

Conveniently enough, I'm already in the Gen Z Discord server and know that the bot's name is `StegoSiili`. After sending the message over WhatsApp, we get the following reply:
```
Good Job analyst. Now contact @challenge_ops
```
Due to the @ character, we know that the username is most likely either in Discord or Twitter. And as Gen Z server doesn't have that username, going to https://twitter.com/challenge_ops brings us a page of "The Operator", matching the footer of the recruiting message.

Messaging the twitter account gives us the flag `GENZ{oneofus}` and the description of the second part of the challenge.