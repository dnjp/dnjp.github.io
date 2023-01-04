---
title: Setup Plan 9 VPS and Drawterm on Linode
tags:
categories:
date: 2023-01-03
lastMod: 2023-01-03
---
Plan 9 is a distributed system that expands on the “everything is a file” metaphor by making almost all components available to the network. Need more CPU for a compute heavy workload? Spin up a CPU server and add it to your network. Need access to that script your friend wrote? Mount their namespace over 9P and just use the script. [The Organization of Networks in Plan 9](http://doc.cat-v.org/plan_9/4th_edition/papers/net/) by Dave Presotto and Phil Winterbottom explains this in greater detail and provides the following summary for how Plan 9 is used in practice:

  + “At work, users tend to use their terminals like workstations, running interactive programs locally and reserving the CPU servers for data or compute intensive jobs such as compiling and computing chess endgames. At home or when connected over a slow network, users tend to do most work on the CPU server to minimize traffic on the slow links. The goal of the network organization is to provide the same environment to the user wherever resources are used.”

The architecture of a Plan 9 system can range from running every component together on your local machine, to having each part distributed across a variety of machines in a much more elaborate configuration. As my first step in trying out some of these features, I’m going setup a basic Plan 9 configuration on a single VPS hosted in [Linode](https://www.linode.com/). I am going to be doing this from my Macbook Air (M1, 2020) which brings along its own set of issues to contend with.

Creating the VPS

  + The first step is to create a Linode account and navigate to the “Linodes” section on the left side panel and select “Create Linode”. Here are the settings I’d recommend for this initial test:

    + Images: Alpine 3.14 (we’ll end up removing this image, so it doesn’t really matter which you select)

    + Region: Newark, NJ (pick whichever region is closest to you)

    + Linode Plan: Shared CPU Nanode 1 GB ($5/month)

    + Linode Label: 9front

    + Root Password: (we’re going to wipe away the Alpine image, so this doesn’t matter)

    + Add-ons: Private IP

  + Once these options are selected, click “Create Linode” on the right to spin up your VPS. Once it’s up, select your new machine and click “Power Off” towards the top because we’re about to create some new disks and setup configurations for installing and running Plan 9.

  + We will be creating 2 disks for this installation - one which will contain the contents of the Plan 9 ISO (we’ll be using 9front) and the other will be our primary disk we will be booting into after the installation.

  + First, delete any disks that Linode created by clicking the “…” button followed by “Delete” next to each disk. Then, click “Add a Disk” and create the two disks in this order:

    + Installer:

      + Label: installer

      + Filesystem: raw

      + Size: 650 MB

    + Boot:

      + Label: boot

      + Filesystem: raw

      + Size: 24950 MB (the remaining space left)

  + Now we’ll create 2 configurations - one to boot the install media and the other we’ll use to boot Plan 9 after the install. Select “Configurations”, click “Add Configuration”, and create these 2 configurations:

    + Installer:

      + Label: installer

      + Virtual Machine:

        + VM Mode: Full virtualization

      + Boot Settings:

        + Select a Kernel: Direct Disk

        + Run Level: Run Default Level

        + Memory Limit: Do not set any limits on memory usage

      + Block Device Assignment:

        + /dev/sda: boot

        + /dev/sdb: installer

        + /dev/sdc: None

        + Root Device: /dev/sdb

      + Filesystem/Boot Helpers: disable all enabled options

    + Boot:

      + Label: boot

      + Virtual Machine:

        + VM Mode: Full virtualization

      + Boot Settings:

        + Select a Kernel: Direct Disk

        + Run Level: Run Default Level

        + Memory Limit: Do not set any limits on memory usage

      + Block Device Assignment:

        + /dev/sda: boot

        + /dev/sdb: None

        + Root Device: /dev/sda

      + Filesystem/Boot Helpers: disable all enabled options

  + With all of this configuration out of the way, now we will actually install Plan 9. Click the “…” icon for your machine and select “Rescue”. This will boot us into a mode where we can unpack the ISO and setup our “installer” disk. Once the machine has booted, click “Launch LISH Console” which will pull up a terminal in your web browser.

  + While this is coming online, copy the link to the [9front release](http://9front.org/iso/) you want to install. In the Linode LISH console, run the following command to write the ISO to the “installer” disk:

    + 
```plain text
wget http://9front.org/iso/9front-8392.16c5ead832f2.amd64.iso.gz -q -O - | funzip | dd of=/dev/sda

```

  + When this has completed, close the LISH Console window, and click the “Boot” button for the “installer” disk. Once the machine has finished restarting, click “Launch LISH Console” again and click the “Glish” tab in the window that opens. If all goes well, you should be in the Plan 9 terminal which will have default options for booting into the graphical environment. Hit Enter for each of these options to select the defaults. When [rio](http://man.cat-v.org/9front/1/rio) launches, it will ask you to select the mouseport where you should enter “ps2intellimouse”. In the terminal window that will open, enter inst/start to begin the installation. At this point, you can follow the [FQA](http://fqa.9front.org/fqa4.html#4.3) to complete installing Plan 9 with a few exceptions.

  + In the partdist step, you will need to delete all of the partitions on /dev/sdC0 with d <partition name> until the disk simply shows empty. Enter w and then q to write the change which will restart the partdist step with the empty partion selected. You can simply enter w and then q again to write the default Plan 9 partitions. During the mountdist step, be sure to select /dev/sdC1/data for the distribution disk.

  + When the installation is completed and the installer restarts the machine, close out of the LISH console. Now, click “Boot” for the “boot” disk to boot up Plan 9 normally. We’ll again select the default options in the terminal which will then launch rio on your new Plan 9 VPS.

Set up CPU Server

  + Obviously, interacting with Plan 9 through the LISH Console is not the most pleasant experience. We will be configuring Plan 9 to run as a standalone CPU server which we will connect to with [Drawterm](http://drawterm.9front.org/).

  + The first step is to modify the default boot settings for Plan 9:

    + 
```plain text
% 9fs 9fat
% cd /n/9fat
% cp plan9.ini plan9.ini.bak  # keep a backup
% sam plan9.ini
```

  + This will open up the [sam editor](http://man.cat-v.org/plan_9/1/sam) so we can edit the ini file. Right click in the bottom section of the editor and select “plan9.ini” and then right click again in the bottom section to open the file.

  + Right click on the desktop, click “New”, and right click drag the size of the window you want to create. In that window, we will get some networking information that we will need to modify the plan9.ini file:

    + 
```plain text
% ip/ipconfig
% cat /net/ndb
```

  + Back in sam, add the following to the end of the plan9.ini file

    + 
```plain text
nobootprompt=local!/dev/sdC0/fs -m <mem> -A -a tcp!*!564
user=glenda
auth=<your ip address>
cpu=<your ip address>
authdom=<choose name>
service=cpu
```

  + Once these changes have been made, click on the top part of the editor and enter “w” and then “q” to save the file and quit sam. Now we will setup the auth server:

    + 
```plain text
% auth/wrkey
# authid: glenda
# authdom: <same as plan9.ini>
# secstore key: <enter password>
# password: <enter same password>
% auth/keyfs
% auth/changeuser glenda
# Password: <same as used above>
# POP secret: doesn't matter
# Expiration Date: never
# Post id: glenda
# User's full name: <whatever you want>
% auth/enable glenda
```

  + Now we’ll configure the network database:

    + 
```plain text
% cd /lib/ndb
% sam ./local
```

  + Towards the end of the file there will be a line that starts with sys=somehost ether=.... Change this line to the following:

    + 
```plain text
sys=<keep value> ether=<keep value> authdom=<same as plan9.ini> auth=<your ip address> ip=<your ip>

```

  + Below the sys line, add the following settings:

    + 
```plain text
ipnet=<give a name> ip=<your ip, replace last number with 0> ipmask=<same as /net/ndb>
	ipgw=<same as /net/ndb>
	auth=<your ip>
	authdom=<same as plan9.ini>
	fs=<your ip>
	cpu=<your ip>
	dns=<same as /net/ndb>
```

  + Save the file and quit sam. Now we will modify your users profile so that when we launch drawterm that we’ll have rio launch automatically:

    + 
```plain text
$ cd $home
$ sam lib/profile
```

  + Find the cpu case in the service switch statement. Directly under the fn cpu%... line, add a new line to start rio:

    + 
```plain text
fn cpu%{ $* }
rio -i riostart -s
```

  + Save the file, quit sam, and sync the disk:

    + 
```plain text
% echo sync >> /srv/hjfs.cmd
% fshalt -r
```

  + This will restart the machine, but make sure it returns to “Running” in Linode. Click “Launch LISH Console” and select the Glish tab after the machine restarts, accept the defaults in the terminal, and that should get you to a prompt starting with cirno#. We’re now ready to set up Drawterm.

Drawterm

  + To get started, clone the [drawterm git repo](http://git.9front.org/plan9front/drawterm/HEAD/info.html) locally and change to its directory. Follow the README to install for your platform. For MacOS, this means:

    + 
```plain text
$ CONF=osx-cocoa make
$ cp drawterm gui-cocoa/drawterm.app/
$ cp -r gui-cocoa/drawterm.app /Applications/
```

  + I’ve made some small changes to the MacOS app to boot my settings by default. You can check them out [here](https://github.com/dnjp/cfg/tree/master/mac/drawterm.app). For Linux, once the build has completed, copy the drawterm binary to /usr/local/bin.

  + Now, copy the IP address for your VPS in Linode and launch drawterm:

    + 
```plain text
$ drawterm -u glenda -a <VPS IP> -h <VPS IP>
```

  + If all goes well, a graphical window should launch and prompt you to enter the password for the glenda user that you set earlier. Enter the password, and you should now have rio running through drawterm on your local machine. A quick note: your local file system is available in drawterm at /mnt/term which enables you to use Plan 9 to work with files on your Unix machine.
