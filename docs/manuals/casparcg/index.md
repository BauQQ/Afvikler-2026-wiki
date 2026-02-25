[← Tilbage til forsiden](../../) | [Media](../mediaManagement/) | [Postgres](../postgres/) | [Node](../node/) | [Blackmagic](../blackmagic/)
## CasparCG Controller Kommandoer

`Play, Stop, Clear, Resume, Pause, Info`

#### Controller.Play(channel, layer, clip);

**Play** sender et medie (Video, Billede eller Template) direkte til output-køen på et specifikt lag. Hvis noget allerede køre på laget, så vil **Play** som standard erstatte det gamle indhold.

|   Parameter   |   Type    |   Description                                                     |
| :------------ | :--------:| :---------------------------------------------------------------- |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som mediet skal afspilles på.     |
| `layer`       |   `int`   |   **Required**. Det lag, som mediet skal ligge på.                |
|   `clip`      |   `string`|   **Required**. Navnet på filen i medie mappen (Uden filendelse)  |

**Eksempel**
```
Controller.Play(1, 1, Nyheder);
```

### Controller.Stop(channel, layer);

**Stop** stopper afspilningen på det specifikke lag med det samme

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som skal stoppes  |
| `layer`    |   `int`   |   **Required**. Det lag der skal stoppes    |

**Eksempel**
```
Controller.Stop(1, 1);
```
Stopper alt på layer 1 på kanal 1

### Controller.Clear(channel, [layer]);

**Clear** rydder alt fra en kanal, eller fra et bestemt lag.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som skal ryddes  |
| `layer`    |   `int`   |   **Optional**. Det lag der skal ryddes    |

**Eksempel**
```
Controller.Play(1); // Ryder alt fra kanal 1
Controller.Play(1, 1;) // Rydder alt fra lag 1 på kanal 1
```

### Controller.Pause(channel, [layer]);

**Pause** pauser afspilningen af det nuværende medie på den angivne kanal eller specifikt lag.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som skal paues  |
| `layer`    |   `int`   |   **Optional**. Det lag der skal pauses    |

**Eksempel**
```
Controller.Pause(1); // Pauser alt fra kanal 1
Controller.Pause(1, 1); // Pauser alt fra lag 1 på kanal 1
```

### Controller.Resume(channel, [layer]);

**Resume** Genoptager afspilningen af et medie, der tidligere er blevet sat på pause.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som skal genoptages  |
| `layer`    |   `int`   |   **Optional**. Det lag der skal genoptages    |

**Eksempel**
```
Controller.Resume(1); // Genoptager alt fra kanal 1
Controller.Resume(1, 1); // Genoptager alt fra lag 1 på kanal 1
```

### Controller.Info([channel], [layer]);

**Info** Henter statusoplysninger fra CasparCG serveren. Komandoen tilpasser sit svar alt efter, hvor specifikt man forespørger.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `channel`   |   `int`   |   **Optional**. Den specifikke kanal(1-9)   |
| `layer`    |   `int`   |   **Optional**. Det specifikke lag.

**Eksempel**
```
// 1. Hent alle kanaler (Tjekker om serveren er online og konfigureret)
>> Controller.Info();
<< 200 INFO OK
<< 1 720p5000 PLAYING
<< 2 PAL PLAYING

// 2. Hent status for en specifik kanal
>>Controller.Info(1);
<< 200 INFO OK (Kanal specifik data returneres her)

// 3. Hent status for et specifikt lag
>>Controller.Info(1, 1);
<< 200 INFO OK (Lag specifik XML data returneres her)
```

#### Standard Returkoder

Alle kommandoer returnerer en statuskode, der bekræfter om handlingen lykkedes:

**202 [COMMAND] OK**. Kommandoen blev modtager og udført korrekt
<br />
**404 NOT FOUND**. Den efterspurgte kanal eller mediefil blev ikke fundet
<br />
**500 ERROR**. Der opstod en fejl på serveren (Tjek log)

## System & Administration

#### Command.Capabilities()
**Capabilities** henter en liste over hardwarens og serverens understøttede formater og funktioner.

**Anvendelse**. Bruges ved app-opstart til dynamisk at konfigurere interfacet (f.eks. ved at deaktivere 4K-knapper, hvis hardwaren ikke understøtter det)

**Eksempel**
```
>> Command.Capabilities()
<< 200 OK
<< [FORMATS: 1080i500, 1080p250] [CONSUMERS: DECKLINK, SCREEN]
```

#### Command.Restart()
**Restart** Tvinger CasparCG-serveren til en fuld genstart

> **Bemærk** Hard Reset. Denne kommando afbryder alle aktive signlaer og tømmer hukommelsen.

**Eksempel**
```
>> Command.Restart()
<< 202 RESTARTING
```

