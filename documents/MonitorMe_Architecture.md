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
