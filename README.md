# Edge Brain: All Basic Operations

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
   * `mkdir -p ~/edge_ws/src`
   * `cd ~/edge_ws/src`
   * `git clone -b real-dev git@github.com:Charlescai123/edge-brain-go2.git` 
   * First use `cd` to route to the righ path where edge_scripts is included, then run `./edge_scripts/build_ws.sh`
     
## 💡 Robot: Communication Reset When Change Ethernet Line
1. Log in Jetson > Netwrok > Ethernet Connected > Wired Setting > Wired (Setting logo) > IPv4 Input: Address: `192.168.123.10019` and Network: `255.255.255.0`, and choose `Mannual`
2. Check Robot Moter's Communication: Terminal > `ping 192.168.123.161`
 
## 💡 Robot: Development Run
1. Check topic list: Terminal > `ros2 ropic list` 
   * If no or only a few, Run `ros2 daemon stop`, then run `ros2 daemon start`, then run `source ~/unitree_ros2/setup.sh`
   * Check `ros2 ropic list`, if still no full list, go to previous step, repeat, untill have full list. 
1. Terminal > `ros2 run keyboard_input keyboard_input`
2. Terminal > `ros2 launch edge_bringup bringup.real.end2end.launch.py`
3. Terminal keyboard: `2 > 2` > robot stand
4. Terminal > `source ~/unitree_ros2/setup.sh` + `ros2 launch edge_nav_bringup system_real_robot_with_route_planner.launch.py`
5. Terminal > `ros2 launch edge_drl_student drl_student.end2end.launch.py`
6. Terminal > keyboard > 4 --> RUN!
7. Safe termination of robot running: Terminal keyboad: `2 > 2 > 1`

## 💡 Download and Update
1. Download: Terminal > `git clone -b real-dev git@github.com:edgebot/edge-brain-go2.git`
2. Update: Route to the right path
   * `cd edge_ws'
   * `cd src'
   * `cd edge_brain_go2`
   * `git add.'
   * `git commit -m "update ..."`
   * `git push` 

## 💡 Other Common Commands
1. Check graph: `rqt_graph`
2. Check topic content: `ros2 topic echo /registered_scan`
3. Check topic frequency: `ros2 topic hz /registered_scan `






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
