# RadioZ (internet radio and Google assistant in an old wooden radio case) ::: PYTHON project

Intro:
I always wanted to build an internet radio which is a real product like other consumer electronics, not just a bunch of wire on my desktop.

Concept:
some years ago I have modded my wifi router and used it as an internet radio for a while (Openwrt,MPD,audio card on usb port) but it was a very simple and useless solution becuse the router was on a fixed place in my house and it was clear after some time that a normal person will never listen radio in the staircase...


Planning & procurement:
- it must look cool :)
  I love old radios. In the century XX, after the 2nd world war radios were built in wooden cases, and due to vacuum tube technology they were huge. (and beautiful :) )
So I started to looking for an old radio which I could use. I found that due to their price I will have to choose a model that was popular. I found an old ORION AR301 ( http://www.radiomuseum.hu/orionar301.html ) on jofogas.hu (hungarian ebay) but an unwaited thing happened. When I plugged it to the wall plug It worked!!! I was not able to hurt it. I had to find another one: an empty wooden radio case. Fortunately I found another wooden case after some search (probably this type http://www.radiomuseum.hu/orion228.html)  but it had to be cleaned from an ugly dark red painting. Originally those radios were only lacquered. I forgot to take a "before" photo but you can see in the gallery what I had to do with it to make it look better.

- What about the brain?
 options:
  an old router with openwrt 
    pros: cheap, I know it would work because I have already made one. 
    cons: I am not sure I can easily control it by buttons/knobs
  arduino
    pros: cheap, 
    cons: I found only mp3 decoder for arduino so not all kind of stream would be supported, I am not sure that I can read any extra info from stream that I could print on the screen     
  raspberry (or RPi clone):
    pros: it is flexible, has a lot of IO ports, later I can add extra features.
    cons: not so cheap because more additional parts are needed compared to the arduino or router solution.
 ----------------------
 RPi WON. I bought an RPi2, a fast 8GB SD card, a 3A power supply, 4x20 LCD from Aliexpress
 



Hardware tasks:
- I want to control it by knobs, I want to "tune" it like a real radio.
 I searched some radio projects on the net and found that the tuning knob could be a rotary encoder.
 it is usually used in consumer electroincs as a volume adjust knob. It gives signals about that if it is turned and the direction.
 For using rotary encoder some resistors and wires are needed as well.
 I need also an On/off button and play/stop button.
 To get loud sound I ordered a cheap audio amplifier panel from Ali.
 Speakers are borrowed from an old CRT TV on the attic.

- Power on/off
 RPi does not have power button. If somebody just unplugs the power cord, the linux on raspberry cannot stop services and cannot unmount volumes on sd card. It can cause that RPI cannot boot properly next time. 
I wanted to solve this. I found a circuit on the web which can control the power by a button and some codes in linux. 
I've built that circuit (not exactly the same becuase I did not get that type of dual FET) for controlling power on the radio by a button on the front panel. Power off is controlled by a python code, it checks the power button during operation and when it is pressed the program shuts down linx but just before it reaches halted status it changes a GPIO pin level to opposit level of previous level. That GPIO pin controls the power supply panel and cuts the power. 

- Audio
  as RPi has PCM audio output only and I wanted better sound I have ordered an USB audio card form Ali. 
  It was a good idea because it is turned out that Google Assistand and Internet radio cannot use the same output. So I configured the radio to use the USB card and Google Ai to use internal audio (PCM) which has reasonable sound quality for human :) voice.

-LCD screen
 it was much simpler than I thought. It is also came from Ali. To be simpler I ordered one with I2C panel because RPi supports I2C communication standard. It means by using an I2C library in PYTHON I can write ASCII characters on the screen and as it is a serial communication standard it uses only 3 wires.

- Microphone for Google AI
It is a simple headset mic placed between the 2 speakers in hole. I have soldered to a 3.5 jack plug and connected to microphone connector on USB audio card.

- wifi
it is also from Ali, a simple wifi stick which is supported by Raspbian

- Software tasks:
The OS is Raspbian. I use it because it has the largest knowledge base on the net and I wanted to run Google assistant on the RPi and this OS was supporded by Google. (https://www.novaspirit.com/2017/05/23/voice-activated-google-assistant-raspberry-pi/)
(Currently I don't use Google AI because the google software seems buggy on raspberry and sometimes it suddenly hangs.)
When raspbian is started radio program starts automatically.
I wrote it in python because it was the most logical. It has I2C library, has GPIO library, it was already used for Google AI and so on.
Features of the radio software:
The radio checks internet connection and radio program will start only if internet connection is available. Otherwise "No Internet" message is shown.
 * wifi connection settings can be changed in wifi.conf file by changing 	ssid="myhomessid" and psk="password"  values.
  
 * station list can be changed any time by replacing a new stationlist.csv file on the SD card (no programming or linux knowledge is needed for that)
    file format: 
    Jazzy;http://streamer.hmx.hu:8000/jazzy.mp3
    MR2 Petofi;http://icast.connectmedia.hu/4738/mr2.mp3
    Atlantica Soleil;http://188.165.196.212:8054/stream
    African FM;http://majestic.wavestreamer.com:5430/Live
    ROCK RADIO SI.;http://212.18.63.135:9034/rock
    ROCK FM;http://s9.voscast.com:8868
    TED talks;http://tunein.streamguys1.com/TEDTalks
    WBUR Boston;http://wbur-sc.streamguys.com/wbur_tunein
    Boston Rock Radio;http://18193.live.streamtheworld.com/SAM01AAC220_SC?ua=RadioTime&ttag=RadioTime&DIST=TuneIn&TGT=TuneIn&maxServers=2
    Lolliradio Italia;http://94.23.67.172:8010/stream
    Radio_Blu_Monopoli;http://79.8.96.3:9000
    Radio Cubana;http://listen.radionomy.com/radio-cubana?type=.mp3/;stream.mp3
    1.FM - blues;http://185.33.21.112:80/blues_128 
    
When the radio is turned on the program compares the content of stationlist.csv file on VFAT partition to the stored copy of stationlist.csv on linux partition. The new station list will be used only for the radio if it is correct. Validation tries to add stream URL-s to MPC playlist and if it is not possible for any reason an error message will warn the user of the radio to fix the file. 

If the new file is correct the station list file will be copied to the linux partition and it will be used in the future.
If the stations are succesfully updated a message informs the user about it.

When radio is booted up, the station that was played last time will be loaded. 

During playing a channel the current station info text is shown on the screen wrapped to 4 lines. It is usually the title/artist of a music. If there is no such incoming data in the stream the name of station will be shown that is given in stationlist.csv
When tune knob is rotated LCD shows the name of current, previous and next station and scrolls the list up or down. 

The radio can be turned off by pressing On/off button.

used libraries:
- rotary encoder: https://bobrathbone.com/raspberrypi/packages/pi_rotary.tar.gz
I had to solve button handling. Id did not handle buttons I need extended it
- GPIO
- LCD
- I2C



