
## Ultros 2021 - MPC Exploration and hacks.

# Akai System Spoofing Data For MPC ONE, LIVE and X

First of all, Thank you to TheKikGen for taking the work out of modding the rom. You've done an excellent job with the tools and documentations. For those of you that haven't first taken a look at TheKikGen's github look here this is the foundation for rom modding and source for technical data: https://github.com/TheKikGen

TheKikgen's ssh enabled firmware can be found here: https://github.com/TheKikGen/MPC-LiveXplore


# The basis for the Hakai 2.9.0 (VERSION 2) force enabled firmware found on mpc-forums.com

In order to have alternate software builds work on the MPC or to unlock features locked by baked in
conditionals one needs only to spoof the identify of the device.

Sadly this identify is set by the kernel, luckily the "mount -o bind" command makes this not a problem.
The trick is to make sure that your spoofed version of the product code is set to permissions 444. READ ONLY.

The issuing of the following mount bind command ****MUST**** take place before the MPC software is executed.


	mount -o bind "/root/models/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"


When the program is exited or when the software is restarted say into a different mode like from Force software to MPC
you then have to issue this following command so you dont continually mount over the same file.


	umount "/sys/firmware/devicetree/base/inmusic,product-code"


# Example snippet from ultros az01-launch-MPC script from the Hakai Version 2 CFW.
	
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
	
# How to make hypesynth content work properly

Format a memory card with fat or ext4 (*grrr typos.. lol*) and name it.... *note* that it must be lower-case: 
	
	acvs-synths
	
This can be acheived on linus with this command if using fat (fat defaults to upper-case disk names, to get around it we do this)

	fatlabel /dev/sda1 acvs-synths

Otherwise gparted will suffice and regular conventional naming. Next you'll want to populate some folders:

	mkdir /media/acvs-synths/Synths
	mkdir /media/acvs-synths/Synths/Hype

The acvs-synths card must be in the unit before you power it on or it will NOT mount! (super frustrating!) I better not fail to mention this as people will want to tear their hair out if they don't figure it out on their own. "acvs-synths" is mounted by a service it seems.


# Bonus info!!

Ok, so I've done some testing with alsa and usb soundcards, damn near bricked my unit... *DO NOT MESS WITH THE DEFAULT ALSA CONFIG!!!!* it will result in a bootloop and your system being dicked unless you DFU or use a timeout script to delay the process while you sneak in and ssh.

Here are my findings, this works on my numark mixdeck express plugged in via usb. Thus meaning the kernel supports usb audio and is functional. If the MPC software tries to run with anything other than the 1st card ("ACVA" in my case) it immediately shuts down the unit.
	
	aplay -D default:Express "/media/acvs-content/Expansions/MPC One/HipHop-Stab-BDWK Stb1.WAV"
	
To see for yourself that your usb cards are listed and recognized try this:

	CARDS=$(aplay -l | awk -F \: '/,/{print $2}' | awk '{print $1}' | uniq)
	for d in $CARDS; do if [ "$CARD"!="ACVA" ]; then echo "$d"; fi; done;

Once you isolate the 2nd card's name issue the command above with a path to your own wave to send it to the usb sound card device... Might be fun to audio-brand the start up. However currently I would say that using a USB soundcard for sampling and playback within the MPC software is not a realistic desire just due to design.

There does appear to be some stuff on thekikgen's repo, but if not analyzed the content yet to see if there's been progress made here. Feel free to look https://github.com/TheKikGen/MPC-LiveXplore/tree/master/conf

I need to proof read more and commit less.

# Oh yeah... 

By the way, these are objects of interest for those of you that may have not had a look.

	#ITEMS OF INTEREST										#CONTENT OF SAID ITEMS
	/sys/firmware/devicetree/base/inmusic,product-code						ACVA
	/sys/firmware/devicetree/base/compatible							inmusic,acvainmusic,az01rockchip,rk3288
	/sys/firmware/devicetree/base/serial-number							AXXXXXXXXXXXXXXXXXX
	/sys/firmware/devicetree/base/inmusic,az01-pcb-rev						J
	/sys/firmware/devicetree/base/inmusic,panel-rotation						N/A (hex)
	


# Results of libinput testing:
I WAS WRONG!!! GASP... touch, mouse and keys are handled by libinput but no midi!! (See TheKikGen, you *DO* know more! LOL)

Results of libinput device listing:

	# libinput list-devices
	Device:           ILI2117 Touchscreen
	Kernel:           /dev/input/event0
	Group:            1
	Seat:             seat0, default
	Capabilities:     touch 
	Tap-to-click:     n/a
	Tap-and-drag:     n/a
	Tap drag lock:    n/a
	Left-handed:      n/a
	Nat.scrolling:    n/a
	Middle emulation: n/a
	Calibration:      identity matrix
	Scroll methods:   none
	Click methods:    none
	Disable-w-typing: n/a
	Accel profiles:   n/a
	Rotation:         n/a

	Device:           Logitech Wireless Receiver Mouse
	Kernel:           /dev/input/event2
	Group:            2
	Seat:             seat0, default
	Capabilities:     pointer 
	Tap-to-click:     n/a
	Tap-and-drag:     n/a
	Tap drag lock:    n/a
	Left-handed:      disabled
	Nat.scrolling:    disabled
	Middle emulation: disabled
	Calibration:      n/a
	Scroll methods:   button
	Click methods:    none
	Disable-w-typing: n/a
	Accel profiles:   flat *adaptive
	Rotation:         n/a

	Device:           gpio-keys
	Kernel:           /dev/input/event1
	Group:            3
	Seat:             seat0, default
	Capabilities:     keyboard 
	Tap-to-click:     n/a
	Tap-and-drag:     n/a
	Tap drag lock:    n/a
	Left-handed:      n/a
	Nat.scrolling:    n/a
	Middle emulation: n/a
	Calibration:      n/a
	Scroll methods:   none
	Click methods:    none
	Disable-w-typing: n/a
	Accel profiles:   n/a
	Rotation:         n/a


Results of libinput debugging:

	# libinput debug-events
	-event0   DEVICE_ADDED     ILI2117 Touchscreen               seat0 default group1  cap:t ntouches 10 calib
	-event1   DEVICE_ADDED     gpio-keys                         seat0 default group2  cap:k
	-event0   TOUCH_DOWN       +0.000s	0 (0) 22.21/36.99 (455.00/758.00mm)
	 event0   TOUCH_FRAME      +0.000s	
	 event0   TOUCH_UP         +0.130s	
	 event0   TOUCH_FRAME      +0.130s	
	-event1   KEYBOARD_KEY     +1.727s	KEY_POWER (116) pressed
	 event1   KEYBOARD_KEY     +1.881s	KEY_POWER (116) released


# Results of midi testing the control interface:

List off the midi devices:

	# aconnect -o
	client 16: 'MPC One MIDI' [type=kernel,card=0]
		0 'MPC Studio Live Public'
		1 'MPC Studio Live Private'
		2 'MPC Studio Live MIDI Port'
	client 130: 'Virtual MIDI Output 1 Input' [type=user,pid=253]
		0 'in              '
	client 131: 'Virtual MIDI Output 2 Input' [type=user,pid=253]
		0 'in      

Then free up the port by deinitilaizing the inmusic-mpc.service and test with aseqdump -p 16:1

	# systemctl stop inmusic-mpc.service
	# aseqdump -p 16:1
	Waiting for data. Press Ctrl+C to end.
	
	Source  Event                  Ch  Data
	 16:1   Note on                 9, note 43, velocity 127
	 16:1   Note on                 9, note 45, velocity 56
	 16:1   Note off                9, note 45, velocity 0
	 16:1   Polyphonic aftertouch   9, note 43, value 13
	 16:1   Polyphonic aftertouch   9, note 43, value 14
	 16:1   Polyphonic aftertouch   9, note 43, value 17
	 16:1   Polyphonic aftertouch   9, note 43, value 28
	 16:1   Polyphonic aftertouch   9, note 43, value 27
	 16:1   Polyphonic aftertouch   9, note 43, value 21
	 16:1   Polyphonic aftertouch   9, note 43, value 20
	 16:1   Polyphonic aftertouch   9, note 43, value 10
	 16:1   Polyphonic aftertouch   9, note 43, value 4
	 16:1   Polyphonic aftertouch   9, note 43, value 1
	 16:1   Polyphonic aftertouch   9, note 43, value 0
	 
Capturing midi events:

	# systemctl stop inmusic-mpc.service
	# arecordmidi -p 16:1 /media/<sdcard>/midi.log

# Internal midi device channel and values

The control interface is registered on channel 9 and here is the diagram. (MPC One)

      +-------------------+
      | 49 | 55 | 51 | 53 |
      |-------------------|
      | 48 | 47 | 45 | 43 |
      |-------------------|
      | 40 | 38 | 46 | 44 |
      |-------------------|
      | 37 | 36 | 42 | 82 |
      +-------------------+

The rest of the buttons (on the mpc one, feel free to add)

	#LABLE                  #MIDI NO
	--------------------------------
	Record: 		73
	Overdub: 		80
	Stop: 			81
	play: 			82
	play start: 	        83
	Step Seq: 		41 
	TC: 			12 
	Sampler: 		125 
	Sample Edit: 	        6 
	Program Edit: 	        2
	Undo:			67
	Tempo: 			53
	Shift: 			49
	Menu: 			123
	Main: 			52 
	Browse: 		50 
	Track Mix: 		116 
	Track Mute: 	        43 
	Next Seq: 		42
	Bank A: 		35
	Bank B: 		36
	Bank C: 		37
	Bank D: 		38
	Qlink Select: 	        0
	Note Repeat: 	        11
	Erase: 			9
	Copy: 			122
	Sixteen Levels:         40
	Fuil Level: 	        39
	Knob: 			100
	Knob-Push: 		111
	Minus: 			12
	Plus: 			125
	Q-Link 1: 		19
	Q-link 2: 		18
	Q-link 3: 		17
	Q-link 3: 		16
	Q-Link 1 Touch 		87
	Q-link 2 Touch		86
	Q-link 3 Touch		85
	Q-link 3 Touch		84

# WOAH MAJOR SOUND BUG FIX?

Theres were the only 3 differeing libs between the Force firmware and the MPC firmware, preloading them before the 2.9.0 mpc software greatly reduces that popping and cracking the synths often give off and really widen out the reverb effect. It would appear that one of these librarys is a major sound fix. Test with and without them with tubsynth and you'll notice a vast difference. I included this in the first version of my firmware before both programs (the one I forked) but when I released the second version I only preloaded those libs when FORCE is enabled. oopsies! Oh well those firmwares were more or less demos.

	export LD_PRELOAD="/usr/lib/libSOUL_PatchLoader.so /usr/lib/libcares.so.2.4.0 /usr/lib/libbluetooth.so.3.19.2" 
	

	
