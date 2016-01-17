# RF Wireless Outlet using Raspberry Pi (including Zero) & Etekcity RF Power outlets
#### Have you ever wanted to wirelessly control power outlets from your phone using a Raspberry Pi?
In this post you’ll find instructions for using a Raspberry Pi to wirelessly control Etekcity power outlets using 433MHz RF.

### Hardware
- [Etekcity Wireless Remote Control Electrical Outlet Switch for Household Appliances](http://www.amazon.com/Etekcity-Wireless-Electrical-Household-Appliances/dp/B00DQELHBS/ref=pd_sim_60_2?ie=UTF8&dpID=410-DyROwEL&dpSrc=sims&preST=_AC_UL160_SR160%2C160_&refRID=126QG4QF1ZY68K2YP2YB)
- Raspberry Pi ( I've used new Raspberry Pi zero $5 computer board)
- [Wireless Adapter (for getting internet in pi using wi-fi)](http://www.amazon.com/Edimax-EW-7811Un-150Mbps-Raspberry-Supports/dp/B003MTTJOY/ref=sr_1_4?s=electronics&ie=UTF8&qid=1452993716&sr=1-4&keywords=wireless+adapter)
- [Alternate to Wireless Adapter, you can use Ethernet Adapter to get direct internet](http://www.amazon.com/Plugable-Gigabit-Ethernet-Network-Adapter/dp/B00AQM8586/ref=sr_1_19?s=electronics&ie=UTF8&qid=1452993797&sr=1-19&keywords=USB+3.0+ethernet+adapter)
- [RF Transmitter & Receiver kit](http://www.amazon.com/SMAKN%C2%AE-433Mhz-Transmitter-Receiver-Arduino/dp/B00M2CUALS)
- [Jumper Wire set](http://www.amazon.com/Breadboard-Jumper-Wire-75pcs-pack/dp/B0040DEI9M)

***Other Required Items:***
- Soldering iron
- 12″ strand of wire (for antenna)

### Step- 1 Install OS on SD Card for Raspberry Pi
- Install [Raspbian](https://www.raspberrypi.org/downloads/) from Raspberrypi.org. You can download "Noobs" and extract it to folder. Later you have to copy that extracted folder to SD Card.
- [Full instruction on how to install Raspbian](https://www.raspberrypi.org/help/noobs-setup/)

### Step - 2 Installing WiringPi
WiringPi is a prerequisite package required by RFSniffer and codesend.

The source files for the software can be pulled using git.
```
git clone git://git.drogon.net/wiringPi
git clone https://github.com/bapatel1/WiringPi // (alternate link if above doesn't work)
```
Execute the build script to compile the code.
```
cd wiringPi
./build
```
To confirm that the build process was a success you can issue the following command.
```
gpio -v
```
You will see copyright message and few raspberry pi details.

Finally you can issue the command below to confirm that wiringPi can read from the GPIO pins.
```
gpio readall
```
If successful you’ll see a listing for all of the gpio pins and their values..

### Step - 3 Install Apache2 and Php for website and server
The Apache web server is used to serve the toggle.php page which will provide a web interface for controlling the wireless outlets.

Use the command below to install Apache with the php modules.
```
sudo apt-get install apache2
sudo apt-get install php5
sudo apt-get install libapache2-mod-php5 -y
```
To confirm that Apache is working put the IP address of your Pi into your web browser, you should see the default Apache test page.
e.g. Run  http://192.168.1.128 on Pi's browser and you will see test page from Apache2 and see some message like "It works".

### Step - 4 Add Antena to Transmitter module
Without an antenna the transmitters range is extremely limited, mine wasn’t able to reach outside the room without one.  I took Tim’s recommendation and used a 12″ piece of wire.

You should see a hole with "ATN" label or something on small transmitter module (which will have 3 pins). Attach one wire to it and that will be your Antena for your transmitter.

### Step 5:  Connect the RF Transmitter and Receiver Modules to the Pi
- The transmitter module is the smaller module which has 3 pins, VCC, data, and ground.
- The receiver is the larger of the two modules with 4 pins, VCC, 2 data pins, and ground.  Only one of the data pins on the receiver is used for this project.

```
Transmitter Module
*******************
DATA (left pin) -> GPIO #17
VCC (center pin) -> +5VDC
GND (right pin) -> Ground

Receiver Module
****************
VCC (left pin) -> +5VDC
DATA (2nd pin from left) -> GPIO 21/27
GND (far right pin) -> Ground
```

### Step 6:  Use RFSniffer to Find the Outlet Control Codes
In this step you’ll use a program called RFSniffer and the 433MHz wireless receiver to read the on and off codes for each pair of buttons.

First pull down a copy of [my rfoutlet code from github](https://github.com/bapatel1/rfoutlet) alterntive you may also get [Tim’s rfoutlet code from GitHub](https://github.com/timleland/rfoutlet).  
```
git clone git@github.com:bapatel1/rfoutlet.git /var/www/html/rfoutlet
```
Set the appropriate ownership and permissions on the codesend executable.
```
sudo chown root.root /var/www/html/rfoutlet/codesend
sudo chmod 4755 /var/www/html/rfoutlet/codesend
```
To sniff the codes run the RFSniffer program.
```
sudo /var/www/html/rfoutlet/RFSniffer
```
You won’t see any output on the console when you run the program but if everything is connected properly you should see output after pressing some of the buttons on your remote.

Each press of a button on the remote should produce something like this:
```
Received 21811
Received pulse 192
```
What you’re looking for is the longer number, not the short 3 digit pulse.  In the example above 21811 would be the code you’re looking for.  Record the code for each of the on and off buttons on the remote.

Here are a few of my notes about reading the codes:

1. The receiver is not very sensitive so make sure you have the remote nearby when reading the codes.
2. I found that sometimes I had to press a button multiple times before a code was received, it’s not a 100% reliable process.
3. If it’s not working, double check the wiring of the receiver module (the longer board with 4 pins)
4. For further troubleshooting try connecting an LED to the data pin of the receiver, it should blink when receiving data.

### Step 7:  Update toggle.php with the Codes for Your Remote
Using your preferred text editor edit /var/www/html/rfoutlet/toggle.php with the codes you recorded in step 5.  You will also need to update $rfPath to point to the correct path which in my case was /var/www/html/rfoutlet/codesend.

### Step 8 Testing and Troubleshooting
Testing the Web Outlet Controls

At this point everything should be done and ready to be tested.  The php based web control page can be accessed by visiting the http://<your-pi-ip>/rfoutlet in your browser.

The on and off buttons on the page should function just as they would on your physical remote.

##### Troubleshooting if error
If the buttons on the web page don’t work then try manually sending a code using the command line.
```
root@raspberrypi:/home/pi# /var/www/html/rfoutlet/codesend 21820
sending code[21820]
```

If sending a code manually works then the transmitter is functioning but there is an issue related to the web setup.  Make sure that you made the correct modifications to toggle.php, specifically the $rfPath variable is pointing to the correct path.

You can also check the apache server logs to see if there is a syntax error in the toggle.php file.
```
tail -100 /var/log/apache2/error.log
```
If the manual code send isn’t working then check to make sure the transmitter is wired properly.  You can also connect an LED to the data pin of the transmitter to confirm that it is receiving output from the Pi, it should blink when you send a code.


**Referred Blog Post:**
- [TimLeland.com/wireless-power-outlets](http://timleland.com/wireless-power-outlets/)
- [How to control power outlets wirelessly? from Sam Kear](https://www.samkear.com/hardware/control-power-outlets-wirelessly-raspberry-pi)
