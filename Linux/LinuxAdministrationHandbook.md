# Linux Administration Handbook
## Booting and System Management Daemons
### Boot Process Overview
* admins can modify bootstrap configurations by editing config files for the system startup scripts or by changing the arguments the boot loader passes to the kernel.

### System Firmware
* The UEFI boot target can be a UNIX or Linux kernel that has been configured for direct UEFI loading, thus effecting a loader-less bootstrap
* UEFI defines standard APIs for accessing the system’s hardware.

### Boot Loader
* The boot loader’s main job is to identify and load an appropriate operating system kernel.
* the argument single or -s usually tells the kernel to enter single-user mode instead of completing the normal boot process

### GRUB
* GRUB understands most of the filesystems in common use and can usually find its way to the root filesystem on its own.
* configuration is specified in /etc/default/grub in the form of sh variable assignments.
* When specified at boot time, kernel options are not persistent. Edit the appropriate kernel line in /etc/grub.d/40_custom or /etc/defaults/grub (the variable named GRUB_CMDLINE_LINUX) to make the change permanent across reboots.
* the new kernels are installed side by side with the previous versions so that you can return to an older kernel in the event of problems.

### System Management Daemon
* Once the kernel has been loaded and has completed its initialization process, it creates a complement of “spontaneous” processes in user space.
* process ID 1 and usually runs under the name init
* init has multiple functions, but its overarching goal is to make sure the system runs the right complement of services and daemons at any given time. ==> maintains notion of the mode in which the system should be operating.
* init normally takes care of many different startup chores as a side effect of its transition from bootstrapping to multiuser mode
* In the traditional init world, system modes (e.g., single-user or multiuser) are known as “run levels.”
* run-level-specific directories that indicate what services are supposed to be running at what run level
* this system has no general model of dependencies among services, so it requires that all startups and takedowns be run in a numeric order that’s maintained by the administrator.
* systemd not only manages processes in parallel, but also manages network connections (networkd), kernel log entries (journald), and logins (logind).

### System in Detail
* systemd is not a single daemon but a collection of programs, daemons, libraries, technologies, and kernel components.
* controlled and supervised by systemd, a resource management slice, a group of externally created processes, or a wormhole into an alternate universe.

* the WantedBy clause says that if the system has a multi-user.target unit, that unit should depend on this one (rsync.service) when this unit is enabled.
* targets have no real superpowers beyond the dependency management that’s available to any other unit.
* isolate subcommand is so-named because it activates the stated target and its dependencies but deactivates all other units.

* Linux software packages generally come with their own unit files.
* systemd knows which network ports or IPC connection points a given service will be hosting, and it can listen for requests on those channels without actually starting the service.
* units do not acquire serialization dependencies unless they explicitly ask for them.
* This service is of type forking, which means that the startup command is expected to terminate even though the actual daemon continues running in the background.
* Termination for this service is handled through KillMode and KillSignal, which tell systemd that the service daemon interprets a QUIT signal as an instruction to clean up and exit.
* /usr/lib/systemd/system/nginx.service because that file will be replaced whenever the NGINX package is updated or refreshed.

* You can configure journald to retain messages from prior boots.

### Reboot and Shutdown Procedures
* it’s a good idea to reboot after modifying a startup script or making significant configuration changes
* Shutting down physical systems: `halt -p`, `reboot`, `shutdown`

### Stratagems for a nonbooting system
* In single-user mode, also known as rescue.target on systems that use systemd, only a minimal set of processes, daemons, and services are started. 
* The root filesystem is mounted (as is /usr, in most cases), but the network remains uninitialized.
* At boot time, you request single-user mode by passing an argument to the kernel, usually single or -s.  

* Under Linux, you can run fdisk -l (lowercase L option) to see a list of the local system’s disk partitions.
* If /etc is part of the root filesystem (the usual case), it will be impossible to edit many important configuration files. To fix this problem, you’ll have to begin your single-user session by remounting / in read/write mode.
* you can boot to emergency mode by adding systemd.unit=emergency.target to the kernel arguments from within the boot loader.
* The fsck command is run during a normal boot to check and repair filesystems.

* you’re probably doing something wrong if your cloud servers require boot-time debugging.
* Within AWS, single-user and emergency modes are unavailable. However, EC2 filesystems can be attached to other virtual servers if they’re backed by Elastic Block Storage (EBS) devices.
* Boot issues within a Google Compute Engine instance should first be investigated by examination of the instance’s serial port information.
* boot a new instance that mounts the disk as an add-on filesystem. You can then run filesystem checks, modify boot parameters, and select a new kernel if necessary.

## Access Control and Rootly Powers
### Standard Unix Access Control
* Although the owner of a file is always a single person, many people can be group owners of the file, as long as they are all part of a single group.
* user identification numbers (UIDs for short) are mapped to usernames in the /etc/passwd file, and group identification numbers (GIDs) are mapped to group names in /etc/group.
* The owner of a process can send the process signals (see this page) and can also reduce (degrade) the process’s scheduling priority. 
* The defining characteristic of the root account is its UID of 0.

* When the kernel runs an executable file that has its “setuid” or “setgid” permission bits set, it changes the effective UID or GID of the resulting process to the UID or GID of the file containing the program image rather than the UID and GID of the user that ran the command.
* You can disable setuid and setgid execution on individual filesystems by specifying the nosuid option to mount. 
* It’s a good idea to use this option on filesystems that contain users’ home directories or that are mounted from less trustworthy administrative domains.

### Management of the Root account
* root logins leave no record of what operations were performed as root. 
* Another disadvantage is that the log-in-as-root scenario leaves no record of who was actually doing the work.
* most systems allow root logins to be disabled on terminals, through window systems, and across the network—everywhere but on the system console. 
* su doesn’t record the commands executed as root, but it does create a log entry that states who became root and when.
* The - (dash) option makes su spawn the shell in login mode.
* On most systems, you must be a member of the group “wheel” to use su.

* sudo takes as its argument a command line to be executed as root (or as another restricted user).
* sudo keeps a log of the command lines that were executed, the hosts on which they were run, the people who ran them, the directories from which they were run, and the times at which they were invoked.
* the dump commands can be run only as operator, not as root.
* To manually modify the sudoers file, use the visudo command, which checks to be sure no one else is editing the file, invokes an editor on it.

* Use scp to copy the current sudoers file from a known central repository, then validate it with visudo -c -f newsudoers before installation to verify that the format is acceptable to the local sudo. 

### Extensions to the standard access control model
* ACLs are part of the filesystem implementation, so they have to be explicitly supported by whatever filesystem you are using. 
* Linux can segregate processes into hierarchical partitions (“namespaces”) from which they see only a subset of the system’s files, network ports, and processes. 

### Modern Access Control
* Linux Security Modules API, a kernel-level interface that allows access control systems to integrate themselves as loadable kernel modules.


## Process Control
### Component of a Process
* A process consists of an address space and a set of data structures within the kernel. 
* The process’s virtual address space is laid out randomly in physical memory and tracked by the kernel’s page tables.
* Each thread has its own stack and CPU context but operates within the address space of its enclosing process.
* A process’s threads can run simultaneously on different cores. 
* namespaces,” which further restrict processes’ ability to see and affect each other.
* When a process is cloned, the original process is referred to as the parent, and the copy is called the child.  The PPID attribute of a process is the PID of the parent from which it was cloned, at least initially.
* A process’s UID is the user identification number of the person who created it, or more accurately, it is a copy of the UID value of the parent process.
* The EUID is the “effective” user ID, an extra UID that determines what resources and files a process has permission to access at any given moment.
* it’s useful to maintain a distinction between identity and permission, and because a setuid program might not wish to operate with expanded permissions all the time.
* Depending on how the filesystem permissions have been set, new files might default to adopting the GID of the creating process.


### the Lifecycle of a Process
* fork creates a copy of the original process, and that copy is largely identical to the parent. 
* Linux systems use clone, a superset of fork that handles threads and includes additional features.
* After a fork, the child process often uses one of the exec family of routines to begin the execution of a new program.
* When a process completes, it calls a routine named _exit to notify the kernel that it is ready to die.
* Before a dead process can be allowed to disappear completely, the kernel requires that its death be acknowledged by the process’s parent, which the parent does with a call to wait. 
* The kernel adjusts the orphan processes to make them children of init or systemd, which politely performs the wait needed to get rid of them when they die.

* Signals are process-level interrupt requests.
* The signals named KILL and STOP cannot be caught, blocked, or ignored. The KILL signal destroys the receiving process, and STOP suspends its execution until a CONT signal is received. CONT can be caught or ignored, but not blocked.
* KILL is unblockable and terminates a process at the kernel level. A process can never actually receive or handle this signal.
* pkill command searches for processes by name (or other attributes, such as EUID) and sends the specified signal
*  you might occasionally see “zombie” processes that have finished execution but that have not yet had their status collected by their parent process (or by init or systemd). If you see zombies hanging around, check their PPIDs with ps to find out where they’re coming from.

### ps: monitor processes
* useful overview of all the processes running on the system with ps aux.
* ps lax includes fields such as the parent process ID (PPID), niceness (NI), and the type of resource on which the process is waiting.

### Interactive monitoring with Top
* top is a sort of real-time version of ps that gives a regularly updated, interactive summary of processes and their resource usage.

### NICE and RENICE: influence scheduling priority
* The “niceness” of a process is a numeric hint to the kernel about how the process should be treated in relation to other processes contending for the CPU.
* The range of allowable niceness values varies among systems. In Linux the range is -20 to +19.
* process’s niceness has no effect on the kernel’s management of its memory or I/O; high-nice processes can still monopolize a disproportionate share of these resources.

### /proc filesystem
* The Linux versions of ps and top read their process status information from the /proc directory, a pseudo-filesystem in which the kernel exposes a variety of interesting information about the system’s state.
* The fd subdirectory represents open files in the form of symbolic links.
* The maps file can be useful for determining what libraries a program is linked to or depends on.

### strace and truss: trace signals and system calls
* Not only does strace show you the name of every system call made by the process, but it also decodes the arguments and shows the result code that the kernel returns.

### runaway processes
* Runaway” processes are those that soak up significantly more of the system’s CPU, disk, or network resources than their usual role or behavior would lead you to expect.
* Under Linux, check total CPU utilization with top or ps to determine whether high load averages relate to CPU load or to I/O. If CPU utilization is near 100%, that is probably the bottleneck.
* The first thing to do in this situation is to determine which filesystem is full and which file is filling it up. The df -h command shows filesystem disk use in human-readable units. Look for a filesystem that’s 100% or more full. (Most filesystem implementations reserve ~5% of the storage space for “breathing room,” but processes running as root can encroach on this space, resulting in a reported usage that is greater than 100%.)
* If you can’t determine which process is using a file, try running the fuser and lsof commands (covered in detail here) to get more information.

### Periodic processes
* cron tries to minimize the time it spends reparsing configuration files and making time calculations. The crontab command helps maintain cron’s efficiency by notifying cron when crontab files change. 
* crontab filename installs filename as your crontab, replacing any previous version. crontab -e checks out a copy of your crontab, invokes your editor on it (as specified by the EDITOR environment variable), and then resubmits it to the crontab directory.
* In general, /etc/crontab is a file for system administrators to maintain by hand, whereas /etc/cron.d is a sort of depot into which software packages can install any crontab entries they might need. 
* When a program crashes, the kernel may write out a file (usually named core.pid, core, or program.core) that contains an image of the program’s address space. 

* Web sites and software repositories are often mirrored to offer better redundancy and to offer faster access for users that are physically distant from the primary site. Use periodic execution of the rsync command to maintain mirrors and keep them up to date.

## Filesystem
* The basic purpose of a filesystem is to represent and organize the system’s storage resources.
* The filesystem can be thought of as comprising four main components:	1) A namespace – a way to name things and organize them in a hierarchy	2) An API – a set of system calls for navigating and manipulating objects	3) Security models – schemes for protecting, hiding, and sharing things		4) An implementation – software to tie the logical model to the hardware

### Pathname
* The filesystem is presented as a single unified hierarchy that starts at the directory / and continues downward through an arbitrary number of subdirectories. 

### Filesystem mounting and unmounting
* Some filesystems live on disk partitions or on logical volumes backed by physical disks, but as mentioned earlier, filesystems can be anything that obeys the proper API: a network file server, a kernel component, a memory-based disk emulator, etc. 
* The /etc/fstab file lists filesystems that are normally mounted on the system. 
* The information in this file allows filesystems to be automatically checked (with fsck) and mounted (with mount) at boot time, with options you specify.

* you can avoid the need to launder PIDs through ps by running fuser with the -v flag. This option produces a more readable display that includes the command name.

### Organization of the file tree
* The file that contains the OS kernel usually lives under /boot, but its exact name and location can vary.
* /etc for critical system and configuration files, /sbin and /bin for important utilities, and sometimes /tmp for temporary files. The /dev directory was traditionally part of the root filesystem, but these days it’s a virtual filesystem that’s mounted separately. 
* Some systems keep shared library files and a few other oddments, such as the C preprocessor, in the /lib or /lib64 directory.
* /usr is where most standard-but-not-system-critical programs are kept, along with various other booty such as on-line manuals and most libraries.

### file types
* Device drivers present a standard communication interface that looks like a regular file. When the filesystem is given a request that refers to a character or block device file, it simply passes the request to the appropriate device driver. 
* Sockets are connections between processes that allow them to communicate hygienically. 

* Like local domain sockets, named pipes allow communication between two processes running on the same host.

* A symbolic or “soft” link points to a file by name. 
* The difference between hard links and symbolic links is that a hard link is a direct reference, whereas a symbolic link is a reference by name.

### file attributes
* every file has a set of nine permission bits that control who can read, write, and execute the contents of the file.
* the file’s owner and the superuser can modify the twelve mode bits with the chmod (change mode) command. Use ls -l (or ls -ld for a directory) to inspect the values of these bits.

* The bits with octal values 4000 and 2000 are the setuid and setgid bits.
* you will be concerned mostly with the link count, owner, group, mode, size, last access time, last modification time, and type.
* The chmod command changes the permissions on a file. Only the owner of the file and the superuser can change a file’s permissions.
* With the -R option, chmod recursively updates the file permissions within a directory.
* The chown command changes a file’s ownership, and the chgrp command changes its group ownership.
* umask to influence the default permissions given to the files you create
* lsattr and chattr to view and change file attributes

### Access Control List
* Access control lists, aka ACLs, are a more powerful but also more complicated way of regulating access to files.
* Systems have largely converged on a common framing for POSIX ACLs and a common command set, getfacl and setfacl, for manipulating them.
* Given this constraint, it’s perhaps not surprising that NFSv4 ACLs are essentially a union of all preexisting systems.
* ACL support is both OS-dependent and filesystem-dependent. 

## Software Installation and Management
### OS installation
* You can set up completely hands-free installations through PXE, the Preboot eXecution Environment.
* Packaging systems define a dependency model that allows package maintainers to ensure that the libraries and support infrastructure on which their applications depend are properly installed.

### Linux Package management systems
* APT and yum, which build on these low-level facilities. 
* rpm -qa lists all the packages installed on the system.

### High-level Linux package management systems
* apt update refreshes apt’s package information cache, but yum update updates every package on the system (it’s analogous to apt upgrade).

## Scripting and the Shell
* Return a meaningful exit code: zero for success and nonzero for failure. You needn’t necessarily give every failure mode a unique exit code, however; consider what callers will actually want to know.
* A < symbol connects the command’s STDIN to the contents of an existing file. The > and >> symbols redirect STDOUT; > replaces the file’s existing contents, and >> appends to them.
* Use `export varname` to promote a shell variable to an environment variable.
* The device /dev/tty is a synonym for the current terminal window.

## User Management
### Account Mechanics
* These layers of abstraction (which are often configured in the nsswitch.conf file) enable higher-level processes to function without direct knowledge of the underlying account management method in use

### /etc/passwd file
* /etc/passwd is a list of users recognized by the system. It can be extended or replaced by one or more directory services, so it’s complete and authoritative only on stand-alone systems.
* you might see special entries in the passwd file that begin with + or -. These entries tell the system how to integrate the directory service’s data with the contents of the passwd file. This integration can also be set up in the /etc/nsswitch.conf file.
* significant weaknesses have been discovered in MD5, salted SHA-512-based password hashes have become the current standard.
* chfn command lets users change their own GECOS information. chfn is useful for keeping things like phone numbers up to date, but it can be misused.

### /etc/shadow file
* On Linux, the shadow password file is readable only by the superuser and serves to keep encrypted passwords safe from prying eyes and password cracking programs. It also includes some additional account information that wasn’t provided for in the original /etc/passwd format. These days, shadow passwords are the default on all systems.

### /etc/group file
* The group password can be set with gpasswd, which under Linux stores the encrypted password in the /etc/gshadow file.
* You can also limit access to newly created files and directories by setting your user’s default umask in a default startup file such as /etc/profile or /etc/bashrc

### Manual steps for adding users
* useradd and adduser create new users’ home directories for you, but you’ll likely want to double-check the permissions and startup files for new accounts.
* you include default startup files for each shell that is popular on your systems so that users continue to have a reasonable default environment even if they change shells.
* Make sure that the default shell files you give to new users set a reasonable default value for umask; we suggest 077, 027, or 022, depending on the friendliness and size of your site.
* By convention, Linux also keeps fragments of startup files in the /etc/profile.d directory. Although the directory name derives from sh conventions, /etc/profile.d can actually include fragments for several different shells.
* Role-based access control (RBAC) allows system privileges to be tailored for individual users and is available on many of our example systems.
* To verify that a new account has been properly configured, first log out, then log in as the new user and execute the following commands:

###  Safe removal of a user’s account and files
* When a user leaves your organization, that user’s login account and files must be removed from the system. If possible, don’t do that chore by hand; instead, let userdel or rmuser handle it.
* To find the paths of orphaned files, you can use the find command with the -nouser argument. 
  


## Logging
![](&&&SFLOCALFILEPATH&&&AAD9BF91-0B06-435C-84CC-F8CA464096BF.png)

* Syslog sorts messages and saves them to files or forwards them to another host over the network.

### Log locations
* wtmp (sometimes wtmpx) contains a record of users’ logins and logouts as well as entries that record when the system was rebooted or shut down.
*  Use the last command to decode the information. 			
* lastlog contains information similar to that in wtmp, but it records only the time of last login for each user.
* For Linux distributions running systemd, the quickest and easiest way to view logs is to use the journalctl command, which prints messages from the systemd journal. You can view all messages in the journal, or pass the -u flag to view the logs for a specific service unit. You can also filter on other constraints such as time window, process ID, or even the path to a specific executable.

### Systemd journal
* the systemd journal stores messages in a binary format. All message attributes are indexed automatically, which makes the log easier and faster to search.
* The /dev/log socket, to harvest messages from software that submits messages according to syslog conventions
* The device file /dev/kmsg, to collect messages from the Linux kernel. 
* /run/systemd/journal/socket, to service software that submits messages through the systemd journal API
* Configuring: add your customized configurations to the /etc/systemd/journald.conf.d directory. Any files placed there with a .conf extension are automatically incorporated into the configuration. 
* Unfortunately, the journal is missing many of the features that are available in syslog. 
* rsyslog can receive messages from a variety of input plug-ins and forward them to a diverse set of outputs according to filters and rules, none of which is possible when the systemd journal is used.
* systemd-journald takes over responsibility for collecting log messages from /dev/log, the logging socket that was historically controlled by syslog. 
* For syslog to get in on the logging action, it must now access the message stream through systemd. Syslog can retrieve log messages from the journal in two ways:
The systemd journal can forward messages to another socket (typically /run/systemd/journal/syslog), from which the syslog daemon can read them.
syslog can consume messages directly from the journal API, in the same manner as the journalctl command. This method requires explicit support for cooperation on the part of syslogd, but it’s a more complete form of integration that preserves the metadata for each message.

### Syslog
* Syslog has two important functions: to liberate programmers from the tedious mechanics of writing log files, and to give administrators control of logging.
* You can read plaintext messages from syslog with normal UNIX and Linux text processing tools such as grep, less, cat, and awk. 

### Kernel & Boot-time logging
* systemd-journald reads kernel messages from the kernel buffer by reading the device file /dev/kmsg. You can view these messages by running journalctl -k or its alias, journalctl --dmesg. 
* logrotate is normally run out of cron once a day. Its standard configuration file is /etc/logrotate.conf, but multiple configuration files (or directories containing configuration files) can appear on logrotate’s command line.

### Management of Logs at scale
* Elasticsearch is a scalable database and search engine with a RESTful API for querying data.
* Logstash accepts data from many sources, including queueing systems such as RabbitMQ and AWS SQS.
* Logstash can parse messages to add additional structured fields and can filter out unwanted or nonconformant data.
* Kibana can create graphs and visualizations that help to generate new insights about your applications.

## Drivers and the Kernel
### Kernel chores for system administrators
* Many of the kernel’s behaviors (such as network-packet forwarding) are controlled or influenced by tuning parameters that are accessible from user space. Setting these values appropriately for your environment and workload is a common administrative task.

### Kernel Version Numbering
You can check with `uname -r` to see what kernel a given system is running.

### Device and their drivers
The devd daemon runs in the background, watching for kernel events related to devices and acting on the rules defined in /etc/devd.conf.

### Linux Kernel Configuration
* You can view and set kernel options at run time through special files in /proc/sys. These files mimic standard Linux files, but they are really back doors into the kernel.
* A more permanent way to modify these same parameters is to use the sysctl command. sysctl can set individual variables either from the command line or from a list of variable=value pairs in a configuration file. By default, /etc/sysctl.conf is read at boot time and its contents are used to set the initial values of parameters.

### Loadable Kernel Modules
* Loadable kernel modules are conventionally stored under /lib/modules/version, where version is the version of your Linux kernel as returned by `uname -r`.
* You can inspect the currently loaded modules with the `lsmod`
* `modprobe `is a semi-automatic wrapper around a more primitive command, insmod. 
* modprobe understands dependencies, options, and installation and removal procedures. It also checks the version number of the running kernel and selects an appropriate version of the module from within /lib/modules. It consults the file /etc/modprobe.conf to figure out how to handle each individual module.

### Booting Alternate Kernels in the cloud
* Booting an alternate kernel on a cloud instance usually requires that you interact with the cloud provider’s web console or API.
* Google Cloud Platform (GCP) is the most flexible platform when it comes to boot management. Google lets you upload complete system disk images to your Compute Engine account. Note that in order for a GCP image to boot properly, it must use the MBR partitioning scheme and include an appropriate (installed) boot loader. UEFI and GPT do not apply here!

### Kernel Errors
* Bad commands entered by privileged users can certainly crash the system, but a more common cause is faulty hardware.
* A soft lockup occurs when the system is in kernel mode for more than a few seconds, thus preventing user-level tasks from running.
* Hard lockups are overtly pathological conditions that are detected relatively quickly, whereas soft lockups can occur even on correctly configured systems that are experiencing some kind of extreme condition, such as a high CPU load.


## TCP/IP Networking
* TCP/IP (Transmission Control Protocol/Internet Protocol) is the networking system that underlies the Internet. TCP/IP does not depend on any particular hardware or operating system, so devices that speak TCP/IP can all exchange data (“interoperate”) despite their many differences.

### Networking Basics
* TCP/IP is a protocol “suite,” a set of network protocols designed to work smoothly together.
* IP, the Internet Protocol, which routes data packets from one machine to another (RFC791)
* ICMP, the Internet Control Message Protocol, which defines several kinds of low-level support for IP, including error messages, routing assistance, and debugging help (RFC792)
* ARP, the Address Resolution Protocol, which translates IP addresses to hardware addresses
* UDP, the User Datagram Protocol, which implements unverified, one-way data delivery (RFC768)
* TCP, the Transmission Control Protocol, which implements reliable, full duplex, flow-controlled, error-corrected conversations

#### IPv4 and IPv6
* Network Address Translation (NAT; see this page) lets entire networks of machines hide behind a single IPv4 address. Classless Inter-Domain Routing (CIDR; see this page) flexibly subdivides networks and promotes efficient backbone routing.
* IPv4 support remains mandatory for a device to be a functional citizen of the Internet.

#### Packets and Encapsulation
* TCP/IP supports a variety of physical networks and transport systems, including Ethernet, token ring, MPLS (Multiprotocol Label Switching), wireless Ethernet, and serial-line-based systems.
* Each packet consists of a header and a payload. The header tells where the packet came from and where it’s going. It can also include checksums, protocol-specific information, or other handling instructions. The payload is the data to be transferred.
* As a packet travels down the protocol stack (from TCP or UDP transport to IP to Ethernet to the physical wire) in preparation for being sent, each protocol adds its own header information. Each protocol’s finished packet becomes the payload part of the packet generated by the next protocol. This nesting is known as encapsulation. On the receiving machine, the encapsulation is reversed as the packet travels back up the protocol stack.

#### Ethernet Framing
* One of the main chores of the link layer is to add headers to packets and to put separators between them. The headers contain each packet’s link-layer addressing information and checksums, and the separators ensure that receivers can tell where one packet stops and the next one begins. The process of adding these extra bits is known generically as framing.
* MAC, the Media Access Control sublayer, and LLC, the Logical Link Control sublayer. The MAC sublayer deals with the media and transmits packets onto the wire. The LLC sublayer handles the framing.

### Packet Addressing
* Ethernet hardware addresses are permanently assigned and immutable. However, many network interfaces let you override the hardware address and set one of your own choosing. This feature can be handy if you have to replace a broken machine or network card and for some reason must use the old MAC address
* Standard services such as SMTP, SSH, and HTTP associate themselves with “well known” ports defined in /etc/services.
* Multicast addressing facilitates applications such as video conferencing for which the same set of packets must be sent to all participants. The Internet Group Management Protocol (IGMP) constructs and manages sets of hosts that are treated as one multicast destination.
* Anycast addresses bring load balancing to the network layer by allowing packets to be delivered to whichever of several destinations is closest in terms of network routing. 

### IP Addresses: the gory detail
* you can now reassign part of the host portion to the network portion by specifying an explicit 4-byte (32-bit) “subnet mask” or “netmask” in which the 1s correspond to the desired network portion and the 0s correspond to the host portion.
* To allow hosts that use these private addresses to talk to the Internet, the site’s border router runs a system called NAT (Network Address Translation). NAT intercepts packets addressed with these internal addresses and rewrites their source addresses, using a valid external IP address and perhaps a different source port number.
* One issue raised by NAT is that an arbitrary host on the Internet cannot initiate connections to your site’s internal machines. To get around this limitation, NAT implementations let you preconfigure externally visible “tunnels” that connect to specific internal hosts and ports. 
* Another NAT-related issue is that some applications embed IP addresses in the data portion of packets; these applications are foiled or confused by NAT. 

### Routing
* You can examine a machine’s routing table with ip route show on Linux.
* Use `netstat -rn` to avoid DNS lookups and present all information numerically, which is generally more useful. 
* Dynamic routing is implemented by a daemon process that maintains and modifies the routing table. 
* the router can notify the sender of its problem by sending an ICMP redirect packet.
* redirects contain no authentication information and are therefore untrustworthy.

### IPv4 ARP and IPv6 Neighbor discovery
* If host A wants to send a packet to host B on the same Ethernet, it uses ARP or ND to discover B’s hardware address. If B is not on the same network as A, host A uses the routing system to determine the next-hop router along the route to B and then uses ARP or ND to find that router’s hardware address.
* Every machine maintains a table in memory called the ARP or ND cache which contains the results of recent queries.

### DHCP: the Dynamic Host Configuration Protocol
* When you plug a device or computer into a network, it usually obtains an IP address for itself on the local network, sets up an appropriate default route, and connects itself to a local DNS server.
* if you need to run a DHCP server, we recommend the ISC package over vendor-specific implementations.
* DHCP is a backward-compatible extension of BOOTP, a protocol originally devised to help diskless workstations boot. DHCP generalizes the parameters that can be supplied and adds the concept of a lease period for assigned values.
* Linux system that has IP forwarding enabled can act as a router. That is, it can accept third party packets on one network interface, match them to a gateway or destination host on another interface, and retransmit the packets.
* ICMP redirects (see this page) can maliciously reroute traffic and tamper with your routing tables. Most operating systems listen to ICMP redirects and follow their instructions by default. 
* Ping packets addressed to a network’s broadcast address (instead of to a particular host address) are typically delivered to every host on the network.
* The source address on an IP packet is normally filled in by the kernel’s TCP/IP implementation and is the IP address of the host from which the packet was sent. However, if the software creating the packet uses a raw socket, it can fill in any source address it likes. This is called IP spoofing and is usually associated with some kind of malicious network behavior.
* Deny IP spoofing at your border router by blocking outgoing packets whose source address is not within your address space.
* The TLS-based VPN solutions seem to be the marketplace winners at this point. They are just as secure as IPsec and considerably less complicated.

### Basic Network Configuration
* you’ll find it helpful to test your connectivity with basic tools such as `ping` and `traceroute`.
* Administrators have various heartfelt theories about how the mapping from hostnames to IP addresses is best maintained: through the hosts file, LDAP, the DNS system, or perhaps some combination of those options. The conflicting goals are scalability, consistency, and maintainability versus a system that is flexible enough to allow machines to boot and function when not all services are available.
* The /etc/hosts file is the oldest and simplest way to map names to IP addresses.
* /etc/hosts contains only local mappings and must be maintained on each client system, it’s best reserved for mappings that are needed at boot time
* you can see all the network interfaces with ip link show.
* To inspect existing routes, use the command `netstat -nr`, or `netstat -r` if you want to see names instead of numbers.
* Use `ip route del `(Linux) or route del (FreeBSD) to remove entries from the routing table.
* Run `ip route flush` (Linux) or route flush (FreeBSD) to initialize the routing table and start over.
* To configure a machine as a DNS client, you need only set up the /etc/resolv.conf file.

### Linux Networking
* The networking variables are under /proc/sys/net/ipv4.
* sysctl command to achieve the same configuration:
`sysctl net.ipv4.icmp_echo_ignore_broadcasts=1`

### Network Troubleshooting
* Make one change at a time. Test each change to make sure that it had the effect you intended. Back out any changes that have an undesired effect.
* Document the situation as it was before you got involved, and document every change you make along the way.
* Start at one end of a system or network and work through the system’s critical components until you reach the problem. For example, you might start by looking at the network configuration on a client, work your way up to the physical connections, investigate the network hardware, and finally, check the server’s physical connections and software configuration.
* Or, use the layers of the network to negotiate the problem. Start at the “top” or “bottom” and work your way through the protocol stack.

* Do you have physical connectivity and a link light?
* Is your interface configured properly?
* Do your ARP tables show other hosts?
* Is there a firewall on your local machine?
* Is there a firewall anywhere between you and the destination?
* If firewalls are involved, do they pass ICMP ping packets and responses?
* Can you ping the localhost address (127.0.0.1)?
* Can you ping other local hosts by IP address?
* Is DNS working properly?
* Can you ping other local hosts by hostname?
* Can you ping hosts on another network?
* Do high-level services such as web and SSH servers work?
* Did you really check the firewalls?
* If a machine hangs at boot time, boots very slowly, or hangs on inbound SSH connections, DNS should be a prime suspect.

* To track down the cause of disappearing packets, first run traceroute (see the next section) to discover the route that packets are taking to the target host. 
* Then ping the intermediate gateways in sequence to discover which link is dropping packets. 
* To pin down the problem, you need to send a fair number of packets. The fault generally lies on the link between the last gateway you can ping without loss of packets and the gateway beyond that.

#### traceroute
* `traceroute`, originally written by Van Jacobson, uncovers the sequence of gateways through which an IP packet travels to reach its destination.
* traceroute works by setting the time-to-live field (TTL, actually “hop count to live”) of an outbound packet to an artificially low number. As packets arrive at a gateway, their TTL is decreased. 
* This process continues until the TTL is equal to the number of hops to the destination host and the packets reach their destination successfully.
* If DNS is broken on the host you are tracing from, use traceroute -n to request numeric output. This option disables hostname lookups; it may be the only way to get traceroute to function on a crippled network.
* traceroute needs root privileges to operate.

#### Packet sniffers
* tcpdump and Wireshark belong to a class of tools known as packet sniffers. They listen to network traffic and record or print packets that meet criteria of your choice. 
* Packet sniffers understand many of the packet formats used by standard network services, and they can print these packets in human-readable form. 

### Network Monitoring
* iPerf opens a connection (TCP or UDP) between two servers, passes data between them, and records how long the process took.
* Cacti can record and graph any SNMP variable (see this page), as well as many other performance metrics. 
* We do not recommend the use of Linux, UNIX, or Windows systems as firewalls because of the insecurity inherent in running a full-fledged, general-purpose operating system.

### Cloud Networking
* On the Google Cloud Platform, networking is functionally part of the platform, as opposed to being represented as a distinct service. GCP private networks are global: an instance in the us-east1 region can communicate with another instance in europe-west1 over the private network, a fact that makes it easy to build global network services. 
* There is no concept of a subnetwork being public or private; instead, instances that don’t need to accept inbound traffic from the Internet can simply not have a public, Internet-facing address.


## Physical Networking
The more logic that a device uses to move bits from one network to another, the more hardware and embedded software the device needs to have and the more expensive it is likely to be.
* A hub retimes and retransmits Ethernet frames but does not interpret them; it has no idea where packets are going or what protocol they are using. With the exception of extremely special cases, hubs should no longer be used in enterprise networks; we discourage their use in residential (consumer) networks as well. Switches make significantly more efficient use of network bandwidth and are just as cheap these days.
* Switches receive, regenerate, and retransmit packets in hardware. Switches use a dynamic learning algorithm. They notice which source addresses come from one port and which from another. They forward packets between ports only when necessary. At first all packets are forwarded, but in a few seconds the switch has learned the locations of most hosts and can be more selective.
* Routers (aka “layer 3 switches”) direct traffic at the network layer, layer 3 of the OSI network model. They shuttle packets to their final destinations in accordance with the information in the TCP/IP protocol headers.


* The main idea of SDN is that the components managing the network (the control plane) are physically separate from the components that forward packets (the data plane).
* SDN and its API-driven configuration system offer you, the sysadmin, a tempting opportunity to integrate network topology management with other DevOps-style tools for continuous integration and deployment. 

## IP Routing

## DNS

## SSO

## Electronic Mail

## Web Hosting

## Storage

## the Network File System

## SMB

## Configuration Management

## Virtualization

## Security

## Monitoring

