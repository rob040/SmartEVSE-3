# LCD Screen Buttons control
### Start/Stop Functionality via LCD Screen Buttons

* Pressing the "O" button for longer than 2 seconds will open the Menu screen.
* Pressing the "<" button for less than 2 seconds will toggle between Smart and Solar modes.
* Pressing the "<" button for longer than 2 seconds will deny access, setting the mode to "Off" and stopping charging.
* Pressing the ">" button for longer than 2 seconds will grant access, activating the previously set mode and resuming charging.
* Simultaneously pressing both "<" and ">" buttons will refresh the LCD screen.

### Locking the LCD Screen Buttons

The Buttons below the LCD screen can be locked/unlocked.
This may be usefull when unauthorized local control is a possibility.
* By pressing both the "O" and ">" buttons at power-up, or
* By checking/unchecking the LCDLock box in the webserver screen.
There is no indication on the LCD when the lock is active.

---

# Webserver Features

Once your Wi-Fi parameters are configured, your SmartEVSE will be accessible on your local network through a built-in webserver. Access the webserver via:

* `http://<ip-address>/`
* `http://smartevse-xxxx.local/` where `xxxx` is the serial number of your SmartEVSE (found on a sticker at the bottom). Ensure mDNS is configured on your LAN.
* `http://smartevse-xxxx.lan/` where `xxxx` is the serial number of your SmartEVSE. mDNS may need to be configured for this.

### Firmware Updates (OTA)

* Navigate to `http://<your-smartevse>/update` or press the "UPDATE" button on the webserver.
* Upload the `firmware.bin` file from this archive, or use `firmware.debug.bin` renamed to `firmware.bin` for a debug version (accessible via Telnet). Ensure the file name is correct; otherwise, flashing will fail.
* In case of failure (FAIL), check your Wi-Fi connection and retry.
* After a successful update (OK), wait 10-30 seconds for the firmware, including the webserver, to go online.

### Wi-Fi Debugging

* If the debug version is flashed, you can access the debugger via Telnet at `http://<your-smartevse>/` to monitor system activity.

### RFID List Uploads (OTA)

* Upload RFID lists via the "update" button or the `/update` endpoint by submitting a file named `rfid.txt`.
* Each line should contain one RFID (NFC) tag UID in hex format (size bytes):
    ```
    112233445566
    0A3B123FFFA0
    ```
* All existing RFID tags are deleted upon upload.
* If Power Share (Master/Slave configuration) is enabled, upload the list to each SmartEVSE device individually to maintain separate lists for each.

---

# Power Share Mode Switching

* When switching the mode on the Master device, the Slaves will automatically switch modes accordingly.
* If you change the mode on a Slave and the Master is not configured with a Smart/Solar toggle switch, the Master and all other Slaves will follow the mode change.
* If a Smart/Solar toggle switch is present, ensure that the Master and all Slaves are set to the same mode. We recommend replacing the toggle switch with a pushbutton switch for ease of use.

---

# Error Messages

If an error occurs, SmartEVSE will stop charging and display one of the following error messages:

* **ERROR: NO SERIAL COM** – No signal has been received from the Sensorbox or another SmartEVSE (used for load balancing) for 11 seconds. Please check the wiring.
* **ERROR: NO CURRENT** – Insufficient current is available to start or maintain charging. The system will retry in 60 seconds.
* **ERROR: HIGH TEMP** – The internal temperature has reached 65°C, stopping charging. Charging will resume once the temperature drops below 55°C.
* **RESIDUAL FAULT CURRENT DETECTED** – A DC Residual Current Monitor has detected a fault, and the Contactor has been switched off. Press any button to reset the error.

---

# Mains meter data

In order to perform the **Smart** charging (take other home consumers into account and prevent electrical
overloads) and **Solar** charging (use surplus power and net zero consumption), the SmartEVSE needs
information about the currents at the mains entry: the Mains-Meter.

This can be accomplished by one of the following methods:
* Sensorbox with current transformer clamps on the mains entry. The sensorbox is connected to SmartEVSE via RS485 modbus plus 12V power.
* Use a dedicated energy/power (kWh) meter with modbus interface at the mains entry.
  See [this support list](installation.md#supported-modbus-kwh-meters) for your options.
* Smart meter DSMR P1 port to Sensorbox2 or compatible products, connected to SmartEVSE via RS485 modbus plus 12V power.
* Smart meter DSMR P1 port to SlimmeLezer or ESPhome DSMR or other (open source-) products, connected over Wi-Fi
* Smart meter DSMR P1 port data obtained from home automation system, like [Home Assistant](https://www.home-assistant.io/), send to SmartEVSE over WiFi.

  The integration with third-party home automation systems allow you to:
    * Change the charging mode.
    * Override the charge current.
    * Pass current measurements (e.g., p1, battery) without additional hardware.
    * Switch between single- and three-phase power (requires an extra 2P relay on the C2 connector).

For sending Mains-meter data over WiFi there are two API methods available:
* [REST API](REST_API.md#post-currents)
* [MQTT API](MQTT_API.md)
In any case, the Mains-meter data contains the L1,L2,L3 phase currents that must be send at regular intervals, never later than 10 seconds (there is a 11 second timeout that aborts charging).

If using Mains-meter data API over WiFi, you have to configure the MainsMeter in LCD setup menu to the 'API' option.

# EV meter data

An optional EV energy/power (kWh) meter with modbus interface at the charge output will measure
power and charged energy, and display this on the LCD and web interface.
With this meter included, the Solar charging mode can more accurately decide between charging at 1-Phase or 3-Phases and changing between these.
See [this support list](installation.md#supported-modbus-kwh-meters) for your options.

Alternatively, EV meter data, consisting of L1,L2,L3 phase currents and energy and power,
can be send to the SmartEVSE using one of the two available API methods:
* [REST API](REST_API.md#post-currents)
* [MQTT API](MQTT_API.md)

If using EV-meter data API over WiFi, you have to configure the EV-Meter in LCD setup menu to the 'API' option.

---

# Simple Timer for Delayed Charging

A simple timer for delayed charging is available via the webserver.

* Upon refreshing the webpage, the "StartTime" field (next to the mode buttons) will display the current system time.
* If you press any mode button, charging will start immediately.
* If you set a future "StartTime," a "StopTime" field will appear. If you leave "StopTime" at the default, it will be ignored. Pressing Normal, Solar, or Smart mode will:
    - Register the StartTime.
    - Switch the mode to "Off."
    - Start the charging session at the designated StartTime, either in Normal or Smart mode.
    - Continue the charging session indefinitely.
* Entering a "StopTime" will enable a "Daily" checkbox, allowing the StartTime/StopTime combination to repeat daily starting from the selected date.
* To clear StartTime, StopTime, and Repeat, refresh the webpage and select Normal, Solar, or Smart mode.

### Known Issues

* If the NTP time is not yet synchronized (e.g., after a reboot), results may be unpredictable. Wait until the system time settles.
* If the StopTime is set more than 24 hours after the StartTime, results are untested. Ensure that values make sense.

---

# EU Capacity Rate Limiting

A European Union directive allows electricity providers to charge consumers based on a "capacity rate," encouraging users to balance their energy consumption more evenly and reduce peak usage.

For more information, visit [this link](https://github.com/serkri/SmartEVSE-3/issues/215).

A menu option, "SumMains" is available for Smart or Solar charging mode.
In addition to other limits (Mains, MaxCircuit), the charging current will be restricted to ensure that the total current across all phases does not exceed the SumMains setting.

SumMains has a default setting of 600A.
If you are unsure how to configure this, it is recommended to leave the setting at its default value.

At time of writing, <ul>Capacity Rate Limiting</ul> is employed in **Belgium**, but not in the Netherlands, or Germany.

