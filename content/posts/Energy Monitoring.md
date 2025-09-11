+++
title = "Real-Time Energy Monitoring"
author = "Abhit"
date = 2025-09-09
tags = ["energy"]
categories = ["projects", "case-studies"]
+++

While working at a renewable energy startup, we set out to answer a key question:

**How can we make electricity use easier to see, cheaper to track, and more useful for both households and utilities?**

Most utilities don’t get detailed, real-time information about how people use electricity. Without this, it’s harder to match renewable energy supply with demand, often leading to costly or polluting backup power. To address this, we built a simple, low-cost IoT prototype that could monitor household electricity use in real time and send the data to the cloud. This data could then be used to manage energy demand better and support a cleaner, more reliable grid.

---

## Hardware prototype

We designed the prototype using:  
- **Arduino microcontrollers** for low-cost computing.  
- **Non-invasive current sensors** (clamp-on CTs) to measure electricity safely without rewiring. 
- A **SIM900 GPRS module** to connect directly over mobile networks, bypassing unreliable Wi-Fi.  

Every 15 minutes, the device measured consumption, batched readings, and pushed them to a cloud server for storage and analysis.  

---

## Measuring Consumption

Electricity consumption data was captured using current transformers, but raw sensor values are noisy. To get a reliable measure, we needed the **RMS current.  

We implemented a simple algorithm in Arduino firmware:  
1. Continuously sample analog current values.  
2. Apply a digital low-pass filter to stabilize around zero.  
3. Square the values, average them, and take the square root.  

This gave us accurate current readings for each 30-second window. Over 15 minutes, the device built up a block of readings to send upstream.  

```cpp
double calcIrms(unsigned int timeout, unsigned int _inPinI) {
   unsigned int NumberofSamples = 0;
   double offsetI, filteredI, sqI, sumI = 0;
   int sampleI;
   unsigned long start = millis();

   offsetI = 512; // center point for ADC

   while ((millis() - start) < timeout) {
      NumberofSamples++;
      sampleI = analogRead(_inPinI);
      offsetI += (sampleI - offsetI) / 1024;  
      filteredI = sampleI - offsetI;
      sqI = filteredI * filteredI;
      sumI += sqI;
   }

   double Irms = (43.47 * (readVcc()/1000.0) / 1024) * sqrt(sumI / NumberofSamples);
   return Irms;
}
```

## Connecting to the Cloud

Collecting data locally wasn’t enough — we needed to **transmit it reliably to the cloud**. That’s where the **SIM900 GPRS module** came in.  

The Arduino spoke to the modem using **AT commands**. The firmware implemented a **step-by-step handshake**:  

1. `AT` → Check module health.  
2. `AT+CGATT?` → Attach to the GPRS network.  
3. `AT+CIPSHUT` → Reset IP session.  
4. `AT+CSTT="airtelgprs.com"` → Set APN for the network.  
5. `AT+CIICR` → Bring up wireless.  
6. `AT+CIFSR` → Get an IP address.  
7. `AT+CIPSTART="TCP", "app.solarfamily.in", "80"` → Connect to server.  
8. `AT+CIPSEND` → Send payload (HTTP GET request).  

Each stage sets a flag when successful. If one step failed, the firmware retried the loop up to two times. This gave the device resilience even with **patchy mobile connections**.  

```cpp
if(flag == 1) {
  gprsSerial.println("AT+CIPSTART=\"TCP\",\"app.solarfamily.in\",\"80\"\r");
  delay(9000);
  flag = search("CONNECT"); // proceed only if connected
}

if(flag == 1) {
  gprsSerial.println("AT+CIPSEND\r");
  gprsSerial.println(cdat); // send data payload
  gprsSerial.write(0x1A);   // end of transmission
  success = 1;
}
```

Once the link was active, the device packaged the readings into a simple **HTTP POST request**. Each request represented a **15-minute block** of data.  

The payload included:  
- A `device_id` for identification.  
- A `block_id` counter for the interval.  
- Comma-separated current readings.  

```
POST http://app.solarfamily.in/api/v1/feed/readings
?device_id=865904027419669&block_id=123&data=0.5,0.6,0.45,0.5
```

This meant that every 15 minutes, the cloud API received fresh data from the field, ready for analysis. If a request failed, the firmware retried — ensuring minimal data loss.  

## Reflections: IoT for Energy Monitoring and Demand-Side Management

What started as a small Arduino prototype turned out to be a powerful lesson in how **IoT can transform energy systems**.  

- With **low-cost sensors** and **mobile connectivity**, we showed that it was possible to get **real-time electricity data** from the field without expensive infrastructure.  
- **Cloud integration** meant this raw data could instantly power **dashboards, analytics, or optimization engines**.  
- Most importantly, it supported the goals of **demand-side management**: shifting consumption to align with renewable energy availability rather than relying on fossil backups. 

This project wasn’t just about code and circuits — it was about bridging the gap between **physical energy systems and digital infrastructure**. It showed that even small IoT prototypes can unlock **smarter, greener grids** and inspire larger-scale innovation.  
