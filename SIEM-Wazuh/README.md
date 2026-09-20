# SIEM Lab Wazuh
Im Rahmen dieses Projekts wurde eine eigene SIEM-Laborumgebung mit Wazuh aufgebaut. Ziel ist es, die grundlegenden Funktionen eines Security Information and Event Management Systems praktisch kennenzulernen und ein solides Verständnis für Log-Sammlung, Event-Analyse, Alerting und Korrelation aufzubauen.


## Aufbau der Laborumgebung
Die gesamte Laborumgebung läuft auf meinem MacBook Pro (M3). Darauf ist Docker Desktop sowie VMware Fusion als Hypervisor installiert. Als SIEM habe ich mich für die Opensource Lösung Wazuh entschieden. Wazuh läuft lokal auf dem MacBook in Docker Containern. Als Client-Systeme verwende ich Windows 11 und Kali Linux. Die beiden Systeme laufen als Virtuelle-Maschinen in VMware Fusion. Auf den beiden Clients wurde dann der Wazuh Agent installiert.

Die nachfolgende Grafik zeigt den Aufbau der Laborumgebung auf:

<p align="center">
  <img src="SIEM-Wazuh/images/Aufbau_Laborumgebung_Bild1.png" width="60%">
  <br>
  <em>Aufbau Laborumgebung</em>
</p>

## Log-Sammlung
Die zentrale Sammlung von Logdaten ist eine grundlegende Funktion eines SIEM. Anstatt die Logs ausschliesslich lokal auf den einzelnen Systemen zu speichern, werden sie an Wazuh übertragen und sollen dort zentral verarbeitet und analysiert werden. 

### Datnequellen
In dieser Laborumgebung werden die Windows-11- und Kali-Linux-VM als Datenquellen verwendet. Auf beiden Systemen ist ein Wazuh Agent installiert, welcher die relevanten Logdaten erfasst und an den Wazuh Manager überträgt. Damit die Logs an Wazuh gesendet werden können, muss auf den Client-Systemen der Wazuh Agent installiert werden. 

Auf diesen beiden Screenshots ist zusehen, dass der Agent als lokaler Service auf den Systemen installiert wurde.

<p align="center">
  <img src="SIEM-Wazuh/images/Wazuh_Agent_Kali_Bild2.png" width="60%">
  <br>
  <em>Wazuh Agent auf Kali</em>
</p>

<p align="center">
  <img src="SIEM-Wazuh/images/Wazuh_Agent_Windows_Bild3.png" width="60%">
  <br>
  <em>Wazuh Agent auf Windows</em>
</p>

Im Wazuh Dashboard werden die beiden Agents bzw. Systeme die mit dem Manager verbunden sind angezeigt:

<p align="center">
  <img src="SIEM-Wazuh/images/Agents_Wazuh_Dashboard_Bild4.png" width="60%">
  <br>
  <em>Agents in Wazuh Dashboard</em>
</p>

### Datnefluss
Die Logdaten werden auf den Windows- und Kali-Systemen durch den jeweiligen Wazuh Agent erfasst und an den zentralen Wazuh Manager übertragen. Der Manager verarbeitet und analysiert die eingehenden Events anhand von Decodern und Regeln. Anschliessend werden die aufbereiteten Daten im Wazuh Indexer gespeichert und über das Wazuh Dashboard für Suche, Analyse und Visualisierung bereitgestellt.

## Die Logs in Wazuh Dashboard
In diesem Kapitel wird aufgezeigt, dass die Logs erfolgreich an den Wazuh Manager gesendet und im Dashboard angezeigt werden können.

### Windows Event
Zur Überprüfung der Log-Sammlung, wurde bewusst ein falsches Passwort bei der Anmeldung eingegeben. Dabei wird bei Windows ein Security Event mit der Event-ID “4625” generiert:

<p align="center">
  <img src="SIEM-Wazuh/images/Windows_Event_Bild5.png" width="60%">
  <br>
  <em>Windows Event</em>
</p>

Der Wazuh Agent erfasst dieses Ereignis und überträgt es an den Wazuh Manager. Im Wazuh Dashboard kann das Event anschliessend analysiert werden:

<p align="center">
  <img src="SIEM-Wazuh/images/Failed_Win_Logon1_Bild6.png" width="60%">
  <br>
  <em>Failed Windows Logon 1/2</em>
</p>

<p align="center">
  <img src="SIEM-Wazuh/images/Failed_Windows_Logon2_Bild7.png" width="60%">
  <br>
  <em>Failed Windows Logon 2/2</em>
</p>

### Kali Event
Auf der Kali VM wurde ebenfalls ein fehlgeschlagener Authentifizierungsversuch erzeugt. Das Ereignis wurde über das systemd Journal erfasst und durch den Wazuh Agent an den Manager übertragen. Im Dashbord sieht dies wie folgt aus:

<p align="center">
  <img src="SIEM-Wazuh/images/Failed_Linux_Logon_Bild8.png" width="60%">
  <br>
  <em>Failed Linux Logon</em>
</p>

## Regeln und Alarme
In diesem Kapitel wird anhand einer einfachen Testregele gezeigt, wie eigene Regeln in Wazuh erstellt und gezielt ausgelöst werden können. 

### Funktionsweise von Rules
In Wazuh können eingehende Events anhand von definierten Regeln erstellt ausgewertet. Eine Regel legt fest, auf welche Merkmale eines Events reagiert werden soll und welcher Schweregrad einem Treffer zugewiesen wird. Wird eine Regel erfüllt, kann daraus ein Alert also ein Alarm entstehen. Neben den bereits vorhandenen Standardregeln können auch eigene Regeln erstellt werden. Dadurch lässt sich Wazuh an spezifische Anforderungen anpassen. 

### Eigene Rule erstellen
Für das Lab wurde eine eigene Regel erstellt. Regeln können im Dashboard unter «Server Management» > «Rules» erstellt werden.
Die Regel wird ausgelöst, wenn ein eingehendes Event den String «SIEM_CUSTOM_RULE_TEST» enthält. Der erzeugte Alert erhält den den Schweregrad «10». 

<p align="center">
  <img src="SIEM-Wazuh/images/Test_Regel_Bild9.png" width="60%">
  <br>
  <em>Test Regel</em>
</p>

### Testevent erzeugen
Um diese Regel zu testen, wurde auf der Kali-VM ein eigenes Event erzeugt:

<p align="center">
  <img src="SIEM-Wazuh/images/Kali_TestEvent_Bild10.png" width="60%">
  <br>
  <em>Kali Testevent erzeugen</em>
</p>

### Alert im Dashboard
In Wazuh kann man nach der definierten Rule ID filtern und der Alert wird angezeigt. Dies zeigt, dass der Agent das Testevent erfasst hat und an den Wazuh Manager weitergeleitet hat. Auch der Schweregrad wurde korrekt gesetzt. 

<p align="center">
  <img src="SIEM-Wazuh/images/Custom_Alert_Bild11.png" width="60%">
  <br>
  <em>Custom Alert in Wazuh Dashboard</em>
</p>

Mit diesem Test konnte gezeigt werden, dass eigene Regeln in Wazuh erstellt und gezielt ausgelöst werden können. Durch die Definition eigener Bedingungen lässt sich das SIEM an spezifische Logquellen und Anwendungsfälle anpassen. Sobald ein eingehendes Event die definierte Bedingung erfüllt, kann Wazuh daraus automatisch einen Alert mit einem festgelegten Schweregrad erzeugen. 



















