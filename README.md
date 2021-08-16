# smartwatering
Dieses Projekt entstand im Rahmen meiner Bachelorarbeit. Es soll dabei helfen die Bewässerung des Rasens effizienter zu gestalten. Das System wurde unter den Gesichtspunkten: Robusheit, Erweiterbarkeit, Integrierbarkeit und Preiswertigkeit entwickelt.

Ein robustes System zu entwickeln, beginnt mit einem möglichst einfachen Aufbau. Grundlage stellt hierbei eine zuverlässige Energieversorgung dar, die jederzeit eine ausreichende Versorgung der Bauteile bereitstellt. Um dies zu gewährleisten, dient ein 5 Ah Blei-Gel Akku zur Speicherung der Energie. Dieser wird über ein 20 Watt Solarpanel versorgt. Um die Bauteile vor Wasser zu schützen, wird ein wasserdichtes Gehäuse, nach IP65-Standard, genutzt. Bei der Wahl des Bodenfeuchtesensors wurde darauf geachtet, dass dieser bei Langzeittests keine Korrosion aufweist.
Durch die Nutzung eines ESP8266 Mikrocontrollers können bis zu 6 Wasserkreisläufe genutzt werden und in Kombination mit dem Analog-Digital-Wandler ADS1115 können gleichzeitig bis zu 5 verschiedene Bodenfeuchtesensoren angeschlossen sein. Dies sorgt für eine Erweiterbarkeit, die wohl für die meisten Gärten ausreicht. Durch die Wahl eines größeren Mikrocontrollers, beispielsweise eines ESP 32, könnte die Zahl bei Bedarf signifikant erhöht werden.
Die Kommunikation über ein MQTT-Protokoll stellt sicher, dass das Bewässerungssystem in sämtliche bestehende Smart-Home Systeme einfach zu integrieren ist.
Bei der Auswahl der Bauteile wurde stets auf einen möglichst geringen Preis geachtet, der jedoch keineswegs eine minderwertige Qualität in Kauf nimmt. 
Die Systemarchtiektur verdeutlicht die Kommunikation der Komponenten in einer vereinfachten Darstellung.
Die Bodenfeuchte wird analog an den Mikrocontroller übertragen. Dieser berechnet aus dem analogen Wert eine tatsächliche Bodenfeuchte und übermittelt sie, mithilfe einer MQTT-Datenübertragung, an den Server des Iobrokers. Die Ansteuerung des Magnetventils geht vom Mikrocontroller aus, kann jedoch auch manuell über die Visualisierung erfolgen. Normalerweise öffnet der Mikrocontroller die Magnetventile bei Unterschreiten eines Schwellwerts von 22 %. Zwischen Magnetventil und ESP8266 befindet sich eine Transistorschaltung, welche das Schalten der Ventile ermöglicht. Diese wie auch der gesamte Grundaufbau des Systems sind in der Schaltskizze ersichtlich.

Viel Spaß beim Nachbauen!

