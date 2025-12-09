# Clean & Correct Procedure to Register Linphone (CLI) With FreeSWITCH

## 1. Create Linphone config directory
```
mkdir -p /root/.local/share/linphone/
chmod 700 /root/.local/share/linphone/
```


Check:
```
ls -l /root/.local/share/linphone/
```

## 2. Generate the HA1 MD5 (for Digest Authentication)

Syntax for HA1 MD5:

username:realm:password


In FreeSWITCH, the realm = IP address of your FS server
Example: For extension 1003 with password 1234 on FS <freeswitchIP>:
```
printf "1003:<freeswitchIP>:1234" | md5sum
```

Example output:

de67bc826f4f51323932f67dc9f4b8a6  -


(This matches the value you got.)

3. Create ~/.linphonerc File

Example correct .linphonerc content:
```
[auth_info_0]
username=1003
ha1=de67bc826f4f51323932f67dc9f4b8a6
realm=<freeswitchIP>
algorithm=MD5

[proxy_0]
reg_proxy=sip:<freeswitchIP>
reg_route=<sip:<freeswitchIP>;lr>
reg_identity=sip:1003@<freeswitchIP>
reg_expires=3600
reg_sendregister=1
publish=0
idkey=proxy_config_gb4PszdKAqJuxoH

```

✔ Make sure username inside auth_info_0 matches reg_identity user.

⚠ You had a mismatch earlier:

username=1001

identity=sip:1005...

That would cause registration issues.

4. Start linphonec and load config

Run:
```
linphonec
```

(To ensure it loads .linphonerc.)

Then verify proxy:

proxy list

5. For persistent registration using linphonecsh

Terminate interactive client:
```
pkill -9 linphonec
```

Initialize:
```
linphonecsh init
```

Register:
```
linphonecsh register --host <freeswitchIP> --username 1003 --password 1234
```

Check:
```
linphonecsh status register
```

Expected:
```
registered, identity=sip:1003@<freeswitchIP> duration=3600
```
🚨 Why your previous registration showed wrong identity?

You saw:
```
registered, identity=sip:1001@<freeswitchIP>
```

This happens when:

.linphonerc still contains old config

linphonecsh is not reading new identity

multiple proxy entries exist

Fix:
```
linphonecsh unregister --host 10.58.181.138
rm ~/.linphonerc
```

Then recreate the .linphonerc with correct values.

🎥 About the Video Errors You Saw Earlier

Error:

LinphoneCore has video disabled for both capture and display, but video policy is to start the call with video.


This happens when:

Video is disabled in .linphonerc

But call policy says: "start video automatically"

Fix by adding into .linphonerc:

```
[video]
display=0
capture=0
automatically_initiate=0
automatically_accept=0
```
Or simply disable video in commands:

```
linphonec> vcap off
linphonec> vdisplay off
```
