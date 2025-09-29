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

We built the device to demonstrate how real-time household electricity monitoring can be implemented. It combined a few low-cost components:

- **Arduino microcontroller** for basic computing and control.
- **Clamp-on current sensor** (based on a current transformer) to measure electricity safely without any rewiring.
- **SIM900 GPRS module** to send data over mobile networks to the cloud.

The device measured household energy consumption and pushed the readings to a cloud server for storage and analysis. Together, these components created a compact unit that could track electricity use in real-time and transmit the data reliably.

<p id="fig-arduino" align="left">
  <img src="/images/circuit_image2.png" alt="IoT components" width="98%">
  <br>
  <em>Figure 1: IoT device for measuring energy consumption</em>
</p>

## Measuring consumption

Current transformers are sensitive enough to capture every tiny fluctuation in the signal, including those that don’t affect energy usage. To turn that noisy stream into a reliable measurement of actual power, we calculated the RMS [(root mean square)](https://en.wikipedia.org/wiki/Root_mean_square) current and used a digital filter. In the Arduino firmware, we implemented a simple algorithm that continuously sampled analog current values, applied the low-pass filter, and computed the RMS. This algorithm produced a reliable current reading for each 30-second interval. Over the course of 15 minutes, the device collected a block of 30 readings, ready to be sent upstream for storage and analysis.

## Connecting to the Cloud

To make the data available for storage and analysis, we needed a way to get it online. So, we added a SIM900 GPRS module, which allowed the device to send data over mobile networks, much like sending a text message from a phone. The Arduino issued a series of instructions to set up the connection, including checking the modem, obtaining an IP address, and connecting to our server. With that link in place, the device pushed data to the cloud every 15 minutes. If the network dropped out, it automatically reconnected, keeping the system resilient even under weak signal conditions.

The Arduino communicated with the modem using simple text-based instructions known as [**AT commands**](https://en.wikipedia.org/wiki/Hayes_AT_command_set). Each command handled one step in setting up the mobile connection to the cloud server. Here are some example commands:

- `AT` → Checks if the modem is responsive  
- `AT+CIPSTART="TCP","app.solarfamily","80"` → Opens a connection to cloud server  
- `AT+CIPSEND` → Sends the data payload  

We chained these commands into a handshake sequence, a step-by-step conversation between the Arduino and the cloud to confirm the connection. Whenever the network dropped, the device repeated the sequence to keep data flowing reliably.

```cpp
// Device-Cloud handshake sequence
if(flag == 1) {
  gprsSerial.println("AT+CIPSTART=\"TCP\",\"app.solarfamily.in\",\"80\"\r");
  delay(9000);              
 flag = search("CONNECT"); // Proceed only if connected
}
 ...

if(flag == 1) {
  gprsSerial.println("AT+CIPSEND\r");
  gprsSerial.println(cdat); // Send data payload
  gprsSerial.write(0x1A); // End of transmission
 success = 1;
}
```

Once connected, the device packaged the readings into a simple HTTP POST request. Each payload contained a device ID to identify the device, a block ID to mark the time interval, and a list of comma-separated current readings. The cloud server exposed a [RESTful](https://en.wikipedia.org/wiki/REST) API endpoint to receive the data.

## Summary

Through the development of this prototype, we showed that:

- Affordable sensors and mobile connectivity provide access to real-time electricity data.
- Cloud integration transforms raw readings into valuable insights through analytics.
- [Demand-side management](/posts/shared-electric/demand-side-management) becomes possible, enabling users to  [shift consumption](/posts/shared-electric/load-shifting) to match renewable energy availability.

It was an essential step toward making energy systems more transparent and accessible. By linking everyday devices to digital platforms, we can empower both households and utilities to build cleaner and more resilient grids.

> With real-time insights, households become active players in building a cleaner grid.