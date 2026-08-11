Linux Troubleshooting Runbook — SSH Service
Target Service

Service: ssh / sshd

Goal: Check the system's basic health, verify SSH availability, inspect SSH logs, and identify what to do if the problem worsens.

1. Environment Basics
#Command 1 — Kernel and system information
 uname -a
 
 Output:
 Linux devops 7.0.0-1010-aws #10-Ubuntu SMP PREEMPT Thu Jul 23 02:06:04 UTC 2026 x86_64 GNU/Linux

 Observation: Confirms the running Linux kernel, architecture, hostname, and system information.

#Command 2 — Operating system information
 lsb_release -a
 
 Output:
 ubuntu@devops:~$ lsb_release -a 
 No LSB modules are available.
 Distributor ID:	Ubuntu
 Description:	Ubuntu 26.04 LTS
 Release:	26.04
 Codename:	resolute

 Observation: Confirms the Ubuntu distribution and release currently running.

2. Filesystem Sanity
#Command 3 — Create a temporary test directory
 mkdir /tmp/runbook-demo

 Output:
 ubuntu@devops:~$ ls /tmp
 runbook-demo
 snap-private-tmp
 systemd-private-88f774a973f24399836e56897aabd391-ModemManager.service-GXigCe
 systemd-private-88f774a973f24399836e56897aabd391-chrony.service-PioUia
 systemd-private-88f774a973f24399836e56897aabd391-irqbalance.service-Me1FD4
 systemd-private-88f774a973f24399836e56897aabd391-polkit.service-zoLX90
 systemd-private-88f774a973f24399836e56897aabd391-redis-server.service-3rQI0g
 systemd-private-88f774a973f24399836e56897aabd391-systemd-logind.service-ICiXG9

 Observation: No output normally means the directory was created successfully.Did 'ls' and saw the folder has been created.

#Command 4 — Copy a file and verify it

Output:
ubuntu@devops:~$ cp /etc/hosts /tmp/runbook-demo/hosts-copy && ls -l /tmp/runbook-demo
total 4
-rw-r--r-- 1 ubuntu ubuntu 221 Aug 11 14:19 hosts-copy

 Observation: The file was successfully copied, confirming basic filesystem read/write operations are working.

3. CPU and Memory
#Command 5 — Check running processes

First identify the SSH process:

ps aux | grep '[s]shd'

Then inspect its CPU and memory usage:

ps -o pid,pcpu,pmem,comm -p <PID>

OUtput:
ubuntu@devops:~$ ps aux | grep '[s]shd'
root         690  0.0  0.8  10716  7576 ?        Ss   13:26   0:00 sshd: /usr/sbin/sshd -D -o AuthorizedKeysCommand /usr/share/ec2-instance-connect/eic_run_authorized_keys %u %f -o AuthorizedKeysCommandUser ec2-instance-connect [listener] 0 of 10-100 startups
root        1303  0.0  1.2  18256 11776 ?        Ss   13:28   0:00 sshd-session: ubuntu [priv]
ubuntu      1422  0.0  0.8  18416  7864 ?        S    13:29   0:00 sshd-session: ubuntu@pts/0
ubuntu@devops:~$ ps -o pid,pcpu,pmem,comm -p 690
    PID %CPU %MEM COMMAND
    690  0.0  0.8 sshd

Observation: SSH is using approximately 0.0% CPU and 0.8% memory. No abnormal resource consumption was observed.

#Command 6 — Check system memory
 
ubuntu@devops:~$ free -h
               total        used        free      shared  buff/cache   available
Mem:           908Mi       352Mi        82Mi       2.8Mi       582Mi       555Mi
Swap:             0B          0B          0B

Observation: System has sufficient available memoryof 555 MB. Check available rather than only free when judging memory pressure.

4. Disk and I/O
#Command 7 — Check filesystem usage
 df -h

 Output:
 ubuntu@devops:~$ df -h
Filesystem       Size  Used Avail Use% Mounted on
/dev/root         19G  3.2G   16G  18% /
tmpfs            455M     0  455M   0% /dev/shm
tmpfs            182M  940K  181M   1% /run
efivarfs         128K  3.3K  120K   3% /sys/firmware/efi/efivars
tmpfs            455M  4.0K  455M   1% /tmp
none             1.0M     0  1.0M   0% /run/credentials/systemd-journald.service
none             1.0M     0  1.0M   0% /run/credentials/systemd-resolved.service
/dev/nvme0n1p13  989M  163M  760M  18% /boot
/dev/nvme0n1p15  105M  6.3M   99M   7% /boot/efi
none             1.0M     0  1.0M   0% /run/credentials/systemd-networkd.service
none             1.0M     0  1.0M   0% /run/credentials/getty@tty1.service
none             1.0M     0  1.0M   0% /run/credentials/serial-getty@ttyS0.service
tmpfs             91M  8.0K   91M   1% /run/user/1000

Observation: Root filesystem is approximately 18% full. No filesystem is critically full.

#Command 8 — Check log directory size
sudo du -sh /var/log

Output:
ubuntu@devops:~$ sudo du -sh /var/log
31M	/var/log
Observation: /var/log is using approximately 31 MB. No unexpectedly large log usage was observed.

5. Network
#Command 9 — Check listening services
sudo ss -tulpn | grep ':22'
ubuntu@devops:~$ sudo ss -tulpn | grep ':22'
tcp   LISTEN 0      4096             0.0.0.0:22         0.0.0.0:*    users:(("sshd",pid=690,fd=3),("systemd",pid=1,fd=104))                      
tcp   LISTEN 0      4096                [::]:22            [::]:*    users:(("sshd",pid=690,fd=4),("systemd",pid=1,fd=105))   

Observation: SSH is listening on port 22 / the configured SSH port.

#Command 10 — Test the SSH service locally
curl -I http://localhost:22

ubuntu@devops:~$ curl -I http://localhost:22
curl: (1) Received HTTP/0.9 when not allowed

Observation: curl is not an ideal protocol-level test for SSH, so an HTTP error is not evidence that SSH is broken. The important check is that SSH is listening on its TCP port.

A better SSH-specific test is:

ssh -v localhost

debug1: Connecting to localhost [127.0.0.1] port 22.
debug1: Connection established.
debug1: Remote protocol version 2.0, remote software version OpenSSH_10.2p1 Ubuntu-2ubuntu3.5
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received

Observation: SSH successfully connected to localhost:22 and completed the initial SSH protocol negotiation and key exchange. The connection reached host-key verification, confirming that the SSH daemon is reachable and responding normally. 
The test was manually stopped before authentication.

6. Logs
#Command 11 — Review SSH service logs
sudo journalctl -u ssh -n 50 --no-pager

Output:
(1). SSH started successfully after boot

Aug 11 13:26:16 devops sshd[690]: Server listening on 0.0.0.0 port 22.
Aug 11 13:26:16 devops sshd[690]: Server listening on :: port 22.
Aug 11 13:26:16 devops systemd[1]: Started ssh.service

This confirms SSH is listening on both IPv4 and IPv6 port 22.

(2). Your login was successful

Accepted publickey for ubuntu from 103.173.241.191 ...
pam_unix(sshd:session): session opened for user ubuntu

That's your successful public-key authentication.

(3). There are random internet connection attempts

For example:

Connection closed by 181.46.69.68
Connection closed by 66.62.126.69
Connection closed by 138.186.29.107

Because your EC2 instance has SSH exposed to the internet, automated scanners/bots can discover port 22 and attempt connections. A Connection closed entry by itself does not mean they successfully logged in.

Importantly, your logs show Accepted publickey for ubuntu for your legitimate login, rather than an accepted login from those other addresses.

(4). This line is interesting

banner exchange: Connection from ::1 port 49038: invalid format

::1 is IPv6 localhost (127.0.0.1's IPv6 equivalent). This likely corresponds to something on the machine connecting to port 22 without speaking the SSH protocol correctly.

Observation: SSH started successfully and is listening on ports 22 for IPv4 and IPv6. Recent logs show successful public-key authentication for the ubuntu user and several unsolicited external connection attempts that were closed without successful authentication.
No SSH service failure is evident.

#Command 12 — Review authentication logs
sudo tail -n 50 /var/log/auth.log

SSH started normally:

sshd[690]: Server listening on 0.0.0.0 port 22.
sshd[690]: Server listening on :: port 22.

SSH is listening on IPv4 and IPv6.

Your successful login:

Accepted publickey for ubuntu from 103.173.241.191

This is the important successful authentication event.

External connections:

Connection closed by 181.46.69.68
Connection closed by 66.62.126.69
Connection closed by 138.186.29.107
Connection closed by 195.80.140.10

These connections were closed, but there is no Accepted event associated with them in this output. Don't call them confirmed attacks; they are simply unsolicited/unsuccessful connections based on the available log entries.

Your localhost testing:

banner exchange: Connection from ::1 ... invalid format
Connection closed by 127.0.0.1 ... [preauth]

These correspond to local connections that did not complete SSH authentication. The ::1 address is IPv6 localhost and 127.0.0.1 is IPv4 localhost.

Normal system activity:

CRON[...] pam_unix(cron:session): session opened for user root
CRON[...] pam_unix(cron:session): session closed for user root

Normal scheduled-task activity.

And your own commands appear in the log:

sudo: ubuntu : ... COMMAND=/usr/bin/du -sh /var/log
sudo: ubuntu : ... COMMAND=/usr/bin/ss -tulpn
sudo: ubuntu : ... COMMAND=/usr/bin/journalctl -u ssh -n 50 --no-pager
sudo: ubuntu : ... COMMAND=/usr/bin/tail -n 50 /var/log/auth.log

That's actually a good demonstration of why /var/log/auth.log is useful: it records authentication and privilege-related activity, including sudo.

Observation:
Recent authentication activity shows successful public-key
authentication for ubuntu, normal sudo and cron activity, and
several closed external connections. No unauthorized successful
login is evident in these 50 entries.

## Overall health: SSH appears healthy because the service is running, the SSH port is listening, system resources are within normal limits, and no critical errors were found in the recent logs.

If This Worsens
1. Check and restart the service
sudo systemctl status ssh
sudo systemctl restart ssh
sudo systemctl status ssh

If the restart fails:

sudo journalctl -u ssh -n 100 --no-pager
2. Check SSH configuration before making changes
sudo sshd -t

If configuration validation reports an error, fix the configuration before restarting SSH again.

3. Collect deeper diagnostic information

If the problem remains:

sudo journalctl -u ssh --since "30 minutes ago"
sudo ss -tnp | grep ':22'
ps aux | grep '[s]shd'

For a persistent process-level problem, consider collecting a system-call trace with strace after confirming that deeper tracing is appropriate.

Cleanup

The temporary test directory can be removed after the drill:

rm -rf /tmp/runbook-demo

Final status: SSH troubleshooting drill completed. System health,filesystem, resources, network connectivity, and SSH logs were checked.
