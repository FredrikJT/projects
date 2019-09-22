### Netgear Nighthawk R7000: Fix broken router login password

## The problem
I was not able to log in to the router because the root password was rejected. I was not able to reset the password by doing any resetting with the buttons on the router.
In the end, I spent four evenings solving this. My hope with this sloppy written story is that someone with the same problem might save some time.

## The solution
Serial communication between Raspberry Pi zero and the router in order to read nvram http_password and make it accept a firmware pushed via TFTP.

## Prelude
I bought a nice R7000 which had dd-wrt installed on it. When plugging it in to my home network it was connected to the internet via a gateway. I did not manage to get internet connection on the router, which made me want to install a different firmware on the router.
I downloaded a dd-wrt firmware, logged in on the router and started to upload the firmware .chk file on wifi. After what seemed to be a successful firmware update the problems began... Now I was not able to log in to the router at all, I tried all standard username-password combinations like root - admin, admin - admin, root, password etc. etc. Then started the googling for solutions.



## First try - vmlinuz with TFTP server (fail)
After googling around I found this guide: [Netgear R6300v2 Advanced Debrick Notes By Sploit](https://forum.dd-wrt.com/phpBB2/viewtopic.php?p=1049230)
This suggests to set up a TFTP server on ip 192.168.1.2 and on that server make a file called vmlinuz which is actually the Initframfs of a Openwrt firmware. After compiling the Openwrt firmware I did not understand what file would be the Initframfs. I did however try to get the router to pull a `.chk` Openwrt firmware file from the TFTP server that I set up. I did not succeed with this. 

## Second try - serial communication (success)
I found this guide: [How to Debrick or Recover NETGEAR R7000, R6300v2, or R6250 Wi-Fi Routers](https://www.myopenrouter.com/article/how-debrick-or-recover-netgear-r7000-r6300v2-or-r6250-wi-fi-routers)
This guide suggests using a USB-TTL cable, which I did not have. Instead I used a Raspberry Pi Zero W to connect to the pins as shown in the picture in the guide. The setup of UART on the RPi was smooth after finding this guide: [The Raspberry Pi UARTs](https://www.raspberrypi.org/documentation/configuration/uart.md)
The cables between the RPi and the R7000 pin-header was wasy to connect after looking at a Raspberry pi zero pinout image. The cables were connected like this:
- Rpi Ground (pin 6) -> R7000 Ground
- Rpi GPIO14 (pin 8) TxD -> R7000 RxD 
- Rpi GPIO15 (pin 10) RxD -> R7000 TxD

Before trying to communicate with the router, I verified that the serial communication worked on the Rpi by connecting the Rpi Txd to the Rpi Rxd and running the serial_read.py and serial_write.py scripts found here: [Read and Write From Serial Port With Raspberry Pi
](https://www.instructables.com/id/Read-and-write-from-serial-port-with-Raspberry-Pi/)
The only alteration I had to do was to change the line `port='/dev/ttyAMA0'` into `port='/dev/serial0'` for it to work.

After making sure the serial communication worked and using the serial settings proposed in the "How to debrick or Recover"-guide (Serial line: /dev/serial0
Speed: 115200
Stopbits: 8-N-1), I connected the Rpi and router together, booted the router and was able to read console data from the router.
I was not able to write tho.
Therefore I installed minicom and was now able to enter the CFE prompt and issue the tftpd command.

I connected a windows 10 computer via ethernet cable to one of the routers lan ports.
The computer had the ubuntu app installed as well as a Tomato by shibby .chk firmware file for the R7000 router downloaded.
In the Ubuntu app I used these commands to send the firmware to the router:
`tftpd`
`connect 192.168.1.1`
`mode serial`
`put [full name of .chk firmware file]`
These commands were found after looking on this page: [TFTP commands](http://www.process.com/docs/tcpware6_0/html/users/chapter_13.html)

With this I managed to get the router to install a Tomato by shibby firmware.
But the initial problem remained, I was not able to log into the router, the root password was denied.

I let the router finish booting, and with minicom pressed ctrl-c which let me enter a linux prompt. In this prompt I wrote `nvram show` as suggested in this thread: [R7000 bricked?](https://www.myopenrouter.com/forum/r7000-bricked-flashed-dd-wrt-tomato-cannot-login-userpw-denied?page=1) all parameters of the nvram showed, and in this I found the parameter `http_password`.
Going to `192.168.1.1` and logging in with root as the username and this http_password worked.

Now I was basically back to where it all began.

I found this guide: [Configuring a second router as a WiFi access point using Tomato by Shibby](https://blog.ostermiller.org/configuring-second-router-wifi-access-point-using-tomato-shibby/) which let me set up my R7000 in a correct way. After following this tutorial everything worked!
