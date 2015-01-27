Bird Feeder Assembly and Setup Instructions
===========================================

This version of the instructions pertains to the IRon curtain with plywood 
casing iteration, developed in the second half of 2014.

# TO DO #

*   Give all dimensions in both metric and imperial units.
*   Describe how to wire the detector.
*   Figure out a solution to the app_key, app_secret dilemma.  (Have everyone
    register their own app?)
*   Describe setting up Twitter integration.


# Parts list #

## Electronics ##

*   A Raspberry Pi.  I used a model A+, and the mounting holes in the casing are
    positioned to match the mounting holes on this model.  If you move the
    mounting holes, you should be able to use a model B+, too.  The older models
    A and B should be fine, too, but you may need to think a bit more about how
    to mount them securely within the casing.

*   The Raspberry Pi Camera Module.

*   A microSD card, preferrably at least 8GB.

*   A microUSB charger appropriate to the Pi, with a long cable.  I used
    [this one](http://www.adafruit.com/products/501).

*   A WiFi dongle compatible with the Pi, such as the Edimax EW-7811Un.  This
    is optional if you're using a Model B or B+ and want to connect the Pi to
    the Internet via an ethernet cable, rather than WiFi.

*   A high brightness IR LED such as the [IR333A](http://www.adafruit.com/products/387)
    or LD274.

*   A 38 kHz IR demodulator.  These instructions assume you're using the
    [TSOP38238](http://www.adafruit.com/product/157).

*   A PN2222A transistor.

*   Two resistors for the LED circuit.  If you're using one of the diodes
    listed above, they should be 2700 Ohms and 56 Ohms.

*   A ribbon cable compatible with the Raspberry Pi GPIO.  If you're using a
    model A+ or B+, you need a 40-pin cable; for the model A or B, a 26-pin
    cable.  Sets consisting of an appropriate connector (FC40-P or FC26-P,
    respectively) plus the cable itself are sold by [Adafruit](http://www.adafruit.com/products/862). 
    Alternately, you can get the two parts separately [on](http://www.amazon.com/gp/product/B00977GOCU) [Amazon](http://www.amazon.com/gp/product/B00LUT4LOG).


## Lumber ##

*   2 ft x 4 ft of 1/4" thick plywood, preferably smooth (sanded).

*   1/2" x 36" dewel.

*   3/4" x 36" dewel.


## Hardware ##

*   Standoffs for mounting the Raspberry Pi within the enclosure.  I used,

    *   Four M2.5 12.0 mm screws,
    *   Four M2.5 6.0 mm screws,
    *   Four M2.5 x 10 mm HEX standoffs.

    Such small metric standoffs and screws can be difficult to find, but they
    match the mounting holes on the Pi exactly.  I got mine from
    [Mouser](www.mouser.com).

*   Screws for securing the electronics enclosure.  Using screws, rather than 
    glue, to secure one of the sides will let you open it later if you need
    the Pi for other projects (or have to fix the electronics).  Any four
    screws will do, really, but the drawings assume you're using,

    *   Another four M2.5 screws, at least 6.0 mm.

*   Weatherproofing solution for the plywood, such as Helmsman Spar Urethane.

*   A sealant for the electronics enclosure, such as silicone caulk.

*   Hardware cloth with 1/4" wire spacing.  You only need about 12 by 10 cm,
    but you will likely need to buy a large roll.

*   An IR reflector.  I used a smooth but not quite mirror-like piece of
    aluminium.

*   A small hinge for the feeder flap.

*   Some fasteners for attaching the hardware cloth to the frame made of the
    1/2" dewel.  If you have access to a staple gun, it will work well.  I
    didn't, so I used 8 small wood screws (shorter than 1/2", the thickness
    of the narrower dewel) with washers.


## Tools ##

*   An HDMI cable, monitor and keyboard for setting up the Pi for the first
    time.  (These are not listed in the parts section as you only need them
    while setting up the device; when you're done, you can [SSH into the Pi][pissh]
    instead.)

*   A soldering iron and solder, for assembling the detector circuit.

*   A jigsaw, for cutting out the casing parts from the plywood panel.

*   A power drill with the following drill bits:

    *   2.5 mm (or 3/32")
    *   5.0 mm (or 13/64")
    *   7.0 mm (or 9/32")
    *   19 mm (or 3/4") hole saw

    General-purpose bits work fine, but lip-and-spur wood bits will produce
    cleaner edges.

*   A power sander.  Sadly, as it is the design requires sanding some parts 
    of the casing to fit, especially the longitudinal partition.  I used
    a [small random-orbit sander from Black and Decker.][BDMouse]


# Electronics and software setup #

## Setting up the Raspberry Pi ##

1.  Download the New Out Of the Box Software (NOOBS) and place it on the SD
    card by following the directions [here][NOOBS_setup].

2.  Get the Raspberry Pi running by following the official
    [Quick Start Guide](http://www.raspberrypi.org/help/quick-start-guide/).
    Choose the (default) Raspbian operating system.

3.  Get accurate internet time by running the following commands in the terminal window:

        sudo apt-get install ntpdate
        sudo ntpdate -u ntp.ubuntu.com

4.  Update the OS to the latest version:
    
        sudo apt-get update
        sudo apt-get upgrade
        sudo apt-get dist-upgrade

5.  If you plan to use the WiFi, set it up.  I used an Edimax EW-7811Un USB
    WiFi dongle and followed the [instructions here][edimax].
    Note that this particular adapter has a power-saving feature that will 
    cause it to go to sleep after a period of inactivity (about 10 hours in
    my experience).  If you want to be able to connect to the Pi at all times,
    for example for debugging purposes, it is best to disable this feature by
    creating a file,

        sudo nano /etc/modprobe.d/8192cu.conf

    and adding the lines,

        # Disable power management
        options 8192cu rtw_power_mgnt=0 rtw_enusbss=0

    Hit `Ctrl-X` to exit.  See [this thread][edimax_power] for more details.

6.  If you haven't done it during setup, run `sudo raspi-config` and enable the
    camera module.

7.  To disable the camera LED (which may scare the birds and produces
    artifacts in low-light pictures), run

        sudo nano /boot/config.txt

    and at the end of the file add a line,

        disable_camera_led=1

    Hit `Ctrl-X` to exit.


## Setting up the bird feeder software ##

1.  Get a copy of our code from GitHub to the Raspberry Pi.  There are various
    ways to do this, but the simplest is to install git,

        sudo apt-get install git

    and then use the command,
    
        git clone https://github.com/tpudlik/Birdfeeder.git

    This will create a directory Birdfeeder in the current working directory
    containing a copy of the code.

2.  Install picamera, the Python library for the Raspberry Pi camera, by typing

        sudo apt-get install python-picamera

3.  Install the pip Python package manager:

        sudo apt-get install python-pip

4.  If you want to upload your photos to Dropbox, install the Python Dropbox
    API:
    
        sudo pip install dropbox

5.  If you want to upload your photos to Twitter, install the Python Twitter
    API:
    
        sudo pip install twython

6.  Install the Python GPIO library:
    
        sudo apt-get install python-dev
        sudo apt-get install python-rpi.gpio


## Configuring Dropbox and Twitter ##

If you want your pictures automatically uploaded, rather than merely stored
locally on the device, you will want to set up Dropbox or Twitter integration.

### Dropbox ###

1.  Go to the `Birdfeeder` folder (where our code is located) and edit the
    `dropbox_authentication.py` file:

        nano dropbox_authentication.py

    Change the value of `app_key` and `app_secret` to,

        app_key = 'fqvaq6adohoxrgg'
        app_secret = 'bpt1q89jkhvdipj'

    Exit nano using `Ctrl-X`.
2.  Run,
    
        python dropbox_authentication.py

    and follow the prompts.  You will have to go to the Dropbox website and
    enter your username and password.  When you’re done, a file
    `access_tokens.py` will appear in the folder (if it wasn’t there already).


### Twitter ###

Making Twitter integration as easy as the Dropbox process described above is 
a work in progress.


## Wiring the detector ##

Consult the circuit schematic, which 



# Operating the bird feeder #

## Editing the parameters ##

Edit the file `parameters.py` by running,

    nano parameters.py

and set the parameters to their desired values.  In particular, make sure the
`TWEET` and `DBOX` variables (which control whether photos are tweeted and
uploaded to Dropbox) have the values you want them to have.


## Running the software ##

Once everything is set up, you can start the Paparazzi Birdfeeder by typing,

    sudo python main.py

When it’s time to deploy the birdfeeder, you will likely do the following:

1.  Place the birdfeeder in its designated location.
2.  Turn on the Raspberry Pi.
3.  SSH into the Pi from another machine.
4.  Run,
    
        sudo python main.py &
        disown -h %1

    The disown command will keep the process running even after you sever
    the SSH connection to the device.

A comprehensive log of all events (detections, photos being taken, photo
uploads, etc) is recorded as birdfeeder.log.

[pissh]: http://www.raspberrypi.org/documentation/remote-access/ssh/
[BDMouse]: http://www.amazon.com/Black-Decker-MS800B-Detail-Collection/dp/B001H0GC04
[NOOBS_setup]: http://www.raspberrypi.org/help/noobs-setup/
[edimax]: http://www.savagehomeautomation.com/projects/raspberry-pi-installing-the-edimax-ew-7811un-usb-wifi-adapte.html
[edimax_power]: http://www.raspberrypi.org/forums/viewtopic.php?p=458761