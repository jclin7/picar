#picar

![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/DSC_0216.jpg)

##Overview
Use your raspberry pi to control a 1/10 scale RC car via a web page hosted wirelessly on your PI.  All you need to do is set up your PI to use your mobile as a hotspot then log onto the appropriate web page and tilt your phone to control your car like a Wii Remote.  I've used an old Tamiya hornet in this example; any car will do but if you are buying one try and get one with enough space under the bodyshell to fit all your electronics.

This is a fork of an existing github project that I've modified to make easier to install and to get rid of some components.  It's also possible to add a video camera but I didn't do this.

##Electrical
This project is based on [shaunuk/picar] but replaces the servo board with a soft PWM driver on the PIs GPIO pins.  It's also possible to get rid of the battery power supply for the PI depending on your specific setup.

https://www.youtube.com/watch?v=JSP6VKiU7F4

###Pi Power supply
When it comes to powering the PI it is necessary to have a fairly stable 5V power supply otherwise the Pi keeps resetting when you drive the motor.  For this example I had an old NiCad battery pack which dropped a lot of voltage when the car accelerated which caused the PI to keep resetting.  To get round that I've added an additional 7.2V AA battery pack and a 5V linear power supply to give a clean 5V supply to the PI independent of the motor demands.  If you have a new NiMH battery you may not need this and might get away with powering off the 5V offered by the ESC.  

To see if this is an option you'll need to use a multi-meter and check how much that 5V line drops when your RC car is accelerated; you might want to put some load on the wheels as well to further load the battery.  You'll also want to check the 5V is fairly stable and doesn't rise 5.5V at any point.  If you’re happy with your on-board 5V supply you can remove the linear regulator and additional battery pack; you then just move the 0V and 5V wired from where the regulator was to the corresponding pins on the ESC (middle pin that's unconnected on my schematic).  

I bought an old RC of ebay and found the supplied NiCad dropped too much voltage when the car accelerated. Note - After writing this I bought a new battery that allowed me to run the Pi direct from the ESC.


    |  Old NiCad  | New NiMh
  ------|-------------|------
  Min ESC V  | 3.4  |  5.1
  Max ESC V  | 5.5  |  5.5 



There are a number of RC car electrical setups but my example uses an ESC and receiver with battery eliminator circuit .  The ectronics supply normally comes from the electronic speed controller (ESC) and powers the receiver and steeirng servo with 5V (note these devices are more tolerant of power supply dips than the Pi).  The receiver normally receives commands from the radio controller than sends them to the ESC (throttle) and steering servo (steering).  These commands fall within the 0-5V supplied by the ESC for example: 1V = steer fwds; 1.5 = steer right; 0.5 = steer left.  Looking at the data sheet for my ESC I it could supply up to 1A which would have been enough for my Pi and my Steering Servo if the voltage had have been stable enough.

###Servo signal levels
We'll be using PWM to control the servos which will be capable of driving 0-3.3V in 3.3mv steps; it is therefore necessary first to measure what your servo command signal voltage levels are and check they fall within this range.    You can do this with a multi-meter connected to the receiver pins.  On my car speed and steering both use 3 pin headers which are wired:
<ul>
   <li>Gnd - Black</li>
   <li>Power - Red</li>
   <li>Throttle / Steer - Orange or White</li>
</ul>

For my car the throttle voltages were:
<ul>
   <li>Full Fwd:     .19V</li>
   <li>Idle:         .28V</li>
   <li>Full Reverse: .36V</li>
</ul>

The steering voltages were:
<ul>
   <li>Full Left:    .19V</li>
   <li>Fwds:         .28V</li>
   <li>Full Right:   .355V</li>
</ul>

Once you are happy about how you are going to power your PI and that the GPIO are up to the job you can start thinking about wiring it up.

![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/picar_scematic.PNG)

For the 3 pin headers I used 2.54mm PCB header and soldered wired direct.  I then used superglue to stop the pins moving about.  You'll also need heat-shrink or equivalent to cover over the solder joints.
To power the PI I chopped a micro USB cable and used the black and red wires from that as my 5V supply.
For my linear power supply I used an LM7805 circuit and put a heatsink on it to keep it nice and cool:

![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/reg cct.PNG)

I then attached a PP3 battery clip and 6xAA pack with a PP3 connector on it.
Check if your battery and ESC are up to powering the car in order to avoid having to go down the extra battery route.  
I've also used a 26w header socket to attach to the raspberry PI GPIO lines; I like this method as it means it's hard to mi-wire when re-connecting plus you can quicly remove you Pi as required.

You can see here how I've packaged everything up in the car.
![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/DSC_0219.jpg)
The Pi Is placed in a plastic bag for protection; it's worth putting peice of card underneath it to make sure the PCB doesn't poke through the plastic.
![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/DSC_0220.jpg)
Here you can see the wiring harness I made up including the linear regulator PCB and GPIO and interface connectors.
![](https://github.com/lawsonkeith/Pi-Rc-Car/raw/master/media/DSC_0221.jpg)

##Software 
The Pi uses node.js to run a web server; a wi-fi dongle on the PI uses your phone as a wireless hotspot to enable wifi communications.  Once you enter the web address of the PI a dialog box appears prompting you to begin racing; at that point you can control your car by tiliting oyur phone.  An emergency stop is built into the app so if it loses comms to your phone the vehicle stops accelerating and steers forwards.  The Pi-Blaster program allows pins 17 and 18 of the PI to act as PWM outputs and control steering and throttle.

The app runs on raspbian using a raspberry PI rev B although you could use the A model.  

To setup your Pi you need to download some software packages then setup your Pi to run the node.js program on boot.

###Get the app 
First we need to clone the app from github and place it in it's own directory.
<ul>
<li>[cd /home/pi]</li>
<li>Get this project from GITHUB.</li>
<li>[sudo git clone https://github.com/lawsonkeith/picar]</li>
<li>[cd picar]</li>
<li>Note - do everything in this dir from now on unless instructed otherwise.</li>
</ul>

###Download node.js
Download a new version of node.js
<ul>
<li>[sudo wget http://nodejs.org/dist/v0.10.21/node-v0.10.21-linux-arm-pi.tar.gz]</li>
<li>Then unzip it.</li>
<li>[sudo tar -xvzf node-v0.10.21-linux-arm-pi.tar.gz]</li>
<li>[sudo rm node-v0.10.21-linux-arm-pi.tar.gz]</li>
<li>Create symbolic links to the node executables</li>
<li>[sudo ln -s /home/pi/picar/node-v0.10.21-linux-arm-pi/bin/node /bin/node]</li>
<li>Package manager</li>
<li>[sudo ln -s /home/pi/picar/node-v0.10.21-linux-arm-pi/bin/npm  /bin/npm]
</li>
</ul>

###Download additional node packages
Next we use the node package manager (npm) to install some packages that we are going to use in node to give us the functionality required to control our RC car.
<ul>
<li>[sudo npm install socket.io node-static sleep optimist pi-blaster.js]</li>
</ul>

###Download PI Blaster soft PWM daemon
The Pi blaster node library also requires a daemon to be downloaded that runs in the background and runs the PWM.  Have a look at [https://github.com/sarfata/pi-blaster.js/] to get some more info about this package.

*[sudo git clone https://github.com/sarfata/pi-blaster]
*[sudo apt-get install autoconf]
*[cd pi-blaster]
*[sudo autogen.sh]
*[sudo ./configure]
*[sudo make]
*[sudo make install]
*[cd ..]

Note - you will want to be able to stop the daemon; see note at end.


###Setup your PWM defaults
I've included a node script file called pwm_test2.  Tou run this you enter:
<ul>
<li>[node pwm_test2]</li>
</ul>

We are now going to integrate the pi int the RC car and check we can control the servos.  You can either connect your Pi to your TV and a keyboard or setup vpn to do this.  

The pi-blaster node setPwm() API requires a pin and demand parameter.  For example setPwm(17,.5) would set ppin 17 to 50% PWM demand or 3.2/2 V.  First off make sure you are happy
with how the API works and make sure you can set the output voltage on pins 17 and 18 using a meter.
Once you are happy with this you can connect up your steering servo and work out the constants you
Need to send to get it to steer left; right forwards and backwards.  Make sure that when you go full left or 
Right that the servo isn't over driving as you can damage the servo.

Do the same with throttle; the demand magnitudes should be the same so it should be just a case of proving the
Wiring and signal directions.

You should now be fairly confident about controlling the servos and powering your pi from the RC platform.  Just modify what's in 'pwm_test2' and keep running it till you are happy with how it all works.

Enter your platform specific constants into the 'readme' file; we'll use them later to do some scaling.

###Configure PI to use Smartphone WiFi
We now need to set your Pi up to use your phone as wifi.

* On your phone enable the wifi hotspot option.
* On your Pi disable the existing WiFi option and connect to your phone.
* You may need to enter a network key to do this.
* Check it all works by accessing the internet from your Pi.
* Reboot your Pi and check it still all works.
* You are now going to set your Pi IP address to static.
* Enter [ifconfig] write down your IP address for wlan0.
* Edit your network setup file [sudo nano /etc/network/interfaces] file and set to to the current wlan0 IP address and from dhcp to static using the 'interfaces' file in this project as an example.
* Reboot the Pi and check all still works; your IP address will differ.

###Setup your smartphone defaults
With the Wifi Setup you can now test out the phone.
* [node app.js]
* Navigate to http://PI_IPADDRESS:8080/socket.html on your phone.
* A message comes up and asks you to race.
* On the Pi you should see the values being sent to the setPWM API by the web page running on your Pi.
* If these work; great you can skip the next step.

###Setup html file for your setup
You may need to setup the scaling specific to your platform...
* In socket.html comment out the min and max gamma if statements.
* Also comment out the scaling M and C terms so the event.beta and event.gamma numbers are sent to the Pi.
* Run up your app [node app.js] and make a note of the raw numbers in the specific orientation you want to use your phone in.
* The nummbers are degrees; in my example I tilted +/- 30 degrees to steer and 0 was full fwds and 60 full reverse.
* Put your numbers in the readme file
* Using excel or a calculator work out the constants you need to use to convert your phone tilt values into the setPWM API demands required to control your car.
* Uncomemnt out the min max demands if construct and enter your specific min max constants at the top.
* Now edit app.js.  In the emergensy stop function enter the values for steering and throttle that stop your Pi.

Now run upp the app a few times and convince yourself it's all working.
 

###Configure PI to run node app by default
We now need to configure the app to run as default when we power up the Pi.

* [sudo cp node-server.sh /etc/init.d/]
* [sudo update-rc.d node-server.sh defaults]

Reboot your Pi and check you can log onto the web page.  You should now be ready to race.

#Issues
PiBlaster can cause issues with the Pi windows environment; mine kept crashing when I moved them when it was running.  If you stop running it the problem will go away.
[sudo /etc/init.d/pi-blaster stop]

For more command info see the 'readme' file.
