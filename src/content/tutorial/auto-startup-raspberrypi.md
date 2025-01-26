---
title: "How to Create an Auto-Startup Script on a Raspberry Pi"
meta_title: ""
description: "Creating an Auto-Startup Script on a Raspberry Pi"
date: 2022-12-08T05:00:00Z
image: "/images/image-placeholder.png"
categories: ["How-to"]
author: "Solomon Ankomah"
tags: ["Raspberry Pi"]
draft: false
---


Setting up a Raspberry Pi to run a program automatically on startup is a common requirement for projects like IoT devices, media servers, or kiosks. This guide walks you through different methods to create an auto-startup script on a Raspberry Pi running Raspberry Pi OS (formerly Raspbian).

---

### Prerequisites

Before getting started, ensure you have the following:

1. A Raspberry Pi running Raspberry Pi OS.
2. Access to the Raspberry Pi via SSH or a connected monitor and keyboard.
3. A program or script (e.g., Python, shell script) that you want to run on startup.
4. Basic familiarity with the Linux command line.

---

### Method 1: Using `rc.local`

`rc.local` is a legacy initialization file that runs commands at the end of the boot process.

#### Steps:

1. Open the `rc.local` file for editing:
   ```bash
   sudo nano /etc/rc.local
   ```

2. Add the command to run your script before the `exit 0` line:
   ```bash
   /usr/bin/python3 /home/pi/my_script.py &
   ```
   - Replace `/usr/bin/python3` with the interpreter for your script (e.g., Python).
   - Replace `/home/pi/my_script.py` with the full path to your script.
   - The `&` at the end runs the script in the background.

3. Save the file and exit (Ctrl+O, Enter, Ctrl+X).

4. Make the script executable (if not already):
   ```bash
   chmod +x /home/pi/my_script.py
   ```

5. Reboot to test:
   ```bash
   sudo reboot
   ```

---

### Method 2: Using `cron`

The `cron` scheduler can be used to run tasks at specific times, including system startup.

#### Steps:

1. Edit the `crontab` file:
   ```bash
   crontab -e
   ```

2. Add the following line to run your script at boot:
   ```bash
   @reboot /usr/bin/python3 /home/pi/my_script.py
   ```

3. Save and exit the editor (e.g., for `nano`, Ctrl+O, Enter, Ctrl+X).

4. Make the script executable (if not already):
   ```bash
   chmod +x /home/pi/my_script.py
   ```

5. Reboot to test:
   ```bash
   sudo reboot
   ```

---

### Method 3: Using `systemd`

`systemd` is the default init system on Raspberry Pi OS. It provides more robust control over services and startup scripts.

#### Steps:

1. Create a new service file:
   ```bash
   sudo nano /etc/systemd/system/my_script.service
   ```

2. Add the following configuration:
   ```ini
   [Unit]
   Description=My Startup Script
   After=network.target

   [Service]
   ExecStart=/usr/bin/python3 /home/pi/my_script.py
   Restart=always
   User=pi
   
   [Install]
   WantedBy=multi-user.target
   ```

   - Replace `/usr/bin/python3` and `/home/pi/my_script.py` with the appropriate paths.
   - `Restart=always` ensures the script restarts if it crashes.

3. Save and exit (Ctrl+O, Enter, Ctrl+X).

4. Enable the service:
   ```bash
   sudo systemctl enable my_script.service
   ```

5. Start the service immediately (optional):
   ```bash
   sudo systemctl start my_script.service
   ```

6. Check the service status:
   ```bash
   sudo systemctl status my_script.service
   ```

7. Reboot to confirm it runs on startup:
   ```bash
   sudo reboot
   ```

---

### Method 4: Using the Desktop Environment (For GUI Applications)

If you’re running a graphical interface and want to start a program when the desktop environment loads:

#### Steps:

1. Open the autostart configuration:
   ```bash
   nano ~/.config/lxsession/LXDE-pi/autostart
   ```

2. Add your command at the end of the file:
   ```bash
   @/usr/bin/python3 /home/pi/my_script.py
   ```

3. Save and exit (Ctrl+O, Enter, Ctrl+X).

4. Reboot to test:
   ```bash
   sudo reboot
   ```

---

### Debugging Tips

- **Check Logs**: If the script doesn’t run as expected, check the system logs:
  ```bash
  journalctl -xe
  ```

- **Script Permissions**: Ensure the script is executable:
  ```bash
  chmod +x /path/to/your_script
  ```

- **Script Path**: Use absolute paths for all files and commands in your script.

- **Dependencies**: Ensure all required libraries or dependencies are installed and accessible to the user running the script.

---

### Conclusion

With these methods, you can easily configure your Raspberry Pi to run a script automatically on startup. Choose the method that best fits your project requirements, and enjoy the convenience of having your Raspberry Pi ready to go as soon as it powers on.

