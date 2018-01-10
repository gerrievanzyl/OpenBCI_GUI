# OpenBCI Marker Documentation 
The OpenBCI stack supports the ability to insert subject simulus markers into the EEG data stream to facilitate subsequent processing. Currently the system supports two different mechanisms to insert markers (optosensor and UDP markers), each mechanism has advantages and disadvantages and the choice will depend on the application.  

This document provides an overview of the two mechanisms and provides documentation of the UDP marker mechanism

## Optosensor Markers
An optosensor is attached to one of the corners of the subject stimulus presenation screen.  The output of the optosensor is connected to a pin on the OpenBCI board (Cyton, Cyton and Ganglion, or all boards?).  The stimulus presentation software is configured to show a white square on the corresponding corner of the screen at the moment a marker should be inserted into the data stream. The OpenBCI board detects the change in light intensity and inserts a marker into the AUX channels of the data stream.  

The primary advantage of his mechanism is the precise timing of the markers with the presentation of the stimulus.  It overcomes any delays (or more importantly jitter) in the communication between the stimulus presentation software and the OpenBCI board as well as the monitor's sync delays and jitter which can be as big as 33ms at 30Hz refresh rates.  

The primary disadvantages of this mechanism is that only timing information is included in the data stream. The sequence of the stimuli presented to subjects must be recorded separately and later combined with the EEG data. 

The very accurate timing of this marker mechanism makes it suitable for ERP studies where the timing of the potentials is critical. 

## UDP Makers
In this mechanism, the stimulus presentation software sends a UDP message with a marker number to the OpenBCI_GUI software. The OpenBCI_GUI software forwards this marker to the OpenBCI board over the communication channel (Bluetooth or Wi-Fi). The OpenBCI board receives this marker and inserts the marker value into the data stream in the AUX1 channel.  

The advantage of this mechanism is that complex studies can be run using any stimulus presentation software (we recommend the python-based SNAP presentation software available from SCCN) and the nature and timing of each stimulus will be recorded in the datastream.  This greatly simplifies the subsequent analysis and allows easy automation of preprocessing.  

The disadvantage is that there is jitter and delays between the time the software presents a stimulus and the time that the marker is inserted in the data steam.  The jitter could be as much as 40ms.  This is likely to make this mechanism unsuitable for ERP studies.  

The richness of the UDP marker information and the ease at which it can be integrated into stimulus presentation software makes it very useful in studies that do not require millisecond precision timing, like motion imagery studies.

# Overview of UDP markers  
An overview of the UDP marker mechanism is illustrated below:
<p align="center">
  <img alt="banner" src="/images/OpenBCI_UDP_Marker_Overview.png/" width="600">
</p>

1. The stimulus presenation software presents a stimulus to the subject and at the same time sends a marker over UDP to port 51000 of the computer running the OpenBCI_GUI software.  The UDP message is "MARKn" where n is an integer 0<n<96.
1. The OpenBCI_GUI receives this marker and sends this marker to the OpenBCI board over the bluetooth or wifi communication channel.  The packet sent by OpenBCI_GUI is "\`x" where x is the ascii character char(n+31)
1. The OpenBCI Board receives the marker and inserts it into the EEG data stream.  The marker is saved with the EEG data to the SD card (if that option was selected) 
1. Finally, the data packet is transmitted back to the OpenBCI_GUI where it is displayed and optionally saved.  

*Note: Currently the simulus presentation software and the OpenBCI_GUI software must run on the same computer.  The reason for this is that the UDP listener on in the OpenBCI GUI will only respond to messages sent from localhost (127.0.0.1).  This limitation can be removed by editing the UDP setup in OpenBCI_GUI.pde

# Configuring the Stimulus Presentation Software
Most stimulus presentation software allows for markers to be sent over UDP.  Configure the software to send markers to Port 51000 on localhost. The format for marker messages is:  

MARKnn

where "*MARK*" is uppercase identifier and *nn* is an integer between 1 and 95 (inclusive)

# The OpenBCI_GUI Marker Widget
The Marker Widget is used to configure the different parts of the system and to monitor the markers during a study.  To use this widget perform the following steps before starting the data stream:
1.  Open the widget and click on the "Marker Mode" button.  This sends a command to the board to put the board into Marker Mode.  In Marker Mode the accelerometer data is not recorded or transmitted back and the value of the marker will be saved in the AUX1 channel.
1.  As each marker is received, the widget will plot the value of the maker on the timeline and the value of the last marker will be indicated above the chart.


