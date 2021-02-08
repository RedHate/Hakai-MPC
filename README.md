
# Ultros 2021 Akai System Spoofing Data For MPC ONE, LIVE and X

## The basis for the Hakai 2.9.0 (VERSION 2) force enabled firmware found on mpc-forums.com

In order to have alternate software builds work on the MPC or to unlock features locked by baked in
conditionals one needs only to spoof the identify of the device.

Sadly this identify is set by the kernel, luckily the "mount -o bind" command makes this not a problem.
The trick is to make sure that your spoofed version of the product code is set to permissions 444. READ ONLY.

The issuing of the following mount bind command ****MUST**** take place before the MPC software is executed.


	mount -o bind "/root/models/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"


When the program is exited or when the software is restarted say into a different mode like from Force software to MPC
you then have to issue this following command so you dont continually mount over the same file.


	umount "/sys/firmware/devicetree/base/inmusic,product-code"


## Example snippet from ultros az01-launch-MPC script from the Hakai Version 2 CFW.
	
	if [ ! -e /tmp/force ]; then
		
		echo "Launching MPC" >> /media/acvs-synths/cfw.log
		
		## ORIGINAL MPC SOFTWARE
		touch /tmp/force
		
		#unmount the magic
		umount "/sys/firmware/devicetree/base/inmusic,product-code"
		umount "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		## mount the magic
		mount -o bind "/root/models/x/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"
		## because sometimes the screen will rotate, odd bug maybe the force uses a diff orientation.
		mount -o bind "/root/models/x/inmusic,panel-rotation" "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		## log it
		cat "/sys/firmware/devicetree/base/inmusic,panel-rotation"  >> /media/acvs-synths/cfw.log
		cat "/sys/firmware/devicetree/base/inmusic,product-code"    >> /media/acvs-synths/cfw.log
		
		if type systemd-inhibit >/dev/null 2>&1
		then
			exec systemd-inhibit --what=handle-power-key /usr/bin/MPC "$@"
		else
			ulimit -S -s 1024
			exec /usr/bin/MPC "$@"
		fi

	else

		echo "Launching FORCE" >> /media/acvs-synths/cfw.log
		
		##FOCE SOFTWARE
		rm /tmp/force
		
		#unmount the magic
		umount "/sys/firmware/devicetree/base/inmusic,product-code"
		umount "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		## mount the magic
		mount -o bind "/root/models/force/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"
		## because sometimes the screen will rotate, odd bug maybe the force uses a diff orientation.
		mount -o bind "/root/models/force/inmusic,panel-rotation" "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		##log it
		cat "/sys/firmware/devicetree/base/inmusic,panel-rotation"  >> /media/acvs-synths/cfw.log
		cat "/sys/firmware/devicetree/base/inmusic,product-code"    >> /media/acvs-synths/cfw.log
		
		export LD_PRELOAD="/usr/lib/libSOUL_PatchLoader.so /usr/lib/libcares.so.2.4.0 /usr/lib/libbluetooth.so.3.19.2" 
		
		if type systemd-inhibit >/dev/null 2>&1
		then
			exec systemd-inhibit --what=handle-power-key /usr/bin/FORCE "$@"
		else
			ulimit -S -s 1024
			exec /usr/bin/FORCE "$@"
		fi
		
	fi
	
## How to make hypesynth content work properly

Format a memory card with fat or ext4 (*grrr typos.. lol*) and name it.... *note* that it must be lower-case: 
	
	acvs-synths
	
This can be acheived on linus with this command if using fat (fat defaults to upper-case disk names, to get around it we do this)

	fatlabel /dev/sda1 acvs-synths

Otherwise gparted will suffice and regular conventional naming. Next you'll want to populate some folders:

	mkdir /media/acvs-synths/Synths
	mkdir /media/acvs-synths/Synths/Hype

The acvs-synths card must be in the unit before you power it on or it will NOT mount! (super frustrating!) I better not fail to mention this as people will want to tear their hair out if they don't figure it out on their own. "acvs-synths" is mounted by a service it seems.


## Bonus info!!

Ok, so I've done some testing with alsa and usb soundcards, damn near bricked my unit... *DO NOT MESS WITH THE DEFAULT ALSA CONFIG!!!!* it will result in a bootloop and your system being dicked unless you DFU or use a timeout script to delay the process while you sneak in and ssh.

Here are my findings, this works on my numark mixdeck express plugged in via usb. Thus meaning the kernel supports usb audio and is functional. If the MPC software tries to run with anything other than the 1st card ("ACVA" in my case) it immediately shuts down the unit.
	
	aplay -D default:Express "/media/acvs-content/Expansions/MPC One/HipHop-Stab-BDWK Stb1.WAV"
	
To see for yourself that your usb cards are listed and recognized try this:

	CARDS=$(aplay -l | awk -F \: '/,/{print $2}' | awk '{print $1}' | uniq)
	for d in $CARDS; do if [ "$CARD"!="ACVA" ]; then echo "$d"; fi; done;

Once you isolate the 2nd card's name issue the command above with a path to your own wave to send it to the usb sound card device... Might be fun to audio-brand the start up. However currently I would say that using a USB soundcard for sampling and playback within the MPC software is not a realistic desire just due to design.

I need to proof read more and commit less.

## Oh yeah... 

By the way, these are objects of interest for those of you that may have not had a look.

	#ITEMS OF INTEREST										#CONTENT OF SAID ITEMS
	/sys/firmware/devicetree/base/inmusic,product-code						ACVA
	/sys/firmware/devicetree/base/compatible							inmusic,acvainmusic,az01rockchip,rk3288
	/sys/firmware/devicetree/base/serial-number							AXXXXXXXXXXXXXXXXXX
	/sys/firmware/devicetree/base/inmusic,az01-pcb-rev						J
	/sys/firmware/devicetree/base/inmusic,panel-rotation						N/A (hex)
