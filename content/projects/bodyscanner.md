+++
title = "Five Seconds to a 3D Body Image"
slug = "bodyscanner"
+++

# A Body Scanner for Medical Compression Stockings 

**Important:** In this story, I have removed all references to those involved in this project. 
It is free of sensitive information or classified data.

## Intro
[Medical compression stockings](https://en.wikipedia.org/wiki/Compression_stockings) help
(among others) especially with edema, a swelling of the leg due to fluid retention in the tissues
or defective venous valves after thrombosis. The goal is to improve the return flow of blood to 
the heart by applying mechanical pressure to the leg veins.  

The natural differences between patients' bodies (some have specialties or additional diseases) require 
the manufacturers of compression stockings to produce in standard size categories but also
custom-made-to-measure stockings. But how many measures are required and who can conduct 
the measurement on a patient's body?  

### Why?
In Germany, most cases of this procedure are done in medical supply stores and pharmacies. It takes 
more than [30 length and circumference measures](https://www.google.com/search?tbm=isch&q=ma%C3%9Fkarte+kompressionsstr%C3%BCmpfe) 
to make up a dimension map of the patient's legs. Yet, it is not trivial to 
determine these measurements by hand, mainly because of the soft nature of the human body's skin.
In addition, people's leg circumferences vary quite a bit throughout the day (you may not be able to see it, but you can measure it).

The entire measurement procedure is time-consuming (+1 for automation) and close to the
body, which can be uncomfortable for both the patient and the medical staff. As is often the case, 
medical supply stores lack trained personnel to perform the measurements.  
Once the patient's data is collected, the medical staff decides whether 
the measurements reasonably fall within the intervals of the standard product sizes or whether a
customized compression stocking must be ordered.  

Fortunately for the patient, there are a variety of products: several vendors, several product lines, 
colors, special features, and so on (with additional costs). It is not easy to choose the 
right stockings, which is why medical staff must also advise on product selection. 
At least twice a year, patients are asked to order new compression stockings as they lose their 
compression effect after this time.  

### Requirements
Here is what my team and I have been asked to create. In addition to the core requirement of capturing the measures of the patient's body, the 3D scanner **should** also meet the following requirements:
- be very fast (under 30 seconds for the data acquisition)
- be accessible by medical staff (often, they are not very affine to tech devices)
- ideally no moving parts and free from barriers (e.g. no high steps)
- compact dimensions, yet representative in the store
- full automation, one-stop-solution from dimensional data to order
- comparable pricing

## Feasibility Study and Prototype
The first project goal was to perform a feasibility study and report the results. We researched
available sensors, based on [structured light](https://en.wikipedia.org/wiki/Structured_light) and [ToF (time of flight)](https://en.wikipedia.org/wiki/Time_of_flight) technologies. At that time,
the sensor of Microsoft's Kinect (One) was already broadly available, but it was discontinued in favor of its successor. The general availability of sensors and the price tag were taken into consideration as well, so
we ended up with sensors from [Orbbec](https://www.orbbec.com/).

To build a body scanner with no moving parts, one of the biggest challenges was to align all 6 static sensors to a one-world coordinate system. After reading almost all scientific papers on depth sensor alignment, we developed our own approach based on components of 
[ROS (the Robot Operating System)](https://www.ros.org/), mainly the [ICP ("Iterative closest point")](https://de.wikipedia.org/wiki/Iterative_Closest_Point_Algorithm) algorithm paired with a pre-registered
calibration.

{{ slider(images=["img/bodyscanner/bs4.jpg", "img/bodyscanner/bs2.jpg", "img/bodyscanner/bs3.jpg", "img/bodyscanner/bs6.jpg", "img/bodyscanner/bs5.jpg", "img/bodyscanner/bs7.jpg"], width=800, height=400) }}

A serial capturing of all involved sensors allowed us to create a 3D image of the patient in under 5 seconds
(without the post-processing of the raw data). Finally, the collected data from the body scanner 
prototype was compared with the values from two manual measurements (of the very same person). The results
were promising and within the specified maximum deviations, so we were given the green light for product development.

<img src="/img/bodyscanner/rotating_body.gif" style="margin: auto; display: block;" alt="A rotating body with detected measures" >
<br/> A 3D avatar of the patient with the relevant circumferences.

## The Device
For the development of the device, we collaborated with a professional industrial designer and a manufacturer based in Germany.

We provided the sensor locations, the layout and the specifications for the required infrastructure as input (mainly, the wiring, touch monitor, compute unit and connectors). I managed the process to
agree on the device's design and checked in regularly with other project goals (i.e. budgets and timeline).

### The Body
The device's body is made from high-quality materials, mostly derivates of medium-density fibreboard. There is
a pane of glass for the patient to stand on during the scan.  
The design is very sleek and clean like most medical devices are. Nevertheless, the body shell integrates some little helpers (for example, paper clips for the prescription, and drawers) to support the user's workflow.

{{ slider(images=["img/bodyscanner/bs8.jpg", "img/bodyscanner/bs9.jpg", "img/bodyscanner/bs10.jpg", "img/bodyscanner/bs11.jpg", "img/bodyscanner/bs12.jpg"], width=800, height=400) }}

### The Touch User-Interface
With the help of professional user-experience design, we created a touch-based user interface and workflow 
on a large touchscreen. The process starts with the login screen and a personal password to 
unlock the device. 
There is a card reader to feed in the patient's data from its German health card. A touch on the "Scan" button
runs the body scan. After *five seconds* the scan is done and the patient can dress again.

After reviewing the scan results (the user can make manual adjustments to the automatically determined values), 
the medical staff can
proceed with the product selection and further customization - directly on the device. Finally, the order can be placed on behalf of the patient directly at the manufacturer.

{{ resize_image(path="img/bodyscanner/bs_scan.jpg", width=900, height=550, op="fill") }}

There are plenty of smart additional functions on the device, for example, a patient's track record, patient search, a news page, device settings and more.

The device's UI is realized with [Electron](https://www.electronjs.org/), Web-technologies and Chromium in Kiosk-mode.

### Data-at-Rest Protection
The device ensures that all data-at-rest (i.e. the data stored on the device) is protected against theft. 
The patient database (a [CouchDB replicator](https://couchdb.apache.org/)) is unmounted and encrypted using
[LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) once the user logs out from the device 
(or it is locked automatically after inactivity). That renders a stolen hard drive unreadable 
without the knowledge of a valid user password.  
Once a user unlocks the device by entering a valid password, the CouchDB data container is decrypted and mounted to 
the database process.

### Data Processing Pipeline
The acquired depth data is processed directly on the device ("edge computing"). We created a flexible pipeline
framework and a data container format for the scans. The pipeline not only smoothens the data but also detects
landmarks in the 3D model to automatically suggest the circumferences at the appropriate length. Our proprietary data processing framework was designed to be portable and upgradeable with existing data containers so we can troubleshoot issues after the scan has already been performed.
All acquired data is replicated (using CouchDB replication) to a cloud-based service platform. In the event of a device failure, no data is lost. Keeping the data online allows the user to proceed with the order on a different end device (for example, a tablet computer). That's up to the preferences of the medical staff and the
patient.

## The Web Interface And The Data
The user management for the device also allows access to a web-based platform. The web interface 
and the touchscreen of the device share large parts of the code base, so the appearance and operation are 
the same.  
Since the local data of the body scanner gets replicated (including deletions) into an online database, the web interface can also grant access to the patient's record. That supports more sophisticated workflows of a shared body scanner or an even more convenient consultation about the final product.

## Fleet Management And Customer Support
The device management and online platform comes with a fleet management application for administration. It
incorporates device health monitoring and usage reporting. For the first level of support, device users can share their patient records (most importantly the scans) and even request on-screen support (via screen sharing).
The device connects to the administration platform via a point-to-point encrypted 
[virtual private network](https://en.wikipedia.org/wiki/Virtual_private_network).

### Rollout
Setting up a device at the client's site requires a calibration of the sensors. This is needed due to the 
production-related deviations of the body frame and sensors. In addition, environmental conditions can affect the results. 
Each device gets a serial number that also encodes optional features (for example, the body color), 
the production date, and a counter. Once the system is calibrated and the initial setup procedure is successfully
performed (i.e. creating initial users), the device is ready to scan patients.

### Monitoring
An integrated monitoring solution allows tracking the usage of individual devices in the field. It collects anonymized data about the usage of features, reports scans with metadata and other metrics. This data allows for improving the workflow and the overall experience for both the user and the patient.
