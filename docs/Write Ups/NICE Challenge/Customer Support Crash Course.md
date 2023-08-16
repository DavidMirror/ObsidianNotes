I first logged into the workstation since I knew from previous attempts that I could access the ticketing system. I clicked the osticket link on the desktop and logged into it on Firefox. I saw the first ticket from Brimlock Stones asking to mount the `employee_info` share on Workstation-Desk.

### Windows Mounting Share
To solve the Windows mounting share issue, I first opened explorer and opened the network sidebar and found that network discovery was turned off, and I enabled it once the yellow bar in explorer was available to press. I opened the file server and noticed that it wasn't serving `employee_info`, so we'd first need to remedy that. 
I logged into the file server, and after googling determined that I'd need to make changes to `/etc/samba/smb.conf`. I then opened that file in vim, copied the last entry section (`[HR]`) and changed the title to `[employee_info]` and made sure it was pointing at the correct path (`[employee_info]`). 

vim commands:
put cursor at the line with `[HR]`
`20yy` (yank the 20 lines including current (just an estimate but more than enough to copy the entire `[HR]` section))
`G`
`p`
move cursor to select the `H` in `[HR]`
`cw`, then type `employee_info`, `esc`
navigate to the line `path = /share/HR`
select the `H` in `HR`
`.` (to repeat command)
`:wq` , `enter`

I then returned to Workstation-Desk and could see the `employee_info` share there.  I right-clicked on it, then selected `Map Network Drive` , selecting `Z:` from the drop-down menu. I clicked Finish, then responded to the ticket and marked it resolved. 


### Remote Access
To allow remote access to Gilly Bates, I logged into the database server, searched "remote", and found "allow remote access". In there, I enabled it and added Gilly Bates as an authorized user. I then messaged Gilly Bates and marked the ticket resolved. 


### Backup Server Networking
I first started pinging 8.8.8.8. This didn't work, so I initially thought it was a routing issue. However, other computers on the network were also unable to `ping 8.8.8.8` but still access the internet, so I next suspected it could be a DNS issue. I checked `/etc/resolv.conf` on the backup server, and it pointed to `127.0.0.1`. I checked the same file on the file share server and it pointed to `172.16.30.5`, so I updated the backup resolv file. I then tried to run sudo apt update to solve the issue, and it worked. Finally, I responded to the ticket from Workstation-Desk and marked it resolved. 

### Mail Server Networking
To resolve the backup server networking, I started pinging the internet, then other networks. All returned network unreachable. I suspected an improperly configured gateway, and after some googling I found an `ip route` command to define a default gateway.  I used the  settings from the backup server, I ran `sudo ip route add default via 172.16.30.2 dev eth0` , rechecked the pings, and found them to work.  Finally, responded to the ticket from Workstation-Desk and marked it resolved. 



### Mounting Share in Linux
Mounting Linux:
I found again that the share that they want `archive` was not currently being served. I found that the `archive` folder was on the `Fileshare` server, and repeated the steps in the Windows Mounting Share section to enable it. I checked that it was available from Workstation-Desk, and performed the following commands from the Backup server.

`sudo mkdir /mnt/backups`
`sudo mount -t cifs //172.16.30.32/archive /mnt/backups`
This command did not initially work for me, giving me an invalid filesystem type error. I tried to install `cifs-utils` and `psmisc` as a mounting smb shares tutorial had indicated. This fixed that error, but I was still getting a `mount error(6): No such device or address`. It turned out that I had named everything the singular `archive` instead of the plural `archives`, so the smb server was trying to serve a nonexistent folder. After cleaning up `/etc/samba/smb.conf`, I reran the updated command:
`sudo mount -t cifs //172.16.30.32/archives /mnt/backups`, which ran without errors.
I then ran `mount -t cifs` to verify that it was mounted and it was.
Finally, responded to the ticket from Workstation-Desk and marked it resolved. 