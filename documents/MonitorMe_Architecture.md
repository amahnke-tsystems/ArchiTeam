# Vision
We provide a comprehensive, reliable and adaptable software architecture for MonitorMe which allows StayHealthy Inc. to fullfil opportunities in the business of hospitals and medical healthcare.

# Use Cases

## 1 Collect and Store patients vital signs
The system has to collect all signs from all monitoring devices, which are:

* Heart rate: every 500ms
* Blood pressure: every hour
* Oxygen level: every 5 seconds
* Blood sugar: every 2 minutes
* Respiration: every second
* ECG: every second
* Body temperature: every 5 minutes
* Sleep status: every 2 minutes

These signs have to be persisted for 24 hours for further investigation of medical personel

## 2 Display latest patient vital signs on Dashboard
As a nurse or doctor I want to see latest vital signs on a Monitoring-Dashboard in the nurse station. It shows signs from all patients in the respective department, the nurse or doctor is responsable for.

## 3 Analyse vitals signs
The system analyzes vital signs from patitents and detects any critical issues which needs immediate attention by a nurse or doctor in order to prevent the patient of serious harms.

## 4 Alert personel
As a nurse or doctor, I want to be informed via the MonitorMe Dashboard and the MonitorMe mobile app immediately when an issue was detected.

## 5 Filter and Review vital signs
As as medical professional I want to review patients vital signs collected in the last 24 hours. I want to filter these signs by time range and type. 

# Architectural overview
The architecture is following an event driven style, so the central components are message brokers (e.g. KAFKA, depicted in gray color in the diagram):

* Vital Signal Event Channels
* Notification Event Channel

The "Vital Signal Event Channels" receives messages from a small component which we have not depicted in the architecture overview. This gets a single signal for one patient and puts a message out to the message broker system, in this case the "Vital Signal Event Channels" system. For each type of signal there is a message topic (queue). The message also contains the patients id.

The second message broker system is "Notification Event Channel". It receives messages from the component "Signal analyzer" which generates messages for this channel if it raises an alert.

The components that subscribes to one of these channels are:

* Patient Signal Collector
* Raw Signal Persister
* MonitorMe Dashboard Service
* MonitoMe App

The "Patient Signal Collector" subscribes to the "Vital Signal Event Channels" and constantly collect all signals for one patient. It holds them in memory in order to have a whole set of recent signals of all type (blood pressure is only measured once an hour). It sends this consolidatd patients data in intervals of 1 second to the "Sign Analyzer" which can now in turn apply the rules to this dataset and therefor analyze the patients health status. The interval is a guess we are making, it could be adjusted according to the actual requirement which has to be made by medical professional.

The "Raw Signal Collector" is just responsible for storing the signals in a database for further use.

The "MonitorMe Dashboard Service" subscribes to the "Vital Signal Event Channels" in order to get the lastes signals without further latency. Historical signal data will be fetched from the db as well as patients data (id, name, room-number etc.) from the patients db. It also subscribes to the "Notification Event Channel" in order to display alerts immediately to nurses.

"MonitorMe App" subscribes to the "Notification Event Channel" in order to alert a doctor as soon as possible on his smartpone.

The last component is the "External Data APIs" which is responsible for providing snapshots to MedicalMe when requested by personel and exposes a secured API for MonitorThem (either as normal API or it implements some push mechanism if needed). 

 
