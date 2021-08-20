# smart_meter_setup
Since Energy Monitoring is a very big topic itself, I will try to pull all information on how to

* Technical Setup
* Usage of Energy Data
* and everything else that might be important

into this section.

## Technical Setup
In order to connect my energy meter to my Homeassistant installation I had to figure out a way to get the data off the meter and into the system. There are a couple of option

1. Get a real smart meter that offers some form of API or cloud service that you can use
2. Pull the information from the optical gateway that sends the data in the SML format
3. count the number of impulses on the interface

In my case I went with option 2 but I will also have a writeup for option 3 with the use of ESPHome soon

### Volkszähler installation

To collect the usage information via the optical gateway and translate the SML data into something useable I went ahead and setup [Volkszähler](https://wiki.volkszaehler.org/howto/raspberry_pi_image) on an old RasPi.

## Setup VZLOGGER (Not needed when using the RaspberryPi Image)

Install needed stuff 
````
sudo apt-get install build-essential git-core cmake pkg-config subversion libcurl4-openssl-dev libgnutls28-dev libsasl2-dev uuid-dev libtool libssl-dev libgcrypt20-dev libmicrohttpd-dev libltdl-dev libjson-c-dev libleptonica-dev libmosquitto-dev libunistring-dev dh-autoreconf
````

Download vzlogger
````
git clone https://github.com/volkszaehler/vzlogger.git
cd vzlogger
./install.sh

````

### Test serial port connection to smart meter
Setup serial port to check the connection 

``` console
stty -F /dev/ttyUSB0 9600 -parenb cs8 -cstopb -ixoff -crtscts -hupcl -ixon -opost -onlcr -isig -icanon -iexten -echo -echoe -echoctl -echoke
```

Test output from serial 
``` console
cat /dev/ttyUSB0
```

If the data looks unreadable you meter returns the data in the SML Format

## Volkszähler configuration 

After you have installed Volkszähler on your Raspberry it is time to start the configuration 

#### Checking for Data
To see what information is send from the smart meter we will open up a shell into the raspberry and start configuration of **vzlogger**


````shell
sudo nano /etc/vzlogger.conf
````

and enter the following basic configuration

````shell
{
"retry" : 0,                    /* sleep between failed requests (seconds) */
"daemon": true,                 /* run as deamon*/
"verbosity" : 15,               /* Loglevel between 0 (nothing) and 15 (higest) */
"log" : "/var/log/vzlogger.log",/* logfile path */

"local" : {
        "enabled" : false,      /* Enable / Disable local HTTP-Server for serving live readings */
        "port" : 80,          /* TCP port for the local HTTP-Server */
        "index" : true,         /* Provide a index listing of available channels */
        "timeout" : 30,         /* timeout for long polling requests (seconds) */
        "buffer" : 600          /* Buffer reading for the local interface (seconds) */
},

"meters" : [{
        "enabled" : true,           /* disable or enable meter */
        "protocol" : "sml",         /* use 'vzlogger -h' for available protocols */
        "device" : "/dev/ttyAMA0",  /* Serial Port of Photodiod */
        }
]}
````
now we need to restart vzlogger 

````shell
sudo systemctl stop vzlogger
sudo systemctl start vzlogger
````
and then log the output to the console
````shell
tail -f /var/log/vzlogger.log
````
and you should see something that looks like this...

````shell
[Aug 12 15:23:55][mtr0] Got 4 new readings from meter:
[Aug 12 15:23:55][mtr0] Reading: id=1-0:1.8.0*255/ObisIdentifier:1-0:1.8.0*255 value=6864421.20 ts=1628774635968
[Aug 12 15:23:55][mtr0] Reading: id=1-0:1.8.1*255/ObisIdentifier:1-0:1.8.1*255 value=6864421.20 ts=1628774635968
[Aug 12 15:23:55][mtr0] Reading: id=1-0:1.8.2*255/ObisIdentifier:1-0:1.8.2*255 value=0.00 ts=1628774635968
[Aug 12 15:23:55][mtr0] Reading: id=1-0:16.7.0*255/ObisIdentifier:1-0:16.7.0*255 value=363.00 ts=1628774635968
````

... ok, bunch of cryptic stuff. What now?

Lets break down the information we can see. 

[Aug 12 15:23:55] -> Date of collected data
[mtr0] -> ID of the meter we have configured before
Reading: id=1-0:1.8.0*255/ObisIdentifier:1-0:1.8.0*255 value=6864421.20 ts=1628774635968 -> OBIS ID, the collected value and the timestamp 

1-0:1.8.0*255 is the OBIS ID for the conusmed energy. See the OBIS Reference for more details

### Channle configuration 

In order to use the data we have seen before we need to create a channle for each OBIS ID that we want to monitor

````shell
sudo nano /etc/vzlogger.conf
````

````shell
           "channels": [{
                "api": "volkszaehler",      // middleware api, default volkszaehler
                "uuid": "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx", // Gesamt Bezug
                "middleware": "http://localhost/middleware.php",
                "identifier" : "1-0:1.8.0",    // Zählerstand
                "aggmode" : "max"
            },
````

The UUID you will need to create with the [middelware](https://wiki.volkszaehler.org/software/middleware/einrichtung#kanaele_im_frontend_anlegen)

# Debug Opions

Run vzlogger in shell to see any erros 

````
vzlogger -c /etc/vzlogger.conf
````
