+++
title = "Real-Time Energy Monitoring"
author = "Abhit"
date = 2016-01-16
tags = ["energy"]
categories = ["projects", "shared-electric"]
+++

Modern energy grids are getting smarter, yet they still lack a key aspect: real-time data on household electricity use. Without it, much of the renewable energy ends up wasted, and the grid has to rely on fossil fuels during peak hours. While working at [Shared Electric GmbH](https://sharedelectric.com/), a renewable energy startup, [Shaaz](https://www.shaazahm.com/) and I built a lightweight IoT device to make electricity monitoring accessible and affordable for households and utilities.
<!--more--> 

## Design

We built a working prototype to demonstrate how real-time household electricity monitoring can be implemented. The device combined a few low-cost components:

- **Arduino microcontroller** for basic computing and control.
- **Clamp-on current sensor** (based on current transformer) to measure electricity safely without any rewiring.
- **SIM900 GPRS module** to send data over mobile networks, avoiding the need for unstable WiFi connections.

The device measured household energy consumption and pushed the readings to a cloud server for storage and analysis. Together, these parts created a compact unit that could track electricity use in real time and transmit the data reliably.

<p id="fig-arduino" align="left">
  <img src="/images/circuit_image2.png" alt="IoT components" width="98%">
  <br>
  <em>Figure 1: IoT device for measuring energy consumption</em>
</p>

## Measuring consumption

Current transformers are sensitive enough to capture every slight fluctuation in a signal. That level of detail is valuable, but it makes accuracy harder to achieve. To filter out the noise and measure actual consumption, we calculated the RMS (root mean square) current. For this, we implemented a simple algorithm in the Arduino firmware. It continuously samples analog current values, applies a digital low-pass filter to stabilize the signal around zero, and then squares the values, averages them, and takes the square root. This approach produced reliable current readings for each 30-second interval. Over the course of 15 minutes, the device collected a block of 30 readings, ready to be sent upstream for storage and analysis.

## Connecting to the Cloud

To make the data available for storage and analysis, we needed a way to get it online. Storing it on the Arduino was not enough. So, we added a SIM900 GPRS module, which allowed the device to send data over mobile networks, much like sending a text message from a phone. The Arduino issued a series of instructions to set up the connection: check the modem, log in to the mobile network, obtain an IP address, and connect to our server. With that link in place, the device pushed data to the cloud every 15 minutes. If the network dropped out, it automatically reconnected, keeping the system resilient even under weak signal conditions.

The Arduino communicated with the modem using simple text-based instructions known as **AT commands**. Each command carried out a specific step in establishing the mobile connection. For example:

- `AT` → Check if the modem is responsive  
- `AT+CGATT?` → Attach to the mobile network  
- `AT+CIPSTART="TCP","app.solarfamily","80"` → Open a connection to our server  
- `AT+CIPSEND` → Send the data payload  

The firmware chained these commands into a handshake sequence and retried them whenever the signal dropped. This ensured the device could continue sending data to the cloud reliably, even in poor network conditions.

```cpp
if(flag == 1) {
  gprsSerial.println("AT+CIPSTART=\"TCP\",\"app.solarfamily.in\",\"80\"\r");
  delay(9000);
  flag = search("CONNECT"); // proceed only if connected
}
 ...

if(flag == 1) {
  gprsSerial.println("AT+CIPSEND\r");
  gprsSerial.println(cdat); // send data payload
  gprsSerial.write(0x1A);   // end of transmission
  success = 1;
}
```

Once connected, the device packaged its readings into a simple HTTP POST request. Each payload contained a `device_id` to identify the device, a `block_id` to mark the interval, and a list of comma-separated current readings. If a request failed, the firmware retried until it succeeded, minimizing data loss and keeping the cloud API consistently updated with field data for analysis.

## Reflections

The prototype demonstrated the potential of IoT to make energy systems smarter and more efficient. Here are some of the ways it created value:

- Affordable sensors and mobile connectivity can provide access to real-time electricity data.
- Cloud integration turns raw readings into valuable insights through dashboards and analytics.
- It creates opportunities for [demand-side management](/posts/shared-electric/demand-side-management), where users can [shift consumption](/posts/shared-electric/load-shifting) to align with renewable energy availability.  

This was an essential step toward making energy systems more transparent and accessible. By linking everyday devices to digital platforms, we can empower both households and utilities to build cleaner and more resilient grids.

> With real-time insights, households become active players in building a cleaner grid.
