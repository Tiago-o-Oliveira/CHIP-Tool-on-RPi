This Document describes a simple setup where a Linux Host communicating with a RCP forms a Matter Fabric, it is based on [Matter Over Thread Demo](https://siliconlabs.github.io/matter/latest/nav_4_thread.html)from Silabs.

![[Pasted image 20260218104240.png|center]]
### Pre-Requisites
#### Software
- [Balena etcher](https://etcher.balena.io/);
- [Simplicity Commander](https://www.silabs.com/software-and-tools/simplicity-studio/simplicity-commander).
#### For RCP
- [BRD2703](https://www.silabs.com/development-tools/wireless/efr32xg24-explorer-kit);
- USB-C to USB-A Cable.
#### For Linux Host
- Raspberry PI 4;
- Ethernet Cable;
- USB-C Source (Prefered 5V 2A);
- SD Card at least 8Gb.
#### For matter End Device
- [BRD4187C](https://www.silabs.com/development-tools/wireless/xg24-rb4187c-efr32xg24-wireless-gecko-radio-board?tab=overview);
- PCB4001 or [PCB4002](https://www.silabs.com/development-tools/wireless/wireless-pro-kit-mainboard?tab=overview);
- USB-A to USB-Bmini Cable;

#### Pre Built Images
- All the Pre-Built binaries are available on: https://siliconlabs.github.io/matter/latest/general/ARTIFACTS.html
### Flashing the Raspberry IMG

As part of its demo, Silabs Provides an PreBuilt Ubuntu Image that already loads [CHIP-Tool](https://project-chip.github.io/connectedhomeip-doc/development_controllers/chip-tool/chip_tool_guide.html),[OTBR](https://github.com/openthread/ot-br-posix)and the MatterTool. The MatterTool is a tool built by Silicon to ease the testing in the demo.

Balena Etcher is the preferred tool for flashing the image in the SD Card, be aware that the image is ~7Gb large, but other tools like the [Raspberry Imager](https://www.raspberrypi.com/software/)can be used as well at own risk, for all that is worth, this was tested using Etcher.

### Flashing The RCP Device

SiliconLabs also provides pre-built bootloader and rcp binaries for several boards, alternatively both can be built from source using simplicity studio.

Connect the kit to a PC with simplicity commander to proceed the firmware flash on the rcp board
Once in possession of the binaries, flash both the bootloader and the rcp .s37 to the kit (BRD2703 in this case), both the bootloader and the rcp firmware must be flashed otherwise it won't work.
### Flashing the MAD Device
This is very similar to the previous step, both the bootloader and the firmware must be flashed to the BRD4187C board. 

In this example the sample "Sensor Example" was chose since it expose several different sensor types and nodes to the network, which will come handy later in the testing phase, also for the sake of testing the firmware version used is the ones that support the PCB4001 built in LCD screen.

### Finding The Raspberry IP
Once Everything is Flashed, connect everything as described on the image on the top of this page, the test setup looks like this:
![[WhatsApp Image 2026-02-18 at 14.25.58 1.jpeg|center|500]]
The Gray cable is going to the internet router and the black goes to the 5V source.


Now run the command on the Remote Terminal (A PC running ubuntu in this test) and run the command ```ip a``` this should return the list of network adapters, and their parameters, in my case it is wlp1s0, inside your adapter parameters, search for the *inet* value, copy it and paste it on the command ```nmap pasteInetHere``` this will return all the ips find in the same network of your computer (if not obvious yet, the PC and the raspberry must be attached to the same network), the command response looks like this:
![[Pasted image 20260218144017.png|350]]

In this case i've probed it and now that 192.168.1.12 is my raspberry IP, Silicon describes [another method of  finding raspberry IP](https://siliconlabs.github.io/matter/latest/general/FIND_RASPI.html) but be aware that your network adapter may not support raw ARP, in this case, like mine, if your network is small you can probe all the IPs found until you find the right one.
### Connecting To Raspberry Via SSH

Once figured the IP, run this command on the remote terminal ```ssh ubuntu@192.168.1.12``` with your raspberry IP, the terminal will prompt you for a password, for this first access, the password is ubuntu . Once you type the password, it will prompt for a password change, choose your password and for now on, every time you access the raspberry via ssh, use the new password.

After logging again, you may see this screen:
![[Pasted image 20260218145610.png|400]]
It means we're in, now for the fun part.
### Starting the Matter Fabric
If the setup is going well so far, it's time to poke it to see if thats 100% true. First we can probe the otbr-agent who is responsible for communicating with the rcp and managing the OpenThread network, to do so, run the command ```sudo systemctl status otbr-agent```, if the response says it's active, like this:

![[Pasted image 20260218150026.png]]

Then everything is alright, if it says otherwise, try this command:
```bash
sudo systemctl restart otbr-agent
```
Wait a minute and probe it again, if it still not active, the you can have a problem caused by:
- All the reasons cited [here](https://siliconlabs.github.io/matter/latest/thread/RCP.html) plus:
- The board you're trying to use is not working properly, it's good to have a spare one just in case
-  The USB cable is not properly working, in this case the option is replace it just for sure.

Once you got this process working, you can use the MatterTool to start the Fabric, by running
```bash
mattertool startThread
```

If no errors appear, we're ready to proceed to comissioning, otherwise probe again the otbr-agent.
### Commissioning a Device
For commissioning the MDA, first put it into advertising mode, this will vary from example to example, read carefully the readme for the example you're trying out.

Once the device is advertising, run the command
```bash
mattertool bleThread
```

If everything is right, after a quadrillion lines of information we aren't diving in now, you should see:

![[Pasted image 20260218151502.png]]

The device is now on the fabric and, although the Matter-Tool is very useful and more of it can be learned at the [documentation](https://siliconlabs.github.io/matter/latest/thread/CHIP_TOOL.html), this test will need bigger guns.
### Listening Devices With CHIP-Tool

With a way to add devices to the network and some control over then, the next step SiliconLabs would suggest is the use of the MatterTool to read the values from devices on demand, which can be fun, but not very useful for more than a test our two, the next step is to follow the information in real time. For this purpose, we have the CHIP-Tool, it gives us full control over our Matter Fabric, and we're going to taste it.

First, we must find where chiptool lies, by running the command
```bash
echo $CHIPTOOL_PATH
```

![[Pasted image 20260218152347.png]]

We then go to the given directory
```bash
cd /home/ubuntu/connectedhomeip/out/standalone/
```

![[Pasted image 20260218152443.png]]

in this folder we can use the chiptool commands, as a test, run the command ```./chip-tool``` it should print a list of all supported clusters.

Now put the chip-tool in the interactive mode
```bash
./chip-tool interactive start
```

In this mode, it won't behave like a normal CLI, now it allows you to subscribe to specific clusters, receiving URCs.

The command synthax for subscribing is the following:(More on each field can be found on chiptool documentation)
```bash
<cluster-name> subscribe <argument> <min-interval> <max-interval> <node_id> <endpoint_id>
```

The MDA used in the test supports the occupancysensing cluster on endpoint 1, so we're going to subscribe on it:
```bash
occupancysensing subscribe occupancy 5 10 31369 1
```

Now every time the occupancy parameter changes on the sensor, the terminal shows the following message:

![[Pasted image 20260218163857.png]]

As a final note, when testing this last bit, i got a problem with the mDNS server running on the controller, after much much hassle, resetting the raspberry fixed it, but it should be noticed.