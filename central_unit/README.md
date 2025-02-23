# Central Unit AgOpenGPS/QtOpenGuidance  
(Informationen in deutscher Sprache unten)

Tested yet:
- USB hub, USB devices, USB-C 
- Atmel section with I/Os needed for AOG
- connection to Ethernet
- power supply
- motor driver incl. current feedback
- WAS input
- RS485 / Modbus

Still untested:
- CAN
- ESP32 option
- Bynav option

Not working:
- USB-C PD and data in parallel

This setup does not include new functions but provides a robust, monolitic unit that may be manufactured by professionel EMS providers. The central unit is compatible to [BrianTee‘s Nano firmware](https://github.com/farmerbriantee/AgOpenGPS/blob/master/Support_Files.zip) and to my [improved version](code/Autosteer_USB_CMPS14_low_jitter_twistable/). If the ESP32 option is used, download [AOG_Autosteer_ESP32](https://github.com/mtz8302/AOG_Autosteer_ESP32).

Special focus is given to the robust housing and industrial M12/M8 connetors (optional). PushIn connectors may also be used for single wire connections).

![pic](documentation/Full_M8_M12.jpg?raw=true)
![pic](documentation/Full_PushIn.jpg?raw=true)

# 150 x 92 mm² 
2-layer 35µm, 1,6mm thickness, technology: 0.15mm/0.15mm/0.3mm; SMD assembly: 72 different parts, 252 parts overall, THT: only 2.54mm headers and plugs of your choice

[Link to EasyEDA project](https://oshwlab.com/GoRoNb/agopengps-main-box). The pick-and-place data in the [production data](production_data) is already rotated correctly for JLC.

# Functions in-cabin-unit for autosteering (all SMD + THT connectors)
- integrated Arduino-Nano hardware
- option for ESP32-Module (Arduino Firmware has to be adapted)
- ADC (ADS1115) for wheel angle sensor (WAS)
- roll sensor (MMA8452)
- BNO085 option (as CMPS14 board)
- motor driver (H-bridge of IBT-2) with 2-channel switch-off
- current feedback from motor driver (may be used to quit autosteering on driver interaction)
- 3 protected digital inputs (2oo3 usable as analog inputs on ESP32)
- one optional digital protected 12V output
- mounting holes for Ardusimple board(s)
- connector for Bynav-C1 board
- mounting holes for rooftop unit board (for dual-RTK)
- USB hub (4-port)
- USB and power supply for tablet/notebook via one USB-C (external DC adapter needed)
- integrated CANtact hardware (2x CAN2.0)
- virtual serial port for RS485/ModbusRTU
- many LEDs
- Ethernet/UDP option 

These [connections](Connections.pdf) are provided:
- Supply 12V/24V (optional for motor)
- USB-C to tablet/notebook
- Wheel angle sensor (WAS)
- Digital inputs
- USB via M12-D cable to RTK-GNSS unit
- Steering wheel motor / hydraulic unit
- 2x CAN2.0
- RS485/ModbusRTU
- Incremental sensor for steering wheel
- 2x USB-C (tablet + DC converter/Aux)
- Pixhawk connector (to Ardusimple, RS232 converter or any I²C/serial equipment)
- QWIIC connector
- internal USB connectors
- internal Bynav-C1 connector
- connector to [Ethernet shield](https://de.aliexpress.com/item/32548974618.html) - cut out "clkout" pin before connecting!!
- option to control a 6/2 valve (rename "xU14" to "U14" and delete R59, R76 and F7 in BOM; make a solder joint underneat CN18 to connect signal "+SERVO" to "-MOTOR").

# Initial Operation Instructions
1. if applicable: [assemble missing SMD parts](https://www.youtube.com/watch?v=dLczChhmDCY)
2. assemble alls THT parts: pin headers and connectors
3. connect 3-4 @ J8 (on)
4. slowly increase input voltage on CN4/POWER with current limit of 100mA while checking the 5V voltage (should be 0mA till about 4V, 20mA @ 4.5V, 12mA @ 12V; +5V must not rise up to more than 5.1V!)
5. check +3.3V (should be in the range 3.2..3.4V)
6. connect Arduino Uno board and flash bootloader via P1 (https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP) 
7. disconnect Uno board and connect USB (either CN2 or CN13/CN8/USB-C): The PC should display a new USB hub at least one several serial device 
8. flash [BrianTee's Arduino code](https://github.com/farmerbriantee/AgOpenGPS/blob/master/Support_Files.zip) via Arduino software
9. connect WAS, buttons, motor and do a test with AgOpenGPS

debugging: use terminal like HTerm and find out the correct COM port: connect to each possible one, set "DTR" and release "DTR". When releasing, the red LED blinks 3 times (bootloader). The green LED next to U3 will blink when sending any serial data to the Nano; this is also true when sending with the Arduino IDE (serial monitor)

Some functions are shown in [my English](https://www.youtube.com/watch?v=BdSSeIprusM) or [German video.](https://www.youtube.com/watch?v=w10TqOEIxdU).

Further information are also provided in the upper folder.

There is also an [English user group](t.me/agopengpsinternational) and a [multilingual forum](https://agopengps.discourse.group/t/smd-pcb-project-for-an-all-in-one-compact-pcb-for-aog-qog/3640). 

----------------------------------------------------------------------------------------------------------------------------

# Functional Description (to be continued..)

The Arduino part including roll sensor, ADC and digital inputs is a copy of what BrianTee sugested. Many thanks for his great work!

An ESP32 MINI module is mountable alternatively. In that case, the Arduino firmware has to be adapted and the Nano has to send to reset mode by using a jumper (pins 5+6 of P1).   With an ESP32, one CAN2.0 and the RS485 port is usable e. g. for external relay modules via ModbusRTU. Wifi and Bluetooth may also be used.

The motor driver is the one of the IBT-2. It comes with separated 2-channel power lines at the power plug, so that a double-pole switch with positively opening contacts may be used for safe switch-off. This is especially mandatory for hydraulic steering. Furthermore, a DCDC converter may be used to generate 24V for driving the common Phidgets-Motor with increased dynamic.

The fifth pin of the power plug may either be used for a protected digital output (jumper on 1-2 of J8) or as on-switching signal (jumper on 2-3 of J8). No high currents have to be switched – a small SPST switch is ok. A switch inside the housing would be connected to 3-4 of J8. If an external switch is used to cut of the supply directly, apply a jumper to 2-3 of J8.

All digital inputs are protected and made of constant current sinks. Therefore, the input is inverted, so that either NC pushbuttons may be used or the Arduino firmware may be modified.

The hardware of the ![CANtact project](https://cantact.io/cantact-pro/users-guide.html) based on the Nexperia µC LPC54616 is optionally mountable and may provide two CAN2.0 ports under control of the tablet. One the two ports may alternativly driven by the ESP32.

The RS485 port is like what is also available with a cheap USB-RS485 stick based on a CH340 driver. 

The 4 port USB hub connects the tablet/Notebook via either CN2 (PushIn) or CN8/CN13/TO_PC (USB-C) to the RTK-GNSS receiver and to the Nano or ESP32, the CANtact hardware, the RS-485 port or the internal 2 USB-A socket or the external USB-C socket. These alternatives are possible:

- USB1:  Arduino Nano Clone  (XOR)   ESP32 (if U3 not mounted.)

- USB2:    RS485      (XOR)     Internal USB-A (if U5 not mounted) (XOR)     External USB-C (if U5 not mounted)

- USB3:    CANtact   (XOR)      Internal USB-A (e. g. for ESP32 parallel to Nano)

- USB4:    External RTK receiver via M12-D (if U19 not mounted) (XOR) Bynav-C1 board (XOR) internal USB (e. g. for Ardusimple via [USB cable](https://de.aliexpress.com/item/4000533815530.html) + [Antenna pigtail](https://de.aliexpress.com/item/4000245307369.html)):

Concerning USB4:

Option1: USB via M12-D: U19, R100, R101 not mounted

Option2: I²C via M12-D, serial device for RS232 - converter or Ardusimple @ ARDU1 or Bynav RTK receiver @ U16: R43, R46 not mounted

Option3: I²C via M12-D, additional internal USB (USB1): R43, R46, U19 not mounted

All voltage and I/O functions are equipped with LEDs.

Tablets with USB-C plug may be connected with a single USB-C wire for power and data, when a 12V USB-C DC converter is connected to the 2nd USB plug. Both USB-C plugs are mounted on a piggyback PCB on top of the M8 plugs.

The PCB project was set up with the EDA program „EasyEDA“, which is freely usable. The Altium export has not been checked. 

Milling the aluminum housing, if using M8/M12 (see [drawings](mechanical_data)):
- let the CNC machine drill the 4 holes
- fix the cover with 4 screws M2.5x15
- mill the holes and the counterboring (tool: D=2mm, feed: 300mm/min, 12000RPM, advance: 2mm)
- repeat this step for the other cover

# Functional Safety Advice

Steering systems have to meet at least the agricultural performance level AgrPL d acc. UN ECE R79 (see professional systems like [this](https://www.mobil-elektronik.com/uploads/media/Prospekt_Brochure_ME_Agricultural_Vehicles_ENG.pdf))!

Especially with hydraulic control, the vehicle may likely get uncontrollable on a failure of the valve-controlling electronic which may result in fatal casualties or serious injuries!

For AgOpenGPS, the required safety level may be archieved by the fail-safe absense of power for the valve/motor controlling unit. This function must have at least the same safety/performance level.

The mandatory application for approval may base on the following arguments based on ISO 13849: 

- emergency-stop key meets IEC 60947-5-5/ISO 13850, in line with fuse meeting IEC60269-1 (both acc. to table D.3, ISO 13849-2)
- use of MELF type resistors for all 8.2kOhm resistors to meet table D.14, ISO 13849-2
- module is mounted inside a IP54 housing (acc. table D.5, ISO 13849-2)
- wiring acc. IEC 60204-1 (acc. table D.4, ISO 13849-2) 
- measure against tin whiskers are taken (acc. D.2.2, ISO 13849-2), e. g. use of 60/40 solder paste
- futhermore the software shall check the values of the signal „safesense“

(some ordering information underneath the German text)

# (German copy) Zusammenfassung

Dieser Leiterplattenentwurf integriert mit Ausnahme des RTK-GNSS-Empfängers alle elektronischen Funktionen, die für einen automatische Lenkung der Projekte AgOpenGPS und QtOpenGuidance notwendig sind. So ist es möglich, auch die zweikanalige RTK-Einheit von Matthias (MTZ8302) zu verwenden. Ziel war es nicht, neue Hardwarefunktionen bereitzustellen, sondern eine robuste, monolitische, von professionellen Bestückern herstellbaren Einheit zu entwicklern, auf der der aktuelle Stand der Arduino-Firmware von BrianTee kann unverändert verwendet werden. 

Besonderer Wert wurde auf ein robustes Gehäuse und insbesondere robuste Steckverbinder und Kabel gelegt. Es sind M12/M8-Steckverbinder gerade oder abgewinkelt vorgesehen. Alternativ sind über auch steckbare PushIn-Steckverbinder bestückbar.

Folgende Funktionen sind vorgesehen:
- integrierte Arduino-Nano-Hardware
- Option für ESP32-Leiterplatte (Arduino-Firmware muss angepasst werden)
- ADC (ADS1115) für Lenkwinkelsensor
- Neigungssensor (MMA8452)
- Motortreiber (H-Brücke wie IBT-2) mit zweikanaliger Abschaltung
- geschützte Digitaleingänge (als Konstantstromsenke)
- USB-Hub (4-port)
- Spannungsversorgung auch für Tablet/Notebook via USB-C
- integrierte CANtact-Hardware (2x CAN2.0)
- virtueller serieller Port für RS485/ModbusRTU
- ganz viele Leuchtdioden

Folgende Anschlüsse sind vorgesehen:
- Stromversorgung 12V/24V (optional für Motor)
- USB-C zu Tablet/Notebook
- Lenkwinkelsensor
- Digitaleingänge
- USB via M12-D - Kabel zu RTK-GNSS-Einheit
- Lenkradmotor bzw. Hydraulikeinheit
- 2x CAN2.0
- RS485/ModbusRTU
- Inkrementalgeber für Lenkradsensor
- 2x USB-C (Tablet + DC-Wandler/Aux)
- Pixhawk als Alterantive zu USB für das Ardusimple-Board, aber auch für allgemeine I²C/serielle Anwendungen
- QWIIC für I²C-Kram

# Im Detail 

(wird fortlaufend ergänzt, allerdings meistens zunächst der englische Teil)

Der Arduino-Nano-Teil einschließlich der Neigungssensoren, des ADC und der Eingänge entspricht dem von BrianTee entwickelten Schaltplan. An dieser Stelle sei ihm für seine hervorragende Arbeit gedankt!

Eine ESP32-Leiterplatte ist alternativ bestückbar. In diesem Fall muss die Arduino-Software angepasst werden und der Nano dauerhaft im Reset-Zustand gehalten werden (Jumper auf 5, 6 von P1). Mit einem ESP32 steht dann hardwaremäßig auch ein CAN2.0 und ein RS485-Port zur Verfügung sowie Wifi und Bluetooth.

Der Motortreiber entspricht dem IBT-2. Er verfügt über eine getrennte, zweikanalig ausgeführte Stromversorgung am Spannungsversorgungsstecker, sodass hier ein geeigneter zweikanaliger zwangsöffnender Sicherheitsschalter angeschlossen werden kann. Insbesondere bei Hydraulikeinbindung ist dies zwingend. Auch kann so ein DC/DC-Wandler 12V > 24V vorgeschaltet werden, um eine höhere Lenkdynamik z. B. für den beliebten Phidgets-Motor zu erreichen.

Die 5. Leitung des Spannungsversorgungssteckers kann entweder mit einem geschützten Digitalausgang belegt werden (Jumper auf 1-2 von J8) oder für den Ein-Schalter verwendet werden (Jumper auf 2-3 von J8). Es wird keine Leistung geschaltet – ein kleiner Schalter reicht. Ein Schalter im Gehäuse wird an die 3-4 von J8 angeschlossen. Wird ein externer Leistungsschalter verwendet, müssen diese Kontakte mit einem Jumper überbrückt werden.

Die Digitaleingänge sind als Konstantstromsenke ausgeführt. Dadurch ist die Logik allerdings invertiert, also bitte Öffnerkontakte verwenden oder die Arduino-Firmware entsprechend anpassen.

Die Hardware des [CANtact-Projektes](https://cantact.io/cantact-pro/users-guide.html) auf Basis des Nexperia-µC LPC54616 ist optional bestückbar und stellt dann 2 CAN2.0-Schnittstellen zur Verfügung. Eine der beiden CAN2.0-Schnittstellen kann alternativ auch mit dem ESP32 angesteuert werden.

Die RS-485-Schnittstelle entspricht dem, was ein preiswerter USB-RS485-Stick auf Basis einen CH340 leistet. 

Der 4-Port-USB-Hub verbindet das Tablet/Notebook zum RTK-GNSS-Empfänger und wahlweise zum Nano bzw. ESP32, der CANtact-Hardware, der RS-485-Schnittstelle bzw. internen 2 USB-A-Buchsen einer externen USB-C-Buchse. Folgende Alternativen sind mit den 4 Ports möglich:

- USB1:  Arduino Nano Clone  (ODER)   ESP32 (wenn U3 nicht bestückt ist - ggf. auslöten mit Heißluft und Pinzette)

- USB2:    RS485      (ODER)     Interner USB-A (wenn U5 nicht bestückt ist) (ODER)     Externer USB-C (wenn U5 nicht bestückt ist)

- USB3:    CANtact   (ODER)      Interne USB-A (z. B. für ESP32 parallel zu Nano)

- USB4:    Externer RTK-Empfänger via M12-D (wenn U19 nicht bestückt ist) (ODER) Bynav-C1 Empfänger (ODER) interne USB (z. B. für Ardusimple via [USB cable](https://de.aliexpress.com/item/4000533815530.html) + [Antenna pigtail](https://de.aliexpress.com/item/4000245307369.html)):

Zu USB4:

Option1: USB via M12-D: U19, R100, R101 nicht bestücken/auslöten

Option2: I²C via M12-D, serieller Konverter für z. B. RS232 ODER Ardusimple @ ARDU1 (Pixhawk-Kabel) ODER Bynav RTK receiver @ U16: R43, R46 nicht bestücken/auslöten

Option3: I²C via M12-D, zusätzliche interne USB-A-Buchse (USB1): R43, R46, U19 nicht bestücken/auslöten

Alle Spannungen und I/O-Funktionen verfügen über Leuchtdioden.

Neben den I/O-Steckverbindern sollte mindestens P1 (Stiftleiste 2x5pol RM 2,54mm) bestückt werden, um die initiale Programmierung mit Hilfe eines Arduino Uno durchführen zu können. Notfalls können dort aber auch Kabel an- und nach erfolgter Programmierung wieder abgelötet werden. Je nach Einschaltoption wird noch J8 (Stiftleiste 1x4pol RM 2,54mm) und ein Jumper benötigt.

Tablets mit USB-C-Schnittstelle können mit einem einzigen Kabel für Daten und Stromversorgung angeschlossen werden, wenn ein 12V-USB-C-Ladeadapter an die zweite USB-C-Buchse angeschlossen wird. Die beiden USB-C-Buchsen sind auf Huckepack-Leiterplatten über den M8-Steckverbindern angeordnet.

Das Leiterplattenprojekt wurde im frei verfügbaren EDA-Programm „EasyEDA“ erstellt. Der Export nach Altium ist ungeprüft. 

Weitere allgemeine Informationen finden sich auch im übergeordneten Ordner.

Hilfestellung gibt es auch in der  [deutschsprachigen Nutzergruppe](t.me/AGOpenGPSGerman) und im [mehrsprachigen Forum](https://agopengps.discourse.group/t/smd-pcb-project-for-an-all-in-one-compact-pcb-for-aog-qog/3640). 

Speziell zur Leiterplatteninbetriebnahme gibt es auch ein [Video](https://www.youtube.com/watch?v=w10TqOEIxdU).

# Erstinbetriebnahme
1. Ggf. fehlende SMD-Bauteile [nachbestücken](https://www.youtube.com/watch?v=dLczChhmDCY)
2. gewünschte Stiftleisten und Stecker auflöten
3. Pins 3-4 @ J8 brücken (on)
4. langsam die Spannung an CN4/POWER mit einer Strombegrenzung von 100mA hochregeln und gleichzeit die 5V-Spannung im Auge behalten (Eingangsstrom: 0mA bis ca. 4V, 20mA @ 4.5V, 12mA @ 12V; +5V darf nicht über 5,1V ansteigen!)
5. +3.3V überprüfen (sollte im Bereich 3.2..3.4V sein)
6. Arduino-Uno-Board mit P1 verbinden und Bootloader flashen (https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP) 
7. Uno-Board abstöpseln
8. über USB mit einem PC verbinden (entweder über CN2 oder CN13/CN8/USB-C): Der PC muss einen neuen Hub und mindestens eine neue serielle Schnittstelle anzeigen; bei mehrereren ist die des Arduino meistens die erste)
9. [BrianTee's Arduino-Code](https://github.com/farmerbriantee/AgOpenGPS/blob/master/Support_Files.zip) wie gewohnt mit der Arduino-Software flashen
10. Lenkwinkelsensor (auch als WAS oder LWS bezeichnet), Schalter, Motor anschliessen und mit AgOpenGPS testen

debugging: use terminal like HTerm and find out the correct COM port: connect to each possible one, set "DTR" and release "DTR". When releasing, the red LED blinks 3 times (bootloader). The green LED next to U3 will blink when sending any serial data to the Nano; this is also true when sending with the Arduino IDE (serial monitor)

Aktuell (2/2021) nicht bei JLCPCB bestückbar: U2, U10 (jeweils Lager leer), U11 (nicht bei LCSC verfügbar) und Steckverbinder (prinzipiell)

Fräsen des Alugehäuses, falls M8/M12 verwendet werden (siehe auch [Zeichnungen](mechanical_data)):
- mit der CNC-Maschine 4 Löcher bohren
- Seitenteil mit 4 Schrauben M2.5x15 dort befestigen
- Löcher und Senkung fräsen (Fräser: D=2mm, Vorschub: 300mm/min, 12000RPM, Zustellung: 2mm)
- Mit dem anderen Seitenteil die anderen Fräsdaten verwenden

# Hinweis zur Funktionalen Sicherheit

Fest verbaute Lenksystem, die z. B. in die Hydraulik eingreifen, müssen den Agricultural performance level AgrPL d nach UN ECE R79 (siehe z. B. professionelle Systeme wie [dieses](https://www.mobil-elektronik.com/uploads/media/Prospekt_Brochure_ME_Agrarfahrzeuge_DEU.pdf)) erfüllen!

Im Falle eines Ausfalls und dadurch ausgelösten Lenkbewegungen kann es zu tödlichen Unfällen oder schweren Verletzungen kommen!

Mit AgOpenGPS kann der notwendige Sicherheitslevel durch die fehlersichere Abschaltung der elektrischen Leistung für die Lenkaktoren erfolgen. Diese Abschaltung muss mindestens diesen Sicherheitslevel erfüllen.

Bei passender Installation mag eine Abnahme mag auf folgenden Argumenten basieren, die sich auf die EN-ISO 13849 stützen: 

- Schalter für die Stromversorgung der Lenkaktorik entspricht EN-IEC 60947-5-5/EN-ISO 13850, die Vorsicherung entspricht IEC60269-1 (beides entsprechend Tabelle D.3, EN-ISO 13849-2)
- die Leiterplatte enthält MELF-Widerstände (8.2kOhm), die Tabelle D.14, EN-ISO 13849-2 erfüllen (standardmäßig ist hier 1206 bestückt)
- die Leiterplatte ist in ein Gehäuse eingebaut, das mindestens IP54 erfüllt (Tabelle D.5, EN-ISO 13849-2)
- Die Verkabelung ist geschützt gemäß EN-IEC 60204-1 (Tabelle D.4, EN-ISO 13849-2) 
- Massnahmen gegen "tin whiskers" wurden getroffen, z. B. mit 60/40 Lötzinn gelötet (nach D.2.2, EN-ISO 13849-2)
- die Firmware sollte die Spannung des Signals „safesense“ messen und plausibilisieren

Alle Daten diese Repositories stehen unter der CERN Open-Hardware-Lizenz 1.2. Jede Haftung für Personenschäden und Sachschäden auf Grund Fehlern der Beschreibung oder Fehlern der Leiterplatte oder Software wird ausgeschlossen! 

# Component Sources (examples) / Beispiele Bezugsquellen:

M12/M8, general components:	    [Mouser](https://www.mouser.com/Connectors/Circular-Connectors/Circular-Metric-Connectors/_/N-76q5i?Keyword=conec&FS=True), [Reichelt](https://www.reichelt.de/de/en/sensor-actor-connectors-c7505.html?MANUFACTURER=CONEC&START=0&OFFSET=16&LANGUAGE=EN&&r=1) - [Reichelt list](https://www.reichelt.de/my/1816987), [cable e. g.](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=9457610150) 

(see [BOM-THT](production_data) for further information)

PushIn:		  Weidmueller [S2C-SMT 3.50/04/180G 3.5SN BK BX](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1290030000) + [B2CF 3.50/04/180 SN OR BX](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1277270000) via [Conrad](https://www.conrad.com)/[digikey](https://www.digikey.com)

(see [BOM-THT](production_data) for further information)

ESP32 MINI:	[Aliexpress](https://de.aliexpress.com/wholesale?catId=0&SearchText=esp32+mini)

Housing for M8/M12 option: [Aliexpress](https://de.aliexpress.com/item/32967598546.html)

RTK receiver: 
- [Ardusimple](https://www.ardusimple.com/product/simplertk2b-basic-starter-kit-ip65/) (state-of-the-art receiver - mounting holes provided / Standardempfänger - Montagelöcher hierfür sind vorgesehen)
- [GNSS store](https://www.gnss.store/gnss-gps-modules/105-ublox-zed-f9p-rtk-gnss-receiver-board-with-sma-base-or-rover.html) (same type of receiver - especially for rooftop unit / gleicher Empfänger - gute Lösung für die Dacheinheit)
- [Bynav](https://www.bynav.com/en/products/gnss-boards/c1.html) (special software needed! / benötigt besondere Software!)

