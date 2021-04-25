# smart_meter_setup
How to setup everything to readout your home smart meter
This is a work in progress and used to document all needed steps (and tests I am doing)

http://www.cip.ifi.lmu.de/~hailer/sites/energy_counter_doc.html

## Test serial port connection to smart meter
Setup serial port to check the connection 

``` console
stty -F /dev/ttyUSB0 9600 -parenb cs8 -cstopb -ixoff -crtscts -hupcl -ixon -opost -onlcr -isig -icanon -iexten -echo -echoe -echoctl -echoke
```

Test output from serial 
``` console
cat /dev/ttyUSB0
```

If the data looks unreadable you meter returns the data in the SML Format

## Setup VZLOGGER

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
