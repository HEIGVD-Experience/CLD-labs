* What is the smallest and the biggest instance type (in terms of
  virtual CPUs and memory) that you can choose from when creating an
  instance?

```
The smallest one can have 1 vCPU and 0.5 GiB of RAM and the max one can have 448 vCPU and 6144 GiB of RAM.
```

* How long did it take for the new instance to get into the _running_
  state?

```
It can take few seconds as much as almost 5 minutes depending on many different factors.
```

* Using the commands to explore the machine listed earlier, respond to
  the following questions and explain how you came to the answer:

    * What's the difference between time here in Switzerland and the time set on
      the machine?
      
    ```
    bitnami@ip-10-0-9-10:~$ date
    Thu 14 Mar 2024 10:23:18 AM UTC

    The time set on the machine is in UTC despite our laptop being in CET.
    ```

    * What's the name of the hypervisor?
    
    ```
    On the console under the details of our instance we can find a section named *Virtualization type* and it's value *HVM*.
    ```

    * How much free space does the disk have?
    
    ```
    After running the command df -h
    bitnami@ip-10-0-9-10:~$ df -h
    Filesystem       Size  Used Avail Use% Mounted on
    udev             466M     0  466M   0% /dev
    tmpfs             96M  384K   95M   1% /run
    /dev/nvme0n1p1   9.7G  3.2G  6.1G  35% /
    tmpfs            476M     0  476M   0% /dev/shm
    tmpfs            5.0M     0  5.0M   0% /run/lock
    /dev/nvme0n1p15  124M   11M  114M   9% /boot/efi
    tmpfs             96M     0   96M   0% /run/user/1000
    We have 6.1G available.
    ```


* Try to ping the instance ssh srv from your local machine. What do you see?
  Explain. Change the configuration to make it work. Ping the
  instance, record 5 round-trip times.

```
ping 15.188.43.46
--- 15.188.43.46 ping statistics ---
13 packets transmitted, 0 packets received, 100.0% packet loss

It didn't work because no inbound rule is set to manage entry ping packet on the SSH server.

I doubt that we have rights to change SSH server parameters.
```

* Determine the IP address seen by the operating system in the EC2
  instance by running the `ifconfig` command. What type of address
  is it? Compare it to the address displayed by the ping command
  earlier. How do you explain that you can successfully communicate
  with the machine?

```
The ifconfig command does not exist so I used the ip a command.
bitnami@ip-10-0-9-10:~$ ifconfig
-bash: ifconfig: command not found
bitnami@ip-10-0-9-10:~$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: ens5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc mq state UP group default qlen 1000
    link/ether 06:44:74:63:9a:81 brd ff:ff:ff:ff:ff:ff
    altname enp0s5
    inet 10.0.9.10/28 brd 10.0.9.15 scope global dynamic ens5
       valid_lft 3398sec preferred_lft 3398sec
    inet6 fe80::444:74ff:fe63:9a81/64 scope link 
       valid_lft forever preferred_lft forever
```