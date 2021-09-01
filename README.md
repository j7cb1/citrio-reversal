# citrio-reversal
Archived reversal of citrio
This reversal is over 6 months old and is being stored here.

# Reversal of the p2c citr.io
This reversal contains information about the loaders protection and the payloads protection/management, It also contains information about how the loader communicates with their web scripts to authenticate users to the loader and how it can be exploited to invoke unlimited subscriptions and access to the client.

# Protection
Both the rage and legit cheat have been (protected) packed using UPX 3.94 (NRV, brute). UPX is a public packer that comes with an unpacker; Sadly the unpacker has a bug and screws up one of the imports preventing one from running the unpacked version; (I did very little debugging to fix this other than seeing what was causing the crash. So sure one could un-pack and either jmp or change one or two jnz to jz to bypass the authentication stages. But the unpacked binary is all you really need to reverse what they are doing to authenticate the user, map the drivers and giggle at. The developer has not encrypted any strings at all.

# Authentication 
After un-packing the binary initial inspection of the 'main' routine lead me to believe that everything required for the cheat is client side and all that the loader relys on is simple text responses such as sub duration and whether the user exists. They have tried to protect them selves from this by creating a snap shot and walking the processes to see if the name matchs 'fiddler.exe'.

![image](https://user-images.githubusercontent.com/64629096/116180818-a6fffc00-a76d-11eb-8406-14292143796b.png)

This is yet again no issue because httpDebugger > fiddler, but still the work around is extremely simple. After logging in with something random I found the 'doesnt exist' code and found out that nothing is hashed, I also picked up on the 'HWID' format looking a bit familiar. 

![image](https://user-images.githubusercontent.com/64629096/116180840-b121fa80-a76d-11eb-87e0-8c4f803d24ad.png)

So already at this stage If I really wanted to share subscriptions with a friend it would be very easy. I also had a quick look to see how the response code is managed.

![image](https://user-images.githubusercontent.com/64629096/116180884-c565f780-a76d-11eb-9741-4518e1d1f210.png)

So simply setting a rule for (https://auth.citr.io/loader/checks.php) to respond with '206' or '205' gets you a valid login. The subscription period is managed more or less the same way, all one needs to do is set another rule to respond with anything that isn't '0' 

![image](https://user-images.githubusercontent.com/64629096/116180910-d0b92300-a76d-11eb-9f43-6a6ad1925a70.png)

the expirey date does nothing other than look cool in the loader. Another little thing they have is (https://auth.citr.io/loader/sub.php) This is used to distinguish whether the user has access to the 'Rage' or 'Legit'. The code for rage is '11' and the code for legit is '4'. Then after all that you are in but once in the loader they have a heart beat to detect whether the user's subscription has expired whilst the loader is open. But is isn't threaded and is only 'beating' when in the user info tab. tbh I don't even think its a heart beat because I never got exited for having '0', All I saw from this is the client pinging their server like mad.

![image](https://user-images.githubusercontent.com/64629096/116180932-d6af0400-a76d-11eb-9f0c-d8c972fde9d9.png)

They also had a bug in their loader to spam their web scripts which crushed fps and website performance.

# Payload
When having a quick gander at the strings in IDA I picked up on a few things the first being a few IAT's contained inside the loader and the use of KDmapper. There is a few approaches to this now the exciting way and the meh way. Because I knew there was a few PE's stored inside the client I used ByteStinker (credit: bright) to obtain the PE files; I did end up using KDstinker to get the main kernel driver because the one I pulled out of the process wasn't mapping happily. 3 PE's were pulled from the loader the first being iqvw64e.sys for KDmapper the second Kernelmode-RAGE.sys and the third zz.sys (the spoofer); I got the names from the .pdb directories (C:\\Users\\Carson\\Desktop\\KERNV2\\stuffs - Copy\\builds\\Kernelmode\\Kernelmode-RAGE.pdb) so as you can see the bro carson has been wildin. Ill start witht the spoofer because it isn't his it's just btbd's spoofer,

![image](https://user-images.githubusercontent.com/64629096/116180957-df9fd580-a76d-11eb-8b2d-ace247e3cdf6.png)

and because it's not his it mapped very happily so now I have his spoofer ... yay . The next exciting piece is the main kernel driver, initially I thought that he would  have some type of communication to the usermode because I did see R6 related shit in the strings of the loader but that was because the kernel driver was stored inside the loader. It turns out it's a kernel mode cheat, exciting right? then I saw,

![image](https://user-images.githubusercontent.com/64629096/116180778-9bacd080-a76d-11eb-898f-c01f0b1f8deb.png)

So yeah not much exciting shit here just to skip through shit this driver does not clear the PiDDBCacheTable or MmUnloadedDrivers but the spoofer does so maybe thats fine but since this cheat is threaded the loader cannot use KDmapper to map the spoofer to clear that for the cheat. But basically what it does once the main routine consists of a whole bunch of gay shit starting with getting the users config from the server this is one of the annoying things because at this stage Ive ditched using httpsDebugger and plus I wouldn't even be able to use it in this instance because the web requests aren't picked up.

![image](https://user-images.githubusercontent.com/64629096/116180997-ee868800-a76d-11eb-9062-f8c1f751f4b2.png)

Also I just want to point out that this is called 3 times. The threaded subRoutine is again nothing really worth looking into I just took a brief glance, all it does is wait for the process 'RainbowSix.exe' then sits looping doing anything, I really didn't go any further at this point.

![image](https://user-images.githubusercontent.com/64629096/116181333-8e441600-a76e-11eb-9c13-2f226eb83b2e.png)
![image](https://user-images.githubusercontent.com/64629096/116181355-9b610500-a76e-11eb-979a-1d05f575be9e.png)




