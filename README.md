# Avi-On-Cooper-HaloHome-Setup-with-MQTT
This documentation goes through setting up any Halo Home lights that are no longer (discontinued) via their halo home API, cooper lighting, or avi-on devices to have Home Assistant see them using MQTT broker.

This is a low level walkthrough which is referencing Oyvindkinsey (https://github.com/oyvindkinsey/avionmqtt) for initial work done to get this to work, along with his additional acknowledgements mentioned.  Also Co-pilot!!!

# Hardware/Software Requirements for using this setup
- The AVI-ON MQTT broker can be installed on any hardware as long as you are able to run linux OS.  There is no real limitation to what you need to run this service as long as you are meeting the OS standards to run the OS. Ultimate solutions would be a rasberry pi with wifi/bluetooth capabilities will work perfect for this use case.
- Bluetooth antenna device connected to your server to pickup your lights if you are not using a device such as a motherboard with an on-board bluetooth antenna.  Alternatively, you can easily purchase a USB bluetooth antenna for very cheap.
- Have your lights setup/configured without your Avi-On app via your smartphone to ensure you have your lights/devices added into your avi-on account.
- Mosquitto Broker within Home Assistant - There is nothing to configure than to ensure the ports are using the default ports.  I am using the MQTT broker for other purposes than just Avi-On, so there is no issue with using the broker for different use cases.

# Information about this guide and devices I am using for this guide
- VM running Linux Ubuntu version 24.04.2 Live Server
- VM has default configuration with basic cores, RAM, and HDD space allocated to operate the Ubuntu OS
- I did purchase a bluetooth antenna (ZEXMTE USB Bluetooth Adapter for PC, Long Range Bluetooth USB Adapter for Windows 11/10, 492FT/150M Bluetooth Dongle 5.1 EDR, Plug & Play for Desktop, Laptop, Printers, Mouse, Speakers…).  But as mentioned previously, any bluetooth adapter should work if you are in need of one.
- There was a lot of issues I ran into which I will try my best to capture the necessary commands to run through from begginging to end.
- I am also running my own local AI assistant in Home Assistant with wyoming satellites.  All i had to do was expose my lights to the AI assistant and I could tell jarvis to turn my lights down to 10, turn them on or off, etc. without having to specify automations beyond having them turn on and off at certain times each day.

# Assumptions
- You have Home Assistant HACS installed on Home Assistant
- You have the Mosquitto Borker installed within your Home Assistant environment
- You have Ubuntu OS installed or similar OS installed on a server or device
- You have access to SSH to run commands within your Ubuntu OS

# Create a Mosquitto Borker user within Home Assistant
The MQTT Broker automatically will use all users within Home Assistant for authentication. So you could use your own user account for this. This is not really secure. Best practice is to create a user account for mqtt which you can use on all your devices.
- Within your Home Assistant frontend/home navigate to the Settings menu
- Click on People > Users
- Click on ADD USER in the lower right corner
- Enter a name, in this example we will use mosquitto
- enter the password and confirm it
- Disable Administrator and local access only
- Click CREATE

# Switch to Root & Install System Dependencies
Start by switching to the root user:
```bash
sudo su
```

Update your system and install essential tools
- python3 - the language AvionMQTT is written in
- mosquitto - this is the MQTT broker needed as the central server that devices (lights) will connect to
- nano - ensures you can easily edit the "settings.yaml" file later.
- libglib2.0-dev ensures Bluepy compiles successfully.
- pkg-config is required for dependency checks.
```bash
apt update
apt install python3 python3-venv python3-pip git mosquitto mosquitto-clients bluez libglib2.0-dev pkg-config nano
```

# Clone AvionMQTT Repository
```bash
git clone https://github.com/fusioncha0s/avionmqtt.git
cd avionmqtt
```

# Create & Activate a Virtual Environment
Setting up a Python virtual environment keeps dependencies isolated.
```bash
python3 -m venv avion-env
source avion-env/bin/activate
```

Automate the activation of the virtual environment to ensure updates to python code can still be done. This ensures that every time you start a new terminal session, the virtual environment is activated. If you open a new terminal/SSH connection to the server, the system defaults back to the global Python environment, which may not have the necessary dependencies for AvionMQTT.  This can be done manually everytime you SSH into your server, otherwise you can automate this using the below code.
- Change "YOUR_USERNAME" to your ubuntu profile name located under your home directory
```bash
echo "source /home/YOUR_USERNAME/avionmqtt/avion-env/bin/activate" >> ~/.bashrc
```

Apply the changes to the automation
```bash
source ~/.bashrc
```

# Install AvionMQTT Dependencies
```bash
pip install .
```

If for any reason "pip install ." errors, ensure bluepy is installed correctly by running the following command
```bash
pip install bluepy
```
If that also fails, you can manually build and install bluephy
```bash
git clone https://github.com/IanHarvey/bluepy.git
cd bluepy
python3 setup.py build
python3 setup.py install
```

# Create the Settings.yaml Configuration File
```bash
nano settings.yaml
```

Use the following configuration file.  Some things to note...
## avion section
- Update "YOUREMAIL@google.com" with your avion email address, no parenthises needed
- Update "YOURPASSWORD" with your avion password, no parenthises needed
## mqtt section
- **Host:** Update "YOURIPOFYOURHOMEASSISTANTSERVER" with your home assistant service IP address, no parenthises needed
- **Ports:** No ports need to be defined unless you are using custom ports. If custom ports are used, uncomment the port section within the configuration and provide your port #.
- **Username** Update your username to the specified username you created within your Home Assistant server and which you are planning to use for your MQTT broker in Home Assistant.  Assuming your username is all one word.
- **Password** Update your password to the specified password you created within your Home Assistant server and which you are planning to use for your MQTT broker in Home Assistant.  **you do need to include parenthises around your password on this line**
## devices, groups, and capabilities_overrides sections
- I went with absolute basic with little testing into other arguments that can be added on these lines.  Devices and Groups section utilize brackets "[]" to ensure all devices and groups are added through the MQTT broker and into Home Assistant, I am not looking to limit devices with my setup, so this should obtain everything you have within your Avi-On app if you were to look at your app on your smartphone.

```bash
avion:
  email: YOUREMAIL@google.com #Your avion email, you can confirm by opening the avi-on app on your smart device and review your p>
  password: YOURPASSWORD #Your avion password used in concjuction with your email

mqtt:
  host: YOURIPOFYOURHOMEASSISTANTSERVER #The IP of the mqtt broker that is installed on your Home Assistant server.  You do not need to include any port
  #port: YOUR_CUSTOM_PORT
  username: TESTUSER #The username of your mqtt broker on your Home Assistant server
  password: "YOURPASSWORD" #The password of your mqtt broker on your Home Assitant server

devices:
  import: true
  include: []  # Empty list means all devices are included
  exclude: []
  exclude_in_group: false

groups:
  import: true
  include: []
  exclude: []

single_device: true

capabilities_overrides:
  dimming: {} # This tells AvionMQTT that the device supports dimming, but no additional parameters are set.
    # min_brightness: 1
    # max_brightness: 100
  color_temp: {} # This indicates that the device supports color temperature adjustments, but no override values are provided.
    # min_kelvin: 2700
    # max_kelvin: 6500
```
Ensure the YAML formatting is correct.  If errors appear, fix indentation issues before proceeding
```bash
python -c "import yaml; print(yaml.safe_load(open('settings.yaml')))"
```

## Force previous brightness of lights

**I am currently looking into how to force the previous brightness of lights as turning them on each time turns them to 100%**

# Verify MQTT Broker Status
Ensure you are running as root
```bash
sudo su
```

Then check if Mosquitto is running.  Within the "Active" line you should see it say "active (running)" in green font.
```bash
systemctl status mosquitto
```

If the mosquitto service is not running, start it
```bash
systemctl start mosquitto
```

Enable automatic startup of the mosquitto service on reboot/startup
```bash
systemctl enable mosquitto
```

# Start AvionMQTT
start the AvionMQTT service.  Some things to note...
- If you have a lot of devices/lights, it does take a minute or two to before you can start turning on/off devices as it needs to fully obtain the devices PIDs and associate them via the MQTT Broker and then to your Home Assistant.
- "--log=DEBUG" - is not needed to start the AvionMQTT service, but it provides more detailed output of what devices connect and don't connect.  If you did not include this in your code, you will get basic details to ensure your MQTT is running and what devices maynote connect.
- The mqtt broker will first attempt to connect to the MQTT broker and mesh.  It will then attempt to scan and register your devices. If debug is included in your start command, you will get an output of each device in deatil including each device's information such as PIDs, MAC Adddresses, Aliases, etc.
- Turning on or off a device through your Avi-On app on your smartphone or within Home Assistant will output in real-time those changes being made within your Ubuntu server via the debug outputs 

```bash
python -m avionmqtt -s settings.yaml --log=DEBUG
```

Press CTRL + C to exit the service, this will end the connection with the MQTT broker and Home Assistant.  You must run the previous command to start the service again to interact with your lights.

# Make AvionMQTT Start at Boot
Start by switching to the root user
```bash
sudo su
```

Then create a systemd service for automatic startup
```bash
nano /etc/systemd/system/avionmqtt.service
```

Add the configuration
```bash
[Unit]
Description=AvionMQTT Service
After=network.target

[Service]
User=YOUR_USERNAME
WorkingDirectory=/home/YOUR_USERNAME/avionmqtt
ExecStart=/home/YOUR_USERNAME/avionmqtt/avion-env/bin/python -m avionmqtt -s /home/YOUR_USERNAME/avionmqtt/settings.yaml --log=DEBUG
Restart=always

[Install]
WantedBy=multi-user.target
```

Enable the systemd service and start it
```bash
systemctl enable avionmqtt
systemctl start avionmqtt
```

Reboot your ubuntu server and ensure the services start
```bash
sudo reboot
```
```bash
systemctl status avionmqtt
```

# Confirm Home Assistant Integration
After reboot, check that lights are loaded
- Open Home Assistant > Devices
- Open MQTT device
- Open either the devices or entities list
- If you opened devices list, open the "Avi-on MQTT Bridge"
- You should see all your connected lights and should be able to interact with them
