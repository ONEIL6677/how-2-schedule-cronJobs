**THIS IS THE SIMPLIED VERSION FOR A QUICK REMINDER

first chech if you have any cron jobs by running this command

```bash
crontab -l
```

if you dont know the path of your file use this command 
because you will in it beloow

```bash
sudo find / -name "filename.txt"
```

To create a cron job run the following command

```bash
crontab -e
```

then it will ask for the editor you want to use. 
you can press enter to continue with default editoor


scroll below and create a cron job that
execute a particular file everyday at 20:30 pm

###############**CODE**################

30 14 * * * /bin/sh /home/users/desktop/list-users.sh

**SAVE and QUIT
this job will execute your command respicting the time you set

NB your cron will not run if your computer is off except in was schedule on the cloud instance 

