
| # | Evidence file (command) | What I expected | What I found | What I concluded |
|---|---|---|---|---|
| 1 | 05 ( systemctl --failed ) | nightly-backup listed as failed | one failed unit, ftp-mirror; nightly-backup is not listed | two questions: is ftp-mirror related, and does the backup job run at all? |
| 2 | 10 ( systemctl status ftp-mirror ) | it has something to do with backups | it uploads /srv/share to an old FTP server, error Name or service not known | it doesn't make backups, it just can't find the FTP server. Check if it's new |
| 3 | 11 ( journalctl -u ftp-mirror ) | if it's the cause, it started failing on Monday | it fails at every boot since Sep 14, same error | dead end , it was broken long before the backups stopped |
| 4 | 15 ( ls -lh /srv/backup ) | last backup before Monday | last file is from Nov 16 02:00. Nothing for Nov 17, 18, 19 | yes, backups stopped right at the reboot (Nov 16 ~08:11, from uptime in 02) |
| 5 | 09 ( journalctl -u nightly-backup ) | an error after Monday | runs fine every night until Nov 16 02:00, then nothing at all | the job didn't fail, it didn't start . Look at the timer |
| 6 | 07 ( systemctl status nightly-backup.service ) | failed | inactive (dead) , started by nightly-backup.timer | inactive is not an error, so the service itself is fine. Maybe that's why the manager thinks there were no errors |
| 7 | 06 ( systemctl list-timers ) | nightly-backup.timer with next run 02:00 | it's not in the list | the timer isn't running |
| 8 | 17 ( systemctl list-timers --all ) | the timer, but inactive | NEXT is - , LAST is Mon Nov 16 02:00 | there is no next run. The LAST time is from before the reboot. I think this is what the manager saw as "it ran" |
| 9 | 18 ( systemctl status nightly-backup.timer ) | enabled | disabled , inactive | disabled means it doesn't start after a reboot. This fits the ticket |
| 10 | 19 ( ls timers.target.wants ) | a link for nightly-backup.timer | only system timers, no nightly-backup | confirms it was never enabled |
| 11 | 21 (bash history of aigerim) | how the timer was set up | systemctl start nightly-backup.timer , but no enable . Then upgrade and reboot | found it. start only works until the next reboot |
| 12 | 20 (apt history) | the update broke something | on Nov 16 the kernel, tar and systemd were updated | the script uses tar, so maybe the new tar is broken. Need to check |
| 13 | 16 (run by hand) | an error if tar is broken | backup written (214M), exit code 0 | dead end , the script and tar work fine |
| 14 | 13 ( ls -l /usr/local/bin ) | wrong permissions | -rwxr-xr-x root , executable | dead end , permissions are fine |
| 15 | 03 ( df -h ) | disk full | 3.1G free on / | not the cause , there's space. But /srv/backup is on the same disk as the data, which is not a real backup (3-2-1 rule) |
| 16 | 08 ( systemctl cat service + timer) | wrong time or path | path correct, 02:00, Persistent=true , WantedBy=timers.target | the files are OK and the timer can be enabled, nobody did it |

Root cause
The timer was only started , never enabled (21-bash-history). That's why it shows disabled (18) and has no link in timers.target.wants (19). After the reboot on Monday it didn't come back, so the backup didn't run anymore (09, 15).

Fix
bash
sudo systemctl enable --now nightly-backup.timer
sudo systemctl start nightly-backup.service one backup right now

Also (not part of this ticket): ftp-mirror is broken since September and could be disabled after asking the owner. The backups should go to another disk, not the same one.

Verification
Check that the timer is enabled:
bash
systemctl is-enabled nightly-backup.timer expected: enabled
systemctl list-timers                      expected: nightly-backup.timer with NEXT 02:00

Check that a backup was made:
bash
ls -lh /srv/backup expected: a new file with today's date

After a reboot:
bash
sudo reboot
systemctl status nightly-backup.timer expected: enabled, active (waiting)

The next morning, ls -lh /srv/backup shows a new file from 02:00. This proves the timer started by itself after the reboot.