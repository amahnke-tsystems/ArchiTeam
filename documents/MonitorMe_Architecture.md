# Baseline
StayHealthy, Inc. is a large and highly successful medical software company based in California, US.
Their most known and popular products are cloud-based, MonitorThem as data analytics platform, and MyMedicalData as patient medical records used by health professionals.

# Business Goals
StayHealthy Inc. would like to provide a new patient monitoring system for hospitals that monitors patient vital signs. The provided solution also contains the proprietary monitoring devices built by StayHealthy, Inc.

# Vision
We provide a comprehensive, reliable and adaptable software architecture for MonitorMe which allows StayHealthy Inc. to fullfil opportunities in the business of hospitals and medical healthcare.

# Context
The MonitorMe system has some relationships to hardware and software already in stock at StayHealthy, Inc. This context is depicted here:

![Context](../diagrams/kata_actors_components_JA_001.png)

# Use Cases

## 1 Collect and Store patients vital signs
The system has to collect all signs from all monitoring devices, which are:

* Heart rate: every 0.5s (2 times per second)
* Blood pressure: every 3600s (1 hour)
* Oxygen level: every 5s
* Blood sugar: every 120s
* Respiration: every 1s
* ECG: every 1s
* Body temperature: every 300s (5 minutes)
* Sleep status: every 120s

These signs have to be persisted for 24 hours for further investigation of medical personel. 

## 2 Display latest patient vital signs on Dashboard
As a nurse or doctor I want to see latest vital signs on a Monitoring-Dashboard in the nurse station. It shows signs from all patients in the respective department, the nurse or doctor is responsible for. 

A sequence diagram of the interaction of the involved components from use case 1 and 2 is here:

![Diagram](../diagrams/comm_diag_UC_collect_and_display_signals.png)

## 3 Analyse vitals signs
The system analyses vital signs from patitents and detects any critical issues which needs immediate attention by a nurse or doctor in order to prevent the patient of serious life state change.

## 4 Alert personel
As a nurse or doctor, I want to be informed via the MonitorMe Dashboard and the MonitorMe mobile app immediately when an issue was detected by the vital sign analyser. 

A sequence diagram of the interaction of the involved components from use case 3 and 4 is here:

![Diagram](../diagrams/comm_diag_UC_analyse_and_alert.png)

## 5 Filter and Review vital signs
As as medical professional I want to review patients vital signs collected in the last 24 hours. I want to filter these signs by time range and type. As per request I want to be able to upload and store the snapshot related to the patient's health records in MyMedicalData.

# Architectural overview
The architecture is following an event driven style, so the central components are message brokers (e.g. KAFKA). These are depicted in gray color in the overview diagram:

![Diagram](../diagrams/architecture_overview.png):

The two message broker systems are:

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

The "External Data APIs" which is responsible for providing snapshots to MedicalMe when requested by personel and exposes a secured API for MonitorThem (either as normal API or it implements some push mechanism if needed).

A crucial component is the "Analyse Rule Builder" which uses the stored patients signals and some general information about the patient (e.g. age) in order to detect/define rules for alerting. We designed this component to have access to patient information, because we think that age would be a factor that has to be considered in order to define thresholds e.g. for heartrate deviations - babys tends to have a much higher heartrate than older people.

# Architectural Characteristics

The Solution design is based on an Event-Driven Architecture, as this provide: 

## Scalability
The solution is deployed in premises but based on a private cloud and designed to be scalable (initially up to 500 patients), this is achieved by a combination of proprietary devices, software and cloud native technologies.

## Security 
Based on premises all data is secure and governed by the required medical standards according to the regional area where it is deployed; exposed APIs externally to integrate with  SaaS is through a single point of integration by using an API Gateway. All communications are encoded.

## Reliability
Services and DBs decomposition allows to have high availability in the solution end to end. It is expected that 8 of the vital signals are monitored separately per specific sensors and data collected independently, see component  ( 3 ) is de-coupled for this specific purposed.

## Performance
By priorizing Event Driven architecture for NFRs data collection (periodicity) per each of the 8 vital signals, see component 1, the aim is to allow the overall solution to adjust to the dimensioned capacity performance. 

## Elasticity
The Design is dimensioned based on  the core initial Requirements;  the deployment of microservices (see roles & responsibilities) allow the solution to cope with traffic peaks driven by users (doctors, nurses and other medical staff).

## Atomicity
Due to requirements to manage and process critital information as vital signals, we proposed decoupling of services and DBs-Intent in order to enable atomicity per transaction . (see Use Cases). 


 
