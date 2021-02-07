
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
		
		echo "Launching MPC X Mode" >> /media/acvs-synth/cfw.log
		
		## ORIGINAL MPC SOFTWARE
		touch /tmp/force
		
		#unmount the magic
		umount "/sys/firmware/devicetree/base/inmusic,product-code"
		umount "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		## mount the magic
		mount -o bind "/root/models/x/inmusic,panel-rotation" "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		mount -o bind "/root/models/x/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"
		
		cat "/sys/firmware/devicetree/base/inmusic,panel-rotation"  >> /media/acvs-synth/cfw.log
		cat "/sys/firmware/devicetree/base/inmusic,product-code"    >> /media/acvs-synth/cfw.log
		
		if type systemd-inhibit >/dev/null 2>&1
		then
			exec systemd-inhibit --what=handle-power-key /usr/bin/MPC "$@"
		else
			ulimit -S -s 1024
			exec /usr/bin/MPC "$@"
		fi

	else

		echo "Launching FORCE Mode" >> /media/acvs-synth/cfw.log
		
		##FOCE SOFTWARE
		rm /tmp/force
		
		#unmount the magic
		umount "/sys/firmware/devicetree/base/inmusic,product-code"
		umount "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		
		## mount the magic
		mount -o bind "/root/models/force/inmusic,panel-rotation" "/sys/firmware/devicetree/base/inmusic,panel-rotation"
		mount -o bind "/root/models/force/inmusic,product-code"   "/sys/firmware/devicetree/base/inmusic,product-code"
		
		cat "/sys/firmware/devicetree/base/inmusic,panel-rotation"  >> /media/acvs-synth/cfw.log
		cat "/sys/firmware/devicetree/base/inmusic,product-code"    >> /media/acvs-synth/cfw.log
		
		export LD_PRELOAD="/usr/lib/libSOUL_PatchLoader.so /usr/lib/libcares.so.2.4.0 /usr/lib/libbluetooth.so.3.19.2" 
		
		if type systemd-inhibit >/dev/null 2>&1
		then
			exec systemd-inhibit --what=handle-power-key /usr/bin/FORCE "$@"
		else
			ulimit -S -s 1024
			exec /usr/bin/FORCE "$@"
		fi
		
	fi
	
