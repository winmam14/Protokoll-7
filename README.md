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
### 3. Quelltextzeilen zur Auswertung vom Temperatursensor


--- 

## 1. 






## 3. Quelltextzeilen zur Auswertung vom Temperatursensor
Im nächsten Punkt werden einige wichtige Zeilen aus dem Quelltext zur **Auswertung vom Temperatursensor** gezeigt. Wichtig dabei ist, dass wir eine **Programmiervorlage** verwendet haben! Somit war die Programmierung unkomplizierter, da die grundlegenden Teile eines Programms schon vorhanden waren. Für uns waren nur die Dateien: **app.c**, **app.h**, **sys.c** von Interesse, welche auch in den Punkten 3.1; 3.2 sowie 3.3 genauer erklärt werden.


### 3.1 app.c
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
  
  
  
  
 ### 3.2 app.h    
       
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

### 3.3 sys.c

```c
ISR (SYS_UART_RECEIVE_VECTOR)
{


static uint8_t lastChar;


uint8_t c = SYS_UDR;


if (c=='R' && lastChar=='@')


{


wdt_enable(WDTO_15MS);


wdt_reset();


while(1) {};


}


lastChar = c;


app_handleUartByte(c);

```
