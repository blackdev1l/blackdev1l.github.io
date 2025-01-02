+++
date = "2015-03-08T15:53:51"
draft = "false"
title = "Torrent-box with Raspberry Pi"
slug = "torrent-box-with-raspberrypi"

+++

Hello folks! Today english post (my first one) , we are going to build a simple torrent-box with the raspberry pi, an old hard drive and a little bit of  GNU/Linux magic! 

## Boot up and update raspbian

First off, make sure to have a fresh clean install of raspbian, just in case something goes wrong, you can debug these steps and see what's wrong, instead of remember months of tweaking and hacking on an old installation.
update & upgrade raspberry  raspbian with `sudo apt-get update` and `sudo apt-get upgrade` , after that, let's install some programs we will need later. 
`sudo apt-get install samba samba-common-bin btrfs-tools avahi-daemon python-dev python-setuptools transmission-daemon`

## SSH all the way

put a fancy [hostname](http://en.wikipedia.org/wiki/Hostname) on your raspberry, the default one is `raspberrypi`, if you want to change it just give `sudo hostname newhostname` , so you can identify your raspberry with a different and original hostname, i prefer to use the default one because i just have one raspberry, from now on raspberrypi will be the hostname on this tutorial.  
With `avahi-daemon` you can forget about ip and such, because you can now ssh using your hostname on domain .local, for more info about avahi-daemon [read here](http://linux.die.net/man/8/avahi-daemon) . 
Just in case, write down your local ip, you can find it with `hostname -I` , So if you can't log in with the hostname.local you can always log in with the ip. 
Now you can unplug your keyboard and monitor and start to use it via ssh. 

## Headless Pi

* Windows  
	on Windows, make sure to have bonjour services installed, if you have Itunes, you should be ready to use .local domains inside your network, open your command line and ping your raspberry with `ping raspberrypi.local` where raspberrypi is the hostname, if it pings back then you can ssh, if not, install [this](https://support.apple.com/kb/DL999?locale=en_US) (it's for printers but indeed it install bonjour services to use with every application) .
    For ssh, i think one of the best option is to use the great [putty](http://www.chiark.greenend.org.uk/~sgtatham/putty/) , In hostname write your "raspberrypi.local" and leave the default port.  
* GNU/Linux  
One of the great things about GNU/Linux is that most of the tools are already installed, so you can just ssh from terminal with `ssh pi@raspberrypi.local` where pi is the default user on Raspbian and raspberrypi.local the hostname name inside the local domain given by the avahi-daemon.

## Use btrfs as filesystem

I decided to use btrfs as filesystem because i like how it handles raid systems and how flexibile it is with the partions. Use it is as simple as any other filesystem, plus, you don't have to worry about the first partition first because it handles all  itself.
assuming you have your hard drive plugged in via usb and it is mapped as `/dev/sda/` (make sure with lslblk) , you can format it with ` sudo mkfs.btrfs -m single /dev/sda1` , it is now ready to use.
## Auto mount Hard Drive at startup
open your `/etc/fstab` with `sudo nano /etc/fstab` and add this line:
`/dev/sda1       /home/pi/share  btrfs   defaults,nossd  0       1`  
With this you will automount your hard drive everytime you plug it and on startup, with the mount position on `/home/pi/share` , if you want, you can change it (usually it is in the /media/ folder) , make sure to create the folder before to mount, so we are going to `mkdir /home/pi/share` and then `mount -a` , give at your user the write/read permission to that folder with `sudo chown -R pi:pi /home/pi/share/`  and you should be all set for the next step. 


## Samba

Samba is a free software that gives share services between devices, it is often used with printers and it is compatible with different OS, so you can see and write/read from a windwos machine to a GNU/Linux server. 
let's start making a copy of the default smb.conf, so if we fuck up the config, we have the default config saved.
`cd /etc/samba` then ` sudo cp smb.conf smb.conf.original` , now let's modify smb.conf with `sudo nano smb.conf` !   
change `#Security` with `security` under **authentication** section.
seach for `wins support` and change it from no to yes.
search for `workgroup` and change it (if you want) with your workgroup name (it can be anything you want).
now, save and exit from file, and restart samba services with `sudo /etc/init.d/samba restart`.
create your samba password for you user with `sudo smbpasswd -a <username>` (you must put your username, if pi is your user then put pi) .
open again your smb.conf with `sudo nano smb.conf` and create your first samba shared folder. 
paste this at the end of the file
```
[USBShare]
comment = USB share
path = /home/pi/share
writeable = yes
only guest = no
create mask = 0777
directory mask = 0777
```

you can change `USBShare` with the name of the samba folder you want to put, and comment too, path is the location of the mounted folder.
give read/write permission too with `sudo chmod 0755 /home/pi/share` and `sudo chown -R <username> /home/pi/share`

restart samba for the last time `sudo /etc/init.d/samba restart`

test it out with another computer!

## Transmission-Daemon

add the transmission-daemon user to the group of your default user: 
`sudo usermod -a -G debian-transmission pi` , now, create some folders and add the proper permission:

`mkdir ~/share/complete`  
`mkdir ~/share/incomplete`  
`sudo chmod 770 ~/share/complete`  
`sudo chmod 770 ~/share/incomplete`  
then make sure to have the configs generated by the daemon, let's start and stop it: 
`sudo service transmission-daemon start`  
`sudo service transmission-daemon stop`  
now modify transmission config: 
`sudo nano /etc/transmission-daemon/settings.json`

change the following:

`download-dir: /home/pi/share/complete`  
`incomplete-dir: /home/pi/share/incomplete`  
`incomplete-dir-enabled: true`  
`peer-port-random-on-start: true`  
`rpc-password: <password>`  
`rpc-username: <username>`  
`rpc-whitelist-enabled: false`  

now start the daemon again, with `sudo service transmission-daemon start` 

you can view your transmission daemon via webclient, just open http://raspberrypi.local:9091/transmission/web/ from your browser and log in with the credentials you given in the config! 
## Python, pip, and virtualenv

Now we are going to set up pip and virtualenv, because we are going to need them for 2 pieces of software that will do almost all the magic.
install pip with `sudo easy_install pip` , then virtualenv with `sudo pip install virtualenv`.
Don't forget to install transmission-rpc, so you can view your torrents from clients such transmission-GUI or transdroid
`sudo pip install transmissionrpc`

### What is virtualenv and why you always should use it

Virtualenv it's a tool that lets you create standalone python enviroments, what does that means? It means that if you are going to download a python software that has a lot of dependencies, and you are afraid about versioning and clean install, you can have an enviroment completely clean to use it just for your software, so you won't have unnecessary libraries and modules on your default python folder, and when you are done with your software, you can easily remove your enviroment (a simple folder) and have all the dependencies deleted from your filesystem. If you are still confuse, read [this](http://simononsoftware.com/virtualenv-tutorial/) and [this](http://iamzed.com/2009/05/07/a-primer-on-virtualenv/), i'm sure you are going to love virtualenv. 

## Flexget


ok, now that we have virtualenv, create one env for flexget with `virtualenv env/flexget` and now we can use the activate script to enter the enviroment. 
`source ./env/flexget/bin/activate` from now on, we are in the flexget enviroment, and we can safely install flexget from here with pip: `pip install flexget`

Flexget is a great piece of software that handles rss feeds, we are going to use it with the feed generated by http://showrss.info/ to have our seris downloaded automatically when they are out. Create an account on showrss, add some seris, then click on "feeds" and generate the url, copy it somewhere and now let's go back to our raspberrypi. 
create .flexget folder and a config inside. 
`mkdir ~/.flexget`  
`nano ~/.flexget/config.yml`

and write here your tasks,you can learn more about tasks [here](http://flexget.com/wiki/Cookbook).
i use this task to download every 6 hours torrents given by my showrss account, and then sorts in the TV folder. Pretty neat uh?
```lang-yaml
tasks:
  task-a:
    rss: <put here generated url>
    all_series: yes
    transmission:
      host: localhost
      port: 9091
      username: pi
      password: <transmission-password>
      path: /home/pi/share/TV/{{series_name}}/Season {{series_season}}
      addpaused: no
    seen: yes

schedules:
  - tasks: [task-a]
    interval:
      hours: 6

```
This task will look every 6 hours if there is any torrent that i haven't download on my generated feed rss from showrss. You can extend or modify this task to do complex things, sky is the limit! 

### Crontab flexget

If you want that flexget starts with the pi, add to your crontab your flexget daemon: 
`crontab -e`  
add this line at the end of file: 
`@reboot /home/pi/env/flexget/bin/flexget daemon start`  
save and close.


## Enjoy

Now you have a full torrent-box with an auto-downloader for your latest show, amazing right? You can do a lot more, i'll probably write another blog post or maybe extend this with auto-download subtitles and other amazing things, stay tuned! 

