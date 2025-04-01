# Avi-On-Cooper-HaloHome-Setup-with-MQTT
This documentation goes through setting up any discontinued Halo Home lights, cooper lighting, or avi-on devices to have Home Assistant see them using MQTT broker.

Referencing Oyvindkinsey (https://github.com/oyvindkinsey/avionmqtt) for initial work done to get this to work along with his additional acknowledgements mentioned.

# Hardware/Software Requirements for using the Avi-On MQTT service
- The AVI-ON MQTT broker can be installed on any hardware as long as you are able to run linux OS.  There is no real limitation to what you need to run this service as long as you are meeting the OS standards to run the OS. Ultimate solutions would be a rasberry pi with wifi/bluetooth capabilities will work perfect for this use case.
- Bluetooth antenna device connected to your server to pickup your lights if you are not using a device such as a motherboard with an on-board bluetooth antenna.  Alternatively, you can easily purchase a USB bluetooth antenna for very cheap.
- Have your lights setup/configured without your Avi-On app via your smartphone to ensure you have your lights/devices added into your avi-on account.
- Home Assistant using HACS - This allows me to add the home assistant MQTT add-on needed to add devices into HA
- Mosquitto Broker within Home Assistant - There is nothing to configure than to ensure the ports are using the default ports.  I am using the MQTT broker for other purposes than just Avi-On, so there is no issue with using the broker for different use cases.

# Information about this guide and devices I am using for this guide
- VM running Linux Ubuntu version 24.04.2 Live Server
- VM has default configuration with basic cores, RAM, and HDD space allocated to operate the Ubuntu OS
- I did purchase a bluetooth antenna (ZEXMTE USB Bluetooth Adapter for PC, Long Range Bluetooth USB Adapter for Windows 11/10, 492FT/150M Bluetooth Dongle 5.1 EDR, Plug & Play for Desktop, Laptop, Printers, Mouse, Speakers…).  But as mentioned previously, any bluetooth adapter should work if you are in need of one.
- There was a lot of issues I ran into which I will try my best to capture the necessary commands to run through from begginging to end.

# Assumptions
- You have Home Assistant HACS installed on Home Assistant
- You have the Mosquitto Borker installed within your Home Assistant environment
- You have Ubuntu OS installed or similar OS installed on a server or device
- You have access to SSH to run commands within your Ubuntu OS

# Install the bluetooth
Change to super user
```bash
su
```
