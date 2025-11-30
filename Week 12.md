# Q1 

This is a Android binary XML. 

# Q2

The most common file is the png file. 

# Q3

Yes, the AndroidManifest is now an xml file. 

# Q4 

Some activities and services have an intent-filter with an action and category already defined, like com.cp.camera.Loading, com.cp.camera.BootService

# Q5

Classes like com.cp.Camera.BootService/Loading, and subclasses. 

# Q6

The parameters are as follows: [String](https://developer.android.com/reference/java/lang/String) destinationAddress, 
                [String](https://developer.android.com/reference/java/lang/String) scAddress, 
                [String](https://developer.android.com/reference/java/lang/String) text, 
                [PendingIntent](https://developer.android.com/reference/android/app/PendingIntent) sentIntent, 
                [PendingIntent](https://developer.android.com/reference/android/app/PendingIntent) deliveryIntent, 
                long messageId)
# Q7

The destinationAddress parameter corresponds to the destination address. 

# Q8

The onCreate function reaches out to some telephone "operator",  which makes a POST-bassed login, returning a json-string with the field "service", that is then set as the destination address. 

![[Pasted image 20251119183231.png]]

# Q10

getDeviceId(), getPhoneNumber(), and onRequestPermissionsResult()

# Q11

onRequestPermissionsResult(), it even makes a text asking nicely!

# Q12

Some exported activities/services are com.dataeye.channel.DCAppService, com.app.myfolder.activity.FoldersAppUpdateActivity, com.app.myfolder.activity.FoldersSearchActivity, com.app.myfolder.activity.FoldersAppDetailActivity, com.app.myfolder.activity.FoldersAddAppActivity, com.app.myfolder.activity.FoldersBoxActivity. 

# Q13

`Runtime.exec` can be used to execute Linux commands. 

# Q14

This is used a couple times, one on `cat /sys/class/net/wlan0/address` (just getting that interface's mac address) and another that allows execution of a string. This second option, originally named a, is called by function i, which passes in the argument "/system/bin/sh". Since the original function a also accepts an ArrayList, and appears to pass this list to a buffered reader tied to a process running the shell command.

# Q15 
This second usage could be vulnerable to attacker manipulation, since attackers might have a way to inject input into this ArrayList, which would result in attacker commands being executed through this spawned shell. 

![[Pasted image 20251119190816.png]]
![[Pasted image 20251119190831.png]]
# Q16

This is in class b, in the onTransact method. 
![[Pasted image 20251119191135.png]]

# Q17

The action android.intent.action.AdupsFota.operReceiver triggers this call chain. 

# Q18

The string list comes from intent.getStringExtra, retrieving string data attached to that intent tied to the "cmd" key. 

![[Pasted image 20251119191346.png]]