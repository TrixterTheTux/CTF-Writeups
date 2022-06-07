# Securesigner
```
You can use our secure signer service to sign your numbers securely.

The used crypto algorithm also supports decrypting our secrects so we don't allow you to input our secret numbers. Feel free to input any other number!

(You can use cyberchef to turn numbers into text: https://gchq.github.io/CyberChef/#recipe=To_Base(16)From_Hex(%27Auto%27) )
https://securesigner.challenge.fi/ 
```

We're given a website which allows us to decrypt arbitrary inputs using the following constants:
```
n = 27657250615199756991908071755730368175386649641863130140043208937153921937676995595979332191751868044424610138413747523524572894735791519525288854524674265877467142473327810518974365701123663469215445489528132452538745033701256137842151028680785299326651087730812834721910556635354671176047393016563858480053276758153507503625634998844595319904330075074872912962038844918719851827364791402329440142181271630051238362119277234319921329919007440655634926874094190410101539490446681737245249085323548016550482239126244449656834087709208986022530274192018120666379028740159178861684993330266093703369485885510565186948479
e = 65537
```

And the encrypted flag:
```
flag_encrypted = flag ^ e mod n = 11594314584695234882122705061649945447368315257039626274140437579759638688025292681062809998434420317618066727301021964608966990020934167899905298305310215490347689693310599680189407996278722251202704302022419789350179003594306621316838751372268690552907608008671626809061166504474966651958437366963607014537591571204100790719083548328188416132785461778076720285484094889137313885177253514294279601355687642487445867199008351732325839623548963542473485408616181569247059244234411557017628724549762320231349894340022586610945922446560728674078091196391788680661438872804275390671184195671539679849749274683627529413423
```

Our goal is to decrypt `flag_encrypted`, which if we put directly into the website we'll notice will fail due to being blacklisted. Though if we were able to rewrite `flag_encrypted` into another format, we'd be able to convert it back after the decryption, for example multiplying the `flag` value by two:
```
(2 * flag) ^ e mod n
=> (flag ^ e mod n) * (2 ^ e mod n)
```

Then we can use the Python REPL to calculate the multiplied `flag_encrypted`:
```py
>>> flag_encrypted * pow(2, e, n) % n
7191751314881103659897328860072043992465807458609906502933809296214377124471918214466235109193995439521619443372283766045946541819699108204640641216727313788520369195946051564438834171184965484373309249376757510951660022586928118494148194621218203256601023467093856231278712834853926879414138352701231794749355733036405537633018043564066710802922015107353636453006760899268605285259234227165286863579184537671798909434827464846108885254428933995842013450474214918875038382451069755167401641865978071434883920397502616241486479774443570261151735973947094241565269574417088250048644933909985144900674675975072874021304
```

Which we can decrypt with the securesigner website:
```
2 * flag = 1189323035521573969825971868696161838874305655853421762138459158800913752449570354536005157177594
```

And calculate the division with Python REPL (note the double slash as otherwise we lose precision):
```py
>>> 1189323035521573969825971868696161838874305655853421762138459158800913752449570354536005157177594 // 2
594661517760786984912985934348080919437152827926710881069229579400456876224785177268002578588797
```
Then we just put it into CyberChef, outputting our flag `GENZ{i_could_do_this_algebra_in_my_head}`.
