# Section and Rate Control Unit for AgOpenGPS
(Informationen in deutscher Sprache unten)
Top view:
![pic](3D_top.png?raw=true)

Bottom view:
![pic](3D_bottom.png?raw=true)

This project is a section control unit for AgOpenGPS. It is intended to be used together with the Central Unit 2.0 or Central Unit 3.0 (=Teensy) in the same or a separate housing (can be found here in Git as well). It may connected by either Ethernet, WiFi or RS485.

Brian's Concept only provided drivers for the sections w/o the possibility of manual interference. In that case, no rocker switches will be mounted, and the unit may also be located directly on the trailer.

![pic](documentation/BlockDiagram.png?raw=true)

MTZ8302 and WEder suggested a manual steering option in addition, so in general the functions provided by a Müller-Elektronik  control plus AgOpenGPS integration. To use that one or two of this PCB can be used, because both functions can either be provided on a single board or divided into two boards, of wich one contains the rocker switches and the µC and the other one (e. g. located on the trailer) the relays (and optionally another µC). If using a single µC-dual PCB solution, both units are conneced via a HDMI cable. If using two µC, options are USB, RS485, Ethernet or WiFi. 

Brian's software will work quite out-of-the box with minimal mods. MTZ8302's needs more adaptations, mainly due to the fact, that each pin can both drive the section and read back the status of the rocker switch.

- USB-C 
- Atmel section with I/Os needed for AOG
- connection to Ethernet via external module is possible
- power supply
- valves output
- RS485 / Modbus option
- ESP32 option (alternative to Atmel)

This setup does not include new functions but provides a robust, monolitic unit that may be manufactured by professionel EMS providers. 

Special focus is given to the robust housing and industrial PushIn connetors. 

# Detailed Description

In general, you can build up this unit as one (no need for the HDMI connector) or divide into a control unit that includes the µC and a power unit that might be directly on the trailer. On the power unit, you need all the parts from the schematic scheet "Machine Section Control" + parts around U3 (12V sensor supply if needed) only, and for the conrol unit on the other hand, you don't need that parts. The HDMI connector is needed on both PCBs for sure.

Heart of the section control is either an Atmel µC as U26 following [Brian's suggestion](https://github.com/farmerbriantee/AgOpenGPS/tree/master/Support/RateControl) or an ESP32 as U6 following [Matthias' suggestion](https://github.com/mtz8302/AOG_SectionControl_ESP32). For using grade control via ultrasonic sensors, the ESP32 is needed. If the ESP32 is used, X3 is not necessary and should be deleted from the BOM in order to avoid additional costs. Do not mount both µCs!

No matter what µC you wanna use, I follow Matthias' idea, that a manual control of the sections should be possible in any case. This is done with the switches SW1..SW8 for 8 different sections maximum. Mount less switches for less sections of course. The priciple is the following: If using an on-off-on switch, you can either switch to "1" (via main switch), "0" or the µC output for AgOpenGPS control. In any case, the µC can read back that status by temporary switching it's I/O pin to input. The output status is stored in the caps C1..C8 for that moment. R1 and Q1 (for channel 0) are a constant current sink, so that common mode voltages do not matter, if you have your power unit on the trailer. That is also the separation between the power drawn from USB (control voltages) and the direct 12V power supply used for the output relays, so that any miswiring of the power section cannot kill any USB-related parts or the tablet.

In order to safe wires, the signals for pressure up/down and slope control are tristate signals: "0", high-impedance, "1". With the help of the comparators U1 and U2, this is tranfered into 2 bits each for the relay control again. U1 and U2 have open collector outputs, so that they can sink the relay driving transistors connected to 12V directly. Same priciple as with Q1..Q8.

Mind that there is intentionally no connection between the USB voltage and the power stage supply voltage. In case, your tablet is fully isolated from the tractor power system (e. g. by using an power inverter + AC/DC charger or running from battery), the 12V LED will now light even if 12V is connected and the 5V LED is lighting; you need to close jumper "ON1" to connect 0V (USB) with 0V (power stage):

![pic](documentation/JumperON1.jpg?raw=true)

The relays do the power driving. Yes, old style, but simple and available right now. They can control electromagnetic valves (power on = flow on) or motor-controlled valves (one polarity = flow on, other polarity = flow off). For that, set one solder bridge on the board (on top side, between REL2 and REL3).

All actuators are connected to the connectors next to the relays. Suggestest type: [Weidmueller LSF-SMT 5.08/02/90 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1824810000) or [Weidmueller LSF-SMT 5.00/02/135 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1884340000). Or the China ones in the BOM, but don't complain... ;)

Sensors may be used for pressure, flow and optionally for speed pulses. In that case, only 7 sections are available and you must not mount the jumper "REL" next to switch "SW1".

The sensors may be supplied with 5V (from USB - they need only some mA) or 12V (U3 works as short circuit protection). Suggestest connector type here: [Weidmueller LSF-SMT 3.50/04/180 3.5SN BK TU](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1825660000) or [Weidmueller LSF-SMT 3.50/04/135 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1885670000). Or again the China ones in the BOM, but don't complain... ;) 

When using Ethernet/Wifi only, the power supply around U250 is needed. If not, delete U250, L250, D251 and C250..C258 from the BOM to safe money.

Same is true for U4, which is only needed for the ESP32. If no ESP32, delete U4 and C65..C67 from BOM.

Control data interface:
USB is needed if using an ESP32 - at least for programming. Suggested is the USB-C plug, but you can also connect [external connectors](https://www.aliexpress.com/item/1005002859111621.html) to CN3.

For serial data communication while in use or programming (Atmel only), you can use either USB or RS485 (most reliable) or - if you have it in the same box as the centrol board - TTL-serial directly via H10. If using RS485, you need U5 and U23 must be either unmounted or the jumper "USBOFF" must be set. If you're using USB, you can delete U5 in the BOM to safe money; if you're using RS485 or H10, you may delete all U23, U5 and USBC as long as you don't need it for programming.

DP1 is optional for displaying things like pressure, etc. 

There's no use for U10..U14 today, as there is not PWM power stage designed yet. Let's see next year..

The initial programming of the Atmel (if any) is done via "PRGBOOT". [Please refer to this how-to.](https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP).

Finally, the steer and work switches may be used for AgOpenGPS steering, if this PCB in mounted inside the same box.

# 150 x 100 mm² 
2-layer 70 or 35µm, 1,6mm thickness, technology: 0.2mm/0.2mm/0.3mm; SMD assembly: 72 different parts, 252 parts overall, THT: only relays, 2.54mm headers and plugs 

[Link to EasyEDA project](https://oshwlab.com/GoRoNb/aog_io-box_copy). The pick-and-place data in the [production data](production_data) is already rotated correctly for JLC.

# Functions in-cabin-unit for section control
- integrated Arduino-Nano hardware
- ESP32-Module option
- 8 valve outputs (relays)
- +/- pressure output
- slope control output
- Steer/Work switch for AgOpenGPS
- LED display
- mounting holes
- USB-C / WiFi / RS485 / (Ethernet)
- virtual serial port for RS485/ModbusRTU
- many LEDs
- Ethernet/UDP option 
- WIFI option via ESP32

These [connections](Connections.pdf) are provided:
- Supply 12V/24V (optional)
- USB-C to tablet/notebook
- RS485/ModbusRTU
- USB-C
- Ethernet module (cable to connector PRGBOOT)

# Initial Operation Instructions
1. if applicable: [assemble missing SMD parts](https://www.youtube.com/watch?v=dLczChhmDCY)
2. assemble all THT parts: pin headers and connectors
3. Atmel µC: apply 5V e. g. via USB (current should be about 22mA w/o U10, U11, U12 and U14). ESP32: Slowly increase input voltage on USB and with current limit of 100mA while checking the 3.3V voltage; +3.3V must not rise up to more than 3.4V!If U250 is mounted: Slowly increase input voltage on 12V input with jumpers "ON1" and "ON2" closed: 0mA@7V, 17mA@8V (Atmel), 15mA@12V, 9mA@20V; 3.3V must not rise up to more than 3.4V, 5V not to 5.1V. 
4. connect Arduino Uno board and flash bootloader via "PRGBOOT" (https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP) 
5. disconnect Uno board and connect USB (if available): The PC should display a new USB virtual serial device based on CH340 
6. flash Atmel/ESP32 via Arduino software
7. connect sensors, etc do a test with AgOpenGPS

Connection for initial programming:
![pic](documentation/InitialProgramming.jpg?raw=true)

debugging: use terminal like HTerm and find out the correct COM port: connect to each possible one, set "DTR" and release "DTR". When releasing, the red LED blinks 3 times (bootloader). The green LED next to U3 will blink when sending any serial data to the Nano; this is also true when sending with the Arduino IDE (serial monitor)

Further information are also provided in the upper folder.

There is also an [English user group](t.me/agopengpsinternational) and a [multilingual forum](https://discourse.agopengps.com/t/section-control-for-the-central-unit/7779/10?u=goronb). 

----------------------------------------------------------------------------------------------------------------------------

# Übersicht

Bei diesem Board handelt es sich um eine Section-Control-Steuerung für AgOpenGPS. Sie kann zusammen mit der Central Unit 2.0 oder Central Unit 3.0 (=Teensy) in einem Gehäuse verbaut werden oder eben auch nicht. Ein Anbindung über Ethernet, WiFi oder RS485 ist auch möglich.

Brian's Konzept sieht lediglich eine Ansteuerung der Section ohne weitere Eingriffsmöglichkeit vor. In dem Fall werden keine Schalter bestückt, und die Einheit kann ggf. auf der Anhängespritze selbst angebaut werden.

MTZ8302 bzw WEder schlagen ergänzend die Möglichkeit einer alternativen Handsteuerung vor, also im Prinzip das, was eine Müller-Elektronik-Steuerung leistet plus die Einbindung in AgOpenGPS. Hierzu können ein- oder zwei Leiterplatten verwendet werden, da sich sowohl beide Funktionalitäten in einem Board bestücken lassen, als auch separat, also z. B. die Leiterplatte mit den Schaltern im Traktor und dieselbe Leiterplatte mit Relais bestückt im Trailer. Dabei kann sowohl eine ein-µC-Lösung gewählt werden, bei der das Relais-Modul via HDMI-Kabel verbunden wird, als auch eine zwei-µC-Lösung mit USB, RS485, Ethernet oder WiFi. 

Die Software von Brian läuft auf dem Atmel ohne große Änderungen. Die Software von MTZ8302 muss allerdings angepasst werden, da an einem Pin sowohl die passende section aktiviert wird, als auch der zugeordnete Schalter zurückgelesen wird.


# Schaltungsbeschreibung (Copy of English text)

Die Teilbreitensteuerung kann als eine Einheit aufgebaut werden (bei der dann keine HDMI-Buchse benötigt wird) oder in Kontrolleinheit und Leistungsteil getrennt werden, falls man z. B. den Leistungsteil direkt auf dem Spritzenanhänger anbringen möchte. Für den Leistungsteil muss nur alles bestückt sein, was sich auf dem Schaltplanblatt "Machine Section Control" befindet + die Bauteile um U3 (Sensorversorgung 12V falls benötigt). Auf dem Controlboard werden diese Teile dann nicht benötigt. Die HDMI-Buchse muss natürlich auf beiden drauf sein..

Kern der Schaltung ist entweder ein Atmel µC (U26), wie von [Brian vorgeschlagen](https://github.com/farmerbriantee/AgOpenGPS/tree/master/Support/RateControl) oder ein ESP32 (U6), wenn man [Matthias' Vorschlag](https://github.com/mtz8302/AOG_SectionControl_ESP32) folgt. Möchte man eine automatische Handneigungsverstellung mit Ultraschallsensoren nutzen, muss ein ESP32 verwendet werden. Wenn ein ESP32 benutze wird, wird X3 nicht benötigt und kann in der BOM gelöscht werden, um Bestückkosten zu sparen. Nicht beide µC bestücken!!

Unabhängig vom verwendeten µC stimme ich mit Matthias überein, dass eine manuelle Kontrolle der Teilbreiten und damit ein Betrieb ohne AgOpenGPS jederzeit möglich sein sollte. Acht Teilbreiten sind maximal möglich (mit Geschwindigkeitssensor sieben). SW1..SW8 sind An-Aus-An-Taster und schalten zwischen "1" (über Gesamtschalter), "0" und Automatikbetrieb um. Dazu kann der µC seine I/Os kurz auf Eingang umschalten und dann den Zustand zurücklesen, der in den Kondensatoren C1..C8 "gespeichert" wird. R1 und Q1 (am Beispiel der ersten Teilbreite) bilden eine Konstantspannungssenke mit hoher Gleichtaktfestigkeit und trennen somit den von USB versorgten Bereich (µC und Co) vom Leistungsteil, so dass Fehler im Leistungsteil nicht am µC oder Tablet zerstören können.

Deshalb sind die Versorgungsspannungen von USB und Leistungsendstufe bewusst unverbunden. Wenn also das Tablet vollkommen potentialfrei betrieben wird (z. B. Akkubetrieb oder Steckernetzteil + 230V-Inverter), wird selbst bei angeschlossenen 12V die 12V-LED nicht leuchten, wenn die 5V LED leuchtet, und es muss Jumper "ON1" geschlossen sein (Bild):

![pic](documentation/JumperON1.jpg?raw=true)

Um Verbindungsleitungen zu sparen, werden für die Druck- und Hangneigungsverstellung 3 Zustände pro Draht übertragen: "0", hochohmig, "1". Mit Hilfe der Komparatoren U1 und U2, die open-collector-Ausgänge haben, werden dann wieder die jeweils 2 Relais angesteuert genauso wie die Teilbreiten über Q1..Q8.

Hier werden Relais für die Ansteuerung der Teilbreiten benutzt, was eigentlich nicht mehr Stand der Technik ist, aber gerade die Bauteile der Wahl sind, weil Endstufenhalbleiter nicht beschaffbar sind. Es muss ja auch noch etwas für nächstes Jahr zu tun geben...

Es können sowohl elektromagnetische Ventile (Spannung an = Düse an) oder auch Motorventile (Düse ist je nach Polarität an oder aus) verwendet werden. Für Motorventile muss die Lötbrücke auf der Oberseite der Leiterplatte zwischen REL2 und REL3 geschlossen werden (Lötpunkt).

Die Aktoren werden über die Steckverbinder neben den Relais angeschlossen. Mein Vorschlag: [Weidmueller LSF-SMT 5.08/02/90 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1824810000) oder [Weidmueller LSF-SMT 5.00/02/135 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1884340000). China geht auch, aber bitte nicht über unzuverlässige Kontakte beschweren... ;)

Sensoren sind vorgesehen für Druck, Druchfluss und optional für Geschwindigkeitspulse. Bei letzterem sind dann "nur" 7 Teilbreiten möglich und der Jumper "REL" darf nicht gesetzt sein. Am besten in dem Fall gleich das Relais "REL1" weglassen.

Die Sensoren werden aus den 5V des USB-Ports versorgt - sie brauchen ja nur ein paar mA oder aus 12V (hier sorgt U3 für den notwendigen Kurzsschlussschutz). Vorschläge für Steckverbinder hier: [Weidmueller LSF-SMT 3.50/04/180 3.5SN BK TU](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1825660000) oder [Weidmueller LSF-SMT 3.50/04/135 3.5SN](https://catalog.weidmueller.com/catalog/Start.do?localeId=en&ObjectID=1885670000). China geht ebenfalls; es gilt das oben gesagte.. ;) 

Wenn zum Betrieb Ethernet/Wifi verwendet werden soll, muss die Stromversorgung um U250 bestückt sein. Sonst bitte U250, L250, D251 und C250..C258 in der BOM löschen, um Bestückkosten zu sparen.

Gleiches gilt für U4, das nur für den ESP32 benötigt wird. Wenn keiner verwendet wird, hier U4 und C65..C67 in der BOM löschen.

Verbindung zu AgOpenGPS:
USB wird immer benötigt, wenn ein ESP32 verwendet wird - mindestens zur initialen Programmierung. Ich schlage eine USB-C-Buchse vor, aber man kann auch einen [externen Connector](https://www.aliexpress.com/item/1005002859111621.html) an CN3 anschließen.

Die Kommunikation im Betrieb oder zum Programmieren (nur für den Atmel) kann entweder über USB oder RS485 erfolgen oder - wenn das Board im der gleichen Kiste wie die Zentraleinheit steckt - über Steckverbinder H10 direkt als TTL-Signal. Für RS485 muss U5 bestückt sein und U23 entweder unbestückt sein oder der Jumper "USBOFF" ist gesetzt. Falls USB nicht benötigt wird, einfach U5 in der BOM löschen. Wenn die Kommunikation über H10 erfolgt, können U23, U5 und USBC gelöscht werden, falls nicht zum Programmieren benötigt.

DP1 ist optional, um ein paar nette Dinge wie Druck, etc anzeigen zu können. 

U10..U14 haben keine Verwendung bis jetzt (können also in der BOM gelöscht werden), solange keine PWM-Leistungsentstufe existiert. Das ist was für nächstes Jahr..

Für die initiale Programmierung des Atmels (falls vorhanden) wird "PRGBOOT" verwendet. [Hier gibt es ein nettes Tutorial.](https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP).

Zu guter Letzt gibt es noch die beiden Schalter STEER und WORK, die direkt mit der Autosteer-Einheit verbunden werden können. Sonst einfach weglassen.

# Erstinbetriebnahme
1. Ggf. fehlende SMD-Bauteile [nachbestücken](https://www.youtube.com/watch?v=dLczChhmDCY)
2. gewünschte Stiftleisten und Stecker auflöten
3. langsam die Spannung an USB mit einer Strombegrenzung von 100mA bis max. 5V hochregeln und gleichzeit die 3.3V-Spannung im Auge behalten
4. +3.3V überprüfen (sollte im Bereich 3.2..3.4V sein)
5. Arduino-Uno-Board mit "PRGBOOT" verbinden und Bootloader flashen (https://www.arduino.cc/en/Tutorial/BuiltInExamples/ArduinoISP) 
6. Uno-Board abstöpseln
7. über USB mit einem PC verbinden: Der PC muss einen neue virtuelle serielle Schnittstelle auf CH340-Basis anzeigen)
8. Code wie gewohnt mit der Arduino-Software flashen
9. Sensoren anschliessen und mit AgOpenGPS testen

debugging: use terminal like HTerm and find out the correct COM port: connect to each possible one, set "DTR" and release "DTR". When releasing, the red LED blinks 3 times (bootloader). The green LED next to U3 will blink when sending any serial data to the Nano; this is also true when sending with the Arduino IDE (serial monitor)

