FIRST STEP
Make sure you can read data from a physical sensor at the system level, and modify data acquisition script accordingly
Fill in correctly the file .config, and run the install.sh with root privilege.

DESCRIPTION

We present a new data acquisition system for sensor observations.
The  objective  of  this  system  is  to  read  data  from  a  sensor  attached  to  a  remote smart device, like a the RPI, and to store the data on an online SOS server.
The intent was to create an easy to configure and to maintain system, capable of managing data in a simple way and to offer the data in the philosophy of Open Data.
Affordable  Remote  Terminal  for  Sensor  Observation  System  (ART4SOS)  is  a software  solution  that  runs  on  Unix-like  systems.   
It  takes  sensor  data  and  puts them  on  an  SOS  server  as  observations.  
The  data  comes  from  a  physical  sensor,the DS18B20 sensor, that  is  recognized  at  system  level.  
The core task is to handle data conversion from a raw reading of a sensor to a SOS compatible data format and to transmit the data to a SOS server.
According to the SOS specs version 2.0, the common way to insert new data on a SOS server is by using the insertObservation service. 
It require a http post call with a data packet - tipically in JSON or SOAP. 


SENSOR

We focus to a simple case of data acquisition from physical sensor. We show how a temperature sensor is configured on the RPI and how to retrieve data. The sensor DS18B20 from the Maxim Integrated Products Inc. is a programmable resolution 1-Wire digital thermometer. The 1-Wire communication protocol is a communication bus, also knows as MicroLAN, from Dallas Semiconductor/Maxim that implement a half duplex transmission channel over two wires, the data wire and  the  ground. It forms a network of devices  using a single master multi  slave scheme. The RPI acts as master and the thermal sensor as slave. It is possible to attach other slave devices in parallel over the same data wire. On the master end, the RPI is able to handle the communication with each one of the slaves. The first step in configuring the sensor is to add the line dtoverlay = w1-gpio, gpiopin = N, where N is the desired data communication pin number on the GPIO header, to the /boot/config.txt file. After rebooting, the command modprobe w1-gpio and modprobe w1 therm will load the appropriate kernel modules to handle either the GPIO and the 1-Wire communication protocol. A directory named after the sensor serial number appears in the path /sys/bus/w1/devices containing the file w1 slave that will give access to the current value read from the sensor.
A raw reading of the w1 slave file, e.g. through the cat command, gives two text lines containing a CRC code and the current temperature value expressed in thousands degree over the Celsius scale



For install this system, you must edit manually the file of configuration "art4sos.conf".
"art4sos.conf" is a text file used for the configuration of the service parameters via a variable-value pairs.  It includes various necessary fields such as the name and the coordinates of the acquisition station, that is the feature of interest according  to  the  SOS  model.   The  spacial  definition  follows  the  sampling  point standard and it is expressed in latitude, longitude, altitude and projection values. This static spacial definition is needed because this version of the system does not have  a  GPS  installed.   It  is  simple  to  extend  the  system  to  make  use  of  a  GPS but this case is out of the scope of this thesis.  We use the PATH variable as the mounting point of a RAM partition,  and then the place where the produced files are buffered locally.  The configuration includes the SOS service interface, which is a accessible Internet URL, on which the data will be loaded. 


The file "art4sosconfigure.py" is a Python script that prepare the SOS server to receive the remote observations. It uses the InsertObservation service, it takes the local parameters from the above configuration file to fill a standard JSON form for the InsertObservation service and makes the actual call to the SOS service. The used JSON package is also locally archived. Once this configuration is accomplished, the service is ready for automatic data entry.


Data Acquisition

The Python script "art4sosacquisition.py" make use of an appropriate library to handle a thermal sensor via the 1-wire micro-lan.  This module is voluntary left simple for sake of clarity as it is simple to be adapted to common IoT device via system support.

Data Format

The file "art4sosformat.py" is a Python script that takes a numeric value in INPUT and produces a JSON package that is compatible to the SOS InsertObservation service.  It use the system time and the configuration values to compose a JSON package starting from a model.  The JSON data file is saved to the disk.

Data Transmission

The Python Script "art4sostransmission.py" looks for local buffered observation files
and it try to transmit each of them on the server.  If the transmission is successfully
performed, it removes the transmitted observation from the local terminal, otherwise
the observation file is left to be later transmitted.  Then, it analyzes the answer of
the  server,  if  it  has  status  ”http  200  OK”  and  it  does  not  contain  any  exception then the transmission was successful and the observation correctly inserted on the data set.  The source code is reported in what follows


"install.sh" is a Bash script to install ART4SOS service.
It assumes that the ART4SOS configuration file has been properly set. It firstly call the ART4SOS configuration script in order to define the observation attributes on the SOS server.  It creates a directory for locally buffering the observation data files.  This directory is used as mounting point of a RAM disk that is defined at the system level via the fstab file. For time synchronization purpose, due to the lack of a Real Time Clock on board of the RPI, a job is defined to be run at every boot by the CRON management with root privilege.   The CRON management system is also used to set periodic run of the ART4SOS data acquisition and data format scripts, in a single system call combining the value through the system pipe.  A further system call is used to periodically run the ART4SOS transmission script.






