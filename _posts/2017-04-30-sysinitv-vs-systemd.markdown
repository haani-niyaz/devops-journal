---
layout: post
title:  "Accidental Sysadmin Series: Sysinitv vs. Systemd"
date:   2017-04-30
author: "Haani Niyaz"
tags: 
 - linux 
 - sysadmin
---

Go beyond the usage basics to understand how `sysinitv` and `systemd` is configured.

## Sysvinit

When a linux operating system finishes booting the linux kernel, it begins to start various startup scripts to start background services, restore system state etc. The system state is determined by an id configured in `/etc/inittab` by the *sysvinit* program. This state is referred to as the **runlevel**. Each runlevel has a certain number of application services that are either **running** or **stopped**. 

The application services managed by init are located in `/etc/rc.d` directory. Within the `/etc/rc.d` directory there is a seperate directory for each runlevel. 

> The init program that runs as PID 1 spawns all other processes on unix-like systems.


### Run levels in linux:

{% highlight  python%}
{% raw %}
0 - System halt; no activity, the system can be safely powered down. 
1 - Single user; rarely used. 
2 - Multiple users, no NFS (network filesystem); also used rarely. 
3 - Multiple users, command line (i.e., all-text mode) interface; the standard runlevel for most Linux-based server hardware. 
4 - User-definable 
5 - Multiple users, GUI (graphical user interface); the standard runlevel for most Linux-based desktop systems. 
6 - Reboot; used when restarting the system.
{% endraw %}
{% endhighlight %}

On my Centos 6.5 machine the default run level is set `id:3:initdefault:` in the `/etc/inittab` file.

During the init process, the `/etc/rc.sysinit` file is run which picks up the default run level from `/etc/inittab`.


### Runlevel Directories

All runlevels executes the available scripts in `/etc/rcN.d` where `N` is the run level. So in my distribution runlevel scripts from `/etc/rc3.d` are executed. The files in this directory are symlinks to the service scripts in `/etc/rc.d`. 

**Note:** `/etc/init.d` is actually a symlink to `/etc/rc.d`


#### Why are file prefixed with `K` or `S`?

You will notice that files inside `/etc/rcN.d` are either prefixed with `K` or `S`. The idea is that if you are exiting a runlevel i.e: from 3 (Full multiuser mode) to 6 (reboot) all files begining with `K` are executed or in other words the service will be killed. Conversely when entering a runlevel files begining with `S` are started. 

#### Why do the files also have numbers suffixed to `K` and `S`?

Services with lower numbers execute first. For example the `network` service is executed before `sshd`.

{% highlight bash %}
{% raw %}
lrwxrwxrwx.  1 root root   17 Jan 16  2014 S10network -> ../init.d/network
lrwxrwxrwx   1 root root   14 Jan 16  2014 S55sshd -> ../init.d/sshd
{% endraw %}
{% endhighlight %}


### Putting it All Together

Applications that require a daemon at startup provide the necessary scripts which you can place in the `/etc/init.d`directory. Mostly this is configured for you when you install the application package such as a RPM.

To do this manually, say if you were building something from source, it would look like the following (I am using httpd as an example service):

1. Place service script in `/etc/init.d` dir as `/etc/init.d/httpd`
2. Add a symlink into runlevel 3 for startup:

{% highlight bash %}
{% raw %}
$ ln -s /etc/init.d/httpd /etc/rc.d/rc3.d/S99httpd
{% endraw %}
{% endhighlight %}

3. Add a symlink into the runlevel 0 dir for halt:

{% highlight bash %}
{% raw %}
$ ln -s /etc/init.d/httpd /etc/rc.d/rc0.d/K01httpd
{% endraw %}
{% endhighlight %}


However this is generally handled by the `chkconfig` tool which controls which services are to be started at which runlevels.


{% highlight bash %}
{% raw %}
$ sudo chkconfig --add httpd
{% endraw %}
{% endhighlight %}

When adding the runlevel via `chkconfig`, you will notice that it will create a symlink to `/etc/init.d/httpd` like `K02httpd` (You might have a different priority #) in every runlevel dir by default. This is to say that the `httpd` service will be stopped when leaving the runlevel.

To run the service when entering a runlevel you need to explicitly add it as follows:

{% highlight bash %}
{% raw %}
$ sudo chkconfig --level 3 httpd on
$ ls -la /etc/rc3.d/*httpd
lrwxrwxrwx 1 root root 16 Jan 26 05:35 /etc/rc3.d/S98httpd -> ../init.d/httpd
{% endraw %}
{% endhighlight %}


This will remove the runlevel file `/etc/rc3.d/K02httpd` (unless until you turn it off via `chkconfig`)

The following instructions are largely based on the following [article.](http://www.linuxvoodoo.com/resources/howtos/sysvinit)

## Systemd

### Characteristics

* The `service` command is replaced with `systemctl`.

* Systemvinit starts processes one by one. Systemd can start services or processes concurrently. 

* Systemd comes with journald which provides events and logging capabilities. On a side note, journald is lost when rebooted
because it is stored in memory.



### Units

A **unit** is any resource the system knows how to operate and manage. The resource is defined in a configuration file.

For an in-depth look at unit files see [this](https://www.digitalocean.com/community/tutorials/understanding-systemd-units-and-unit-files) article.

The follow commands shows available unit types:

{% highlight bash %}
{% raw %}
$ systemctl -t help

service
socket
busname
target
snapshot
device
mount
automount
swap
timer
path
slice
scope

{% endraw %}
{% endhighlight %}

#### What is a `service` unit?

> systemd service units are the units that actually execute and keeps track of programs and daemon, and dependencies are used to make sure that services are started in the right order. They are the most commonly used type of units.


### Targets

Sysinitv **runlevels** are replaced with **targets**. Targets are also a type of unit. 

Targets are located in `/etc/systemd/system`


### HTTPD Example

Install apache with `yum install -y httpd`.

This creates a unit file as shown below:

{% highlight bash %}
{% raw %}
$ cat /usr/lib/systemd/system/httpd.service 

[Unit]
Description=The Apache HTTP Server
After=network.target remote-fs.target nss-lookup.target
Documentation=man:httpd(8)
Documentation=man:apachectl(8)

[Service]
Type=notify
EnvironmentFile=/etc/sysconfig/httpd
ExecStart=/usr/sbin/httpd $OPTIONS -DFOREGROUND
ExecReload=/usr/sbin/httpd $OPTIONS -k graceful
ExecStop=/bin/kill -WINCH ${MAINPID}
# We want systemd to give httpd some time to finish gracefully, but still want
# it to kill httpd after TimeoutStopSec if something went wrong during the
# graceful stop. Normally, Systemd sends SIGTERM signal right after the
# ExecStop, which would kill httpd. We are sending useless SIGCONT here to give
# httpd time to finish.
KillSignal=SIGCONT
PrivateTmp=true

[Install]
WantedBy=multi-user.target
{% endraw %}
{% endhighlight %}


#### Check Status

{% highlight bash %}
{% raw %}
$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
{% endraw %}
{% endhighlight %}


Let's take a look at the following line from output above:

`Loaded: loaded (/usr/lib/systemd/system/httpd.service; disabled; vendor preset: disabled)`

This tells us the unit file is loaded but the service is `disabled`. This means apache will not automatically start on reboot.

`httpd.service` unit file we saw the following:

{% highlight bash %}
{% raw %}
[Install]
WantedBy=multi-user.target
{% endraw %}
{% endhighlight %}

The `multi-user.target` is what most daemons are grouped under. We can view this here:

{% highlight bash %}
{% raw %}
$ ls /etc/systemd/system/multi-user.target.wants

auditd.service    docker.service      NetworkManager.service  postfix.service   sshd.service     vboxadd-service.service
cadvisor.service  irqbalance.service  nfs-client.target       remote-fs.target  tuned.service    vboxadd-x11.service
crond.service     kdump.service       ntpd.service            rsyslog.service   vboxadd.service
{% endraw %}
{% endhighlight %}


You can see the depedencies by running the followng command:

{% highlight bash %}
{% raw %}
$ sudo systemctl list-depdencies multi-user.target
multi-user.target
● ├─auditd.service
● ├─brandbot.path
● ├─cadvisor.service
● ├─crond.service
● ├─dbus.service
● ├─docker.service
● ├─httpd.service
...
{% endraw %}
{% endhighlight %}


As you can see the `apache.service` is not present. So, let's enable the apache.

{% highlight bash %}
{% raw %}
$ sudo systemctl enable httpd.service
Created symlink from /etc/systemd/system/multi-user.target.wants/httpd.service to /usr/lib/systemd/system/httpd.service.
{% endraw %}
{% endhighlight %}


As you can see it creates a symlink inside `multi-user.target` dir to the `httpd.service` unit file.

If we check the status again, we will see that the service is now `enabled` to start on boot.

{% highlight bash %}
{% raw %}
$ systemctl status httpd.service
● httpd.service - The Apache HTTP Server
   Loaded: loaded (/usr/lib/systemd/system/httpd.service; enabled; vendor preset: disabled)
   Active: inactive (dead)
     Docs: man:httpd(8)
           man:apachectl(8)
{% endraw %}
{% endhighlight %}
	
For a more comprehensive list of commands see [RHEL7: How to get started with Systemd](https://www.certdepot.net/rhel7-get-started-systemd/).

## References

1. [What sets systemd apart from other init systems?](https://unix.stackexchange.com/questions/114476/what-sets-systemd-apart-from-other-init-systems)
2. [SysVinit explained: starting and stopping of services](http://www.linuxvoodoo.com/resources/howtos/sysvinit)
3. [RHEL7: How to get started with Systemd](https://www.certdepot.net/rhel7-get-started-systemd/)
