# Limitations
This documentation intends to address issues arrived at in limitations of using the BWMG and RTMPBee.

# Amount
The total concurrent amount limit on an instance and the resources that RTMPBee uses.

## Question
```sh
we have a large server that we are running a bee on but it seems to have problems when we go over 100 connection the error seems to be related to Too many open files do you know what is causing this or how to get around it?```
```

## Issue
The amount of file handles allowed by the system.

You can determine the amount of file handles opened for a current process by running the following:

```sh
$ cd proc/:PID/fd
$ ls -l | wc -l
```

> Run `ps aux | grep bee` to find the PID

Running the RTMPBee JAR with `8` stingers allocated, that provides a value of `1038`. Running with `2` stingers: `78`. That would leave us to believe that:

* Starting a bee spins up a base of `53` file handles
* Each subsequent stinger is an additional `10` file handles

### Instances
The server instance has an assigned hard file handle limit. To find that limit:

```sh
ubuntu@ip-10-153-249-17:~$ ulimit -a
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 14721
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 1024
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 14721
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```

You may need to up the hard limit in order to run more bees.

### Executor Limits
The stingers of a Bee are offloaded to threads an run in an executor. The current equation of the amount of concurrent stringers is: `CPUs * 8`.

If your instance has 1 CPU, you can only fire `8` stringers for a bee at a time. It is noted that the left over `2`, are in queue and fired as soon as threads become open again.

### Articles
* http://javarevisited.blogspot.com/2013/08/how-to-fix-javanetsocketexception-too-many-open-files-java-tomcat-weblogic.html
* http://linux.101hacks.com/unix/java-too-many-open-files/


