# PowerShell
It is a cross-platform task automation solution mad up of command-line shell, scripting language and a configuration management framework.

Poweshell comes installed by default in any modenr windows OS.

It is not an .exe but it is a library .dll 'System.Management.Automation.dll'.

### Powershell scripts and modules
We can load a PowerShell script using:
- dot sourcing
	- `. C:\AD\Tools\PowerView.ps1`
- A module or a script can be imported with:
	- `Import-Module C:\AD\Tools\ADModule-master\AcriveDirectory\ActiveDirectory.psd1`
- All commands in a module can be listed as:
	- `Get-Command -Module <name>`


We can also adownload execute cradle:
- ![[Pasted image 20261116102547.png|505]]
	- This is very abused, since it is the ivkoe expression.

In facts, windows intuced soome protection mechanisms.
### Powershell detections
These mechanisms are:
- System wide transcription, this stores in a file everything that is digited and visualized in the powershell console
- Script Block logging, it storeds the scripts when powershell decoded it, so it stores the real payload deobfuscated.
- Antimalware Scan Interface (AMSI), it connects powershell with the antivirus
- Constrained Language Mode (CLM), integrated with applocker and WDAC (Device Guatrd)
	- It blocks the mnost dangeours .net functions, but it is activated only on non-trusted scrips (which are not trusted by the applocker and WDAC)


### Powershell tradecraft
Offensive powershell is not dead.
The detections depend on the organization and they might use customized code.

There exist bypasses and obfuscated bypasses.


### Bypass Powershell Security

We can use `Invisi-Shell` [https://github.com/omerya/invisi-shell] for bypassing the security controls.

This tool hooks the .NET assemblies (`System.Management.Automation.dll` and `System.Core.dll`) to bypass logging.

It uses CRL profile API to perform the hook.
- CRL is a common language runtime and is a dynamic link library `.dll` that consists of functions that receive messages fromm, and send messages to the CRL by using the profiling API. 
- The profiler DLL is loaded by the CRL at run time.


We can use it with Admin privileges 
- `RunWithPathAsAdmin.bat`

Or with no admin privilages:
- `RunWithRegistruNonAdmin.bat`

If you type `exit` from the new Powershell session a clean-up is performed.


### Bypass AV signatures for powershell
We can also load scripts and avoid detection bypassing AMSI.

The idea is to use `DefenderCheck` to identify code and strings from binaries file or scripts that Windows Defender may flag.
- So we can remove the portion of script detected and that is not used 

We can use the `ByteToLineNumber.ps1` script in `\AD\Tools` folder to understand in which line the script is detected:
- ![[Pasted image 20261116105410.png]]
	- Here there is `0x1DCD2`
With `ByteToLineNumber.ps1` we can find the right line:
- ![[Pasted image 20261116105500.png|569]]


And using the editor we can go to the line after it to understand which part is detected.
In this example it is a base64 function:
- ![[Pasted image 20261116105752.png|452]]

So we could remove it we don't need that portion and try to understand if after removing it we have the problem with the checker:
- ![[Pasted image 20261116105850.png]]



### Bypass AV signature with MIMIKATZ
`Invoke-Mimikatz` is one of the most heavily signatured Powershell script.
In this case we need multiple steps to evade Windows Defender.


# Offensive .NET - tradecraft

When using .NET there are some challenges:
- Detection by countermisures like AV, EDR
- The delivery of the payload ( recall powershell's downloaded-execute cradles)
- the detection by logging, like the process creation loggin ore command line logging etc.

### AV bypass - Source Code Obfuscation
There are also tool used to obfuscate code in order to bypass the AV signature detection:
`Codecepticon` [https://github.com/Accenture/Codecepticon]
-  it needs to be compiled in VS Code and the used in command line to generate obfuscated code.

The idea is to:
1. Compile the project in VS code
2. navigate the output directory
3. open `CommandLineGenerator.html` and use it to the obfuscation
	1. ![[Pasted image 20261116111833.png|476]]



We can also use the command to obfuscate it:
- ![[Pasted image 20261116111914.png]]


You can see that everything is changed in the code:
- ![[Pasted image 20261116112000.png|405]]

### AV Bypass - Binary obfuscation
Once compiled it we can also use `ConfuserEx` [https://mkaring.github.io/ConfuserEx/] to obfuscate the compiled binary.

It is a free .NET obfuscator that can help us to sto AV from performing signature based cdetection.


So what we can do is:
1. Run it from `\AD\Tools` directory
2. Add the `Release folder` of the compiled binary
	1. ![[Pasted image 20261116112412.png|473]]
3. Add a rule and on it and on the rule add `Preset: Maximum`
	1. ![[Pasted image 20261116112503.png|460]]
4. Click `Protect` to produce the obfuscated binary and verify it with `DefenderCheck`
	1. ![[Pasted image 20261116112607.png|464]]



## Offensive .NET - Payload Delivery

We can deliver our payload with `NetLoader`, it can be used to load binary from filepath or URL and patch AMSI & ETW while executing.

Usage example:
- ![[Pasted image 20261116113402.png|484]]

So we use NetLoader obfuscated with Codeception and ConfuserEx to evade Windows Defender.


# Methodology and Lab material

## Assume Breach
The methodology used in the CRTP is basically this:
- ![[Pasted image 20261116114128.png|575]]



## Lab Environment
The lab of CRTP is a fictional enterprise that has:
- fully patched Server 2022 with windows defender
- Server 2016 forest functional level
- multiple forests and mutiple doamin
- Minimal firewall usage


To access the lab you can use: [https://www.enterprisesecurity.io/auth?]
