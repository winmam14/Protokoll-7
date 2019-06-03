# Protokoll-7  
## Thema: Temperatursensor Auswertung
**Professor:** SX  
**Übungsdatum:** 02.04.2019  
**Author, KNr.:** Winter Matthias, 17    
**Anwesend:** Alois Vollmaier, Patrick Wegl, Winter Matthias, Winter Thomas  
**Abwesend:** Sarah Vezonik, Mercedes Wesonig  

---

## Themenübersicht 

### 1. Temperatursensor am µC
### 2. Programmiervorlage
### 3. Aufgabenstellung
### 4. Quelltextzeilen zur Auswertung vom Temperatursensor
#### 4.1 app.c
#### 4.2 app.h


--- 

## 1. Temperatursensor am  µC
Im Unterricht verwenden wir den Arduino Nano welcher mit einem ATmega328p ausgestattet ist. Dieser besitzt einen eingebauten Temperatursensor. Diesen haben wir versucht über Modbus-ASCII auszulesen.
    
  ***Beliebte **AVR-Chips**, die über einen internen Temperatursensor verfügen:***

**ATmega8 :** Nein  
**ATmega8L :** Nein   
**ATmega168A :** Ja  
**ATmega168P :** Ja  
**ATmega328 :** Ja  
**ATmega328P :** Ja  
**ATmega2560 (Arduino Mega 2560) :** Nein  

## 2. Programmiervorlage

Im unterricht verwenden wir eine **Programmiervorlge** in der alle grundlegenden Funktionen eingebaut sind die in den meisten Programmen benötigt werden. Doch falls einmal weniger Funktionen benötigt werden, zum Beispiel für kleinere Übungsprogramme, gibt es verschiedene Kategorien von Programmiervorlagen (Level 1-Level 4). Je höher das Level desto mehr Funktionen wurden vom Entwickler implementiert.  Weniger **Quelltext** bedeutet **schnellerer upload** in den Speicher des µC sowie **geringerer Platzbedarf** im Speicher.    
  
  ***Übersicht AIIT-Programmiervorage, Level 1-4:***  
  

|  Programmiervorlage  |  Inhalt  |
|------------------------|--------------------|
|Level 1 |keine Inhalte, empty |
|Level 2 |Komunikation über UART  |
|Level 3 |Level 2, Tasks |
|Level 4 |level 3, Debug möglichkeiten |

## 3. Aufgabenstellung
In dieser Einheit war das Ziel einen Temperatursensor, welcher am Microcontroller verbaut ist, aus zu lesen und auf der Konsole auszugeben. Hierfür gibt es mehrere Lösungsansätze. Als erstes mussten wir entscheiden wie die Verbindung zwischen Microcontroller und Teminal aufgebaut wird.  
  Wir haben uns für eine Kabelgebundene Übertragung entschieden. Somit wussten wir, dass wir über einen **UART/USB** konverter, welcher am Arduino Nano bereits verbaut ist, die verbindung über USB mit dem PC herstellen müssen. Danach haben wir uns dazu entschieden **Modbus-ASCII** als Komunikationsprotokoll festzulegen. Anschießend konnten wir das benötigte Programm dafür schreiben.  
  Die Temperaturwerte werden als **16Bit Werte** übertragen. Weiters werden die werte in **Festkommacodierung** übertragen somit sind **links** und **rechts** vom Komma 8 Bit. Um nun vom Temperaturwert z.B 23,5°C zum **Hexwert** zu kommen muss man den wert zuerst mit 256 Multiplizieren und danach in eine **Hexadezimalzahl** umwandeln -> 23,5 * 256 = 6016 => 1780hex  
    
   Nachdem wir dass Programm soweit Lauffähig hatten konnten wir es testen. Dort fiel auf dass **falsche** werte in der Konsole ausgegeben wurden. Dies war jedoch kein großes Problem, da der Fehler statisch war und wir ihn somit durch **einfaches Kallibrieren** beheben konnten.


## 4. Quelltextzeilen zur Auswertung vom Temperatursensor
Im nächsten Punkt werden einige wichtige Zeilen aus dem Quelltext zur **Auswertung vom Temperatursensor** gezeigt. Wichtig dabei ist, dass wir eine **Programmiervorlage** verwendet haben! Somit war die Programmierung unkomplizierter, da die grundlegenden Teile eines Programms schon vorhanden waren. Für uns waren nur die Dateien: **app.c**, **app.h** von Interesse, welche auch in den Punkten 4.1 sowie 4.2 genauer erklärt werden.


### 4.1 app.c
```c
  {
    memset((void *)&app, 0, sizeof(app));
    ADMUX = (1 << REFS1) | (1<< REFS0) | ( 1<< ADLAR) | 0x08;
    ADCSRA = (1<< ADEN) | 7; // 7 bedeutet durch 128, dass sind 125 kHz
    app.modbus.frameIndex = -1;
  }


  void app_task_16ms (void) {

    app.adch = ADCH;
    ADCSRA |= (1<< ADSC); //Starte ADC
  }
  

  void app_handleUartByte(char c){
    struct Modbus *p = &app.modbus;
    
  if (p ->frameIndex < 0)
  {
    return;
  }

  if( c == ':')
  {
    p ->frameIndex = 0;
    p->frameError = 0;
  }

  else if ( c == '\n'){
  if (p->frameError == 0){
    app_parseModbusFrame();
    }
  }


  else if (p ->frameIndex < 16){
    p ->frameIndex [p ->frameIndex] = c;
  }


  else if (p -> errCnt == 0){
    p->frameError = 1;
      
  if(p->errCnt < 0xffff){
    p-> errCnt++;
    }
  }
}
```  
Die Referenzspannung für den Analog-Digital-Wandler kann durch die Bits **REFS1** und **REFS0** im **ADMUX**-Register ausgewählt werden, die Referenzspannung liegt dann auch am **AVCC Pin** an. Möglich sind **VCC** oder die interne Referenzspannung von **2,56V**.  
  
  Der Analog-Digital-Wandler erzeugt ein 10-bit Ergebnis, das in den ADC Data Registern **ADCH und ADCL** abgelegt wird. Normalerweise wird das Ergebnis rechtsbündig in den beiden Registern abgelegt, optional kann das Ergebnis aber auch linksbündig in **ADCH und ADCL** geschrieben werden. Die Einstellung erfolgt mit dem **ADLAR**-Bit im **ADMUX**-Register.
  
  
  
  
 ### 4.2 app.h    
       
```c
struct Modbus // Struktur um alle Komponenten
{
    char frame[16];
    int8_t frameIndex;
    uint16_t frameError;
    uint16_t errCnt;
};

struct App
{
    uint8_t adch;
    struct Modbus modbus;
};
```

In der Headerdatei werden Structuren deklariert die für unser Programm notwendig sind um unser Programm möglichst übersichtlich zu machen! In der Struktur **Modbus** sieht man die Variablendeklaration: **uint16_t errCnt**. Dies bedeutet, dass diese Variable einen **16-bit, vorzeichenlosen ganzzahligen Wert** annehmen kann. Diese Variable wird verwendet um aufgetretene Fehler mit zu zählen.  

|  Variablenbezeichnung  |  Verwendungszweck  |
|------------------------|--------------------|
|char frame[16]          |Temperaturwert welcher zur übertragung mit dem Modbusprotokoll geeignet ist (Frame)  |
|int8_t frameIndex       |eine Stelle vom Frame  |
|uint16_t frameError     |Fehler-Frame|
|uint16_t errCnt         |um aufgetretene Fehler mit zu zählen  |

