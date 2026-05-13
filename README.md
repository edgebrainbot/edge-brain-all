# edge-brain-all

## 💡 Robot: External Initial
1. Download App: Unitree Go - Embodied AI
2. Connect with App Robot
3. Log in App > Go! > Mode > Damping
4. Return to main > Device > Service Status > sport_model, mcf, pet_go, and gesture_recognition, are all set to disable. 

## 💡 Robot: Communication Setup 
1. Use AnyDesk to connect with Robot's Jetson. If Jetson AP mode is not set, follow steps below to setup AP mode. If has set, enable it by running `sudo systemctl enable jetson-ap.service,` and then restart by `sudo reboot.` If you want to download or update, disable it by running `sudo systemctl disable jetson-ap.service,` and then and then restart by `sudo reboot.`
2. Wireless Connection: Other Netwroks > Jetson IDs > Connect
3. Anydesk > Enter Remote Address > 10.42.0.1 > Connect
4. Password: edge1234 or edge123
5. Terminal: Download and Compile
   * `mkdir -p ~/edge_ws/src,`
   * `cd ~/edge_ws/src`
   * `git clone -b real-dev git@github.com:Charlescai123/edge-brain-go2.git` 
   * First use `cd` to route to the righ path where edge_scripts is included, then run `./edge_scripts/build_ws.sh`,
     
## 💡 Robot: Development Operate 





   * 5) `ros2 run keyboard_input keyboard_input`, 6) `ros2 launch edge_bringup bringup.real.end2end.launch.py`,7) keepboard: `2 > 2` > robot stand 



7) `source ~/unitree_ros2/setup.sh`, 8) `ros2 launch edge_nav_bringup system_real_robot_with_route_planner.launch.py`, and 9) `ros2 launch edge_drl_student drl_student.end2end.launch.py.`
7. Safe termination of robot: keyboard_input terminal: 2 > 2 > 1.

## download and push 
git clone -b real-dev git@github.com:edgebot/edge-brain-go2.git

cd edge_ws
cd src
cd edge_brain_go2
git add. 
git commit -m "update ..."
git push 








# 🚀 Setup Jetson AP Mode (Auto Start at Boot)

This guide configures the Jetson device to automatically start a WiFi hotspot (AP mode) on boot using nmcli and a systemd service. 

> [!NOTE]
> This setup provides a reliable connection between a laptop and the Jetson, enabling the use of remote access tools (e.g., [AnyDesk](https://anydesk.com/en/downloads/linux)) for convenient GUI interaction. It is essential for *on-device debugging* in the network-constrained environments such as *remote* or *field deployments*.

## 1. Check the wireless interface name on your Jetson
```bash
ifconfig
```

Typical interface names look like: `wlP1p1s0`, `wlan0`, etc.


## 2. Create AP Startup Script
```bash
sudo vim /usr/local/bin/start_ap.sh
```

Add the following content:
```bash
#!/bin/bash

# Wait for system and network initialization
sleep 3

# Enable Wi-Fi
nmcli radio wifi on

# Disconnect any existing connection to avoid conflicts
nmcli device disconnect <WIRELESS_INTERFACE> 2>/dev/null

# Start hotspot
nmcli device wifi hotspot ifname <WIRELESS_INTERFACE> ssid <AP_NAME> password <AP_PASSWORD>
```

Replace `<WIRELESS_INTERFACE>`, `<AP_NAME>`, and `<AP_PASSWORD>` with your actual values.

## 3. Make the script executable
```bash
sudo chmod +x /usr/local/bin/start_ap.sh
```

## 4. Create systemd Service
Create a dedicated service:
```bash
sudo vim /etc/systemd/system/jetson-ap.service
```

Paste the following:
```bash
[Unit]
Description=Jetson WiFi Access Point
After=network-online.target
Wants=network-online.target

[Service]
Type=simple
ExecStart=/usr/local/bin/start_ap.sh
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
```

## 5. Enable and Start Service
Reload systemd and enable the service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable jetson-ap.service
sudo systemctl start jetson-ap.service
```

## 6. Debugging (Optional)
Check service status:
```bash
systemctl status jetson-ap.service
```

View the logs:
```bash
journalctl -u jetson-ap.service -f
```
