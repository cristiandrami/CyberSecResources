A cron job is a scheduled task that is executed using a timer.

They are used to execute task repeteadly or in general with a specific schedule.

<mark style="background: #BBFABBA6;">We can see which scheduled jobs and cron there are in the system with:</mark>
```bash
cat /etc/crontab
```
- ![[Pasted image 20250117154755.png]]


The section `user` says to us which user is running them.


When a cron has all `*` it means it will be executed every minute:
- ![[Pasted image 20250117155144.png]]



So if we are able to modify these `.sh` files we can get a shell as root because they are run by `root` user


# Payloads all the thing
Useful exploit idea can be found in [https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/#scheduled-tasks](https://swisskyrepo.github.io/InternalAllTheThings/redteam/escalation/linux-privilege-escalation/#scheduled-tasks)
- ![[Pasted image 20250117155355.png]]
