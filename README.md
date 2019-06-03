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
### 3. Programm zur Auswertung vom Temperatursensor


--- 

## 1. 








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
