# Tally Arbiter GPO Listener
Tally Arbiter GPO Listener was written by Joseph Adams and is distributed under the MIT License.

Tally Arbiter GPO Listener is an accessory program that allows you to connect to a Tally Arbiter server and control GPO Output of a Raspberry Pi based on the incoming tally information.

To learn more about the Tally Arbiter project, [click here](http://github.com/josephdadams/tallyarbiter).

It is not sold, authorized, or associated with any other company or product.

To contact the author or for more information, please visit [www.techministry.blog](http://www.techministry.blog).

# Installing Python Libraries and Script
The Tally Arbiter GPO Listener Client uses the following libraries:
* `python-socketio[client]`
* `RPi.GPIO`

These will have to be installed on the Pi in order for the script to function correctly.

1. In your SSH terminal session, run the following:
    * `sudo pip3 install "python-socketio[client]<5"`: This library is used to communicate with a Tally Arbiter server over websockets.
    * `sudo pip3 install RPi.GPIO`: This is the GPIO library used to communicate with the General Purpose In/Out pins on your Raspberry Pi.

    *If `pip3` is not installed, you can get it by running `sudo apt-get install python3-pip`.*

1. Now that all the necessary libraries are installed and compiled, you need to copy the `tallyarbiter-gpolistener.py` file to your Pi. You can do this a number of ways, but one simple way is to execute this command through your SSH connection: `wget https://raw.githubusercontent.com/josephdadams/TallyArbiter-GPOListener/master/tallyarbiter-gpolistener.py`. This will copy the file into your current folder (you should still be the home folder for the `pi` account).
1. Once the Python script has been copied over, go ahead and test it out to make sure everything is working properly. Run this in the SSH session: `sudo python3 tallyarbiter-gpolistener.py`
    
    Be sure to replace the IP address `192.168.1.6 config_gpo.jspon` with the actual IP of your Tally Arbiter server. If you leave off the port, it will attempt to connect using port `4455`. You can specify another path for `config_gpo.json` if you are running the script from another folder location. If you leave the argument off completely, it will look for a file called `config_gpo.json` in the current working directory.

1. If it is working properly, you will be able to view the newly added GPO listener client in the Tally Arbiter Settings page under the "Listeners" section. Use the "flash" button if you want to see the server communicate with the GPO listener. It will take each GPO pin associated with that device and alternate taking the pin High and Low two times before going back to the previous state of the GPO.

## Setting up the script to start at boot
Now that it is working properly, you will want to set up the script to run on boot so that all you have to do is turn on the Pi and wait for it to launch and connect to the server. There are several different methods in the Raspberry Pi OS to do this. The following describes how to do it using the `rc.local` file.
1. In your SSH session, type `sudo nano /etc/rc.local`.
1. Just before the last line of this file (`exit 0`), add the following: `sudo python3 /home/pi/tallyarbiter-gpolistener.py /home/pi/config_gpo.json &`. The `&` is important because it allows the script to launch in a separate thread since we want the program to continue running in the background.
1. Now reboot the Pi to test that the script runs on boot: `sudo reboot`. This will end your SSH session.

The program should now launch every time the Pi boots up, and automatically connect to your Tally Arbiter server once the server is available.

# Running the software
The software is written in Python but only intended to run on a Raspberry Pi with GPO pins. If you run it on any other platform, you will receive an error.

Upon startup, the program will enumerate through the `config_gpo.json` file and attempt to connect to the specified Tally Arbiter server.

# GPO Configuration
Tally Arbiter GPO Listener uses the built-in GPIO pins of a Raspberry Pi. In order for the program to use the correct pins for your wiring, it must be configured prior to execution. A configuration file, `config_gpo.json` is required to be in the same folder as the listener script.

The `config_gpo.json` file contains two sections:

* `server_config`: The IP and Port of the Tally Arbiter server.
```javascript
 "server_config":
 {
	"ip": "192.168.11.141",
	"port": 4455
 }
```

* `gpo_groups`: The groupings of GPO pins that you want to control. Each GPO Group can be associated with one Tally Arbiter Device.

Example `gpo_group` entry:
```javascript
{
	"id": 1,
	"gpos": [
		{
			"gpoNumber": 1,
			"busType": "preview",
		},
		{
			"gpoNumber": 2,
			"busType": "program"
		}
	],
	"deviceId": "ed34bacd"
}
```

* `id`: A unique identifier.
* `gpos`: The array of GPIO pins assciated in this group. It has the following properties:
	* `gpoNumber`: The actual GPIO Pin number on the Pi.
	* `busType`: Either `preview` or `program`.
* `deviceId`: If configured, the Tally Arbiter Device Id. If this Device Id is invalid or the property does not exist in your config file, Tally Arbiter will automatically reassign this GPO group to the first Device on the server. You can then reassign it to a new Device using the Tally Arbiter interface.

Each GPO Group will be represented as a listener client on the Tally Arbiter server.

Once your configuration file is created and you've made the physical connections to your outboard devices, start up the Tally Arbiter GPO Listener and it will attempt to connect to the Tally Arbiter Server.

# Improvements and Suggestions
I welcome all improvements and suggestions. You can submit issues and pull requests, or contact me through [my blog](http://www.techministry.blog).
