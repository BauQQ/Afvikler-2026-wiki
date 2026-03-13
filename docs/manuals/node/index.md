[← Tilbage til forsiden](../../) | [CasparCG](../casparcg/) | [Media](../mediaManagement/) | [Postgres](../postgres/) | [Blackmagic](../blackmagic/)
## Node

## Protocol & Communication

### Message Types
For at sikre stabil kommunikation mellem klient og server benyttes en række specifikke beskedtyper. Hver type definerer forventningen til et svar.

#### PING / PONG
Systemets "hjerteslag". Bruges til at verificere, at forbindelsen er aktiv

**Logik**: Hvis klienten sender ``PING``, skal serveren svare med ``PONG`` inden for et defineret timeout-interval.

**Fejl**: Hvis ``PONG`` udebliver, betragtes forbindelsen som tabt.

#### CALL
Anmodes når klienten vil have serveren til at udføre en aktiv handling (f.eks. ``PLAY``).

**Krav**: Et ``CALL`` forventer altid en efterfølgende ``RESP`` (Response) eller en fejlmeddelelse.
**Parametre**: Skal indeholde et token (eller 0) som valideringsparameter.

#### RESP
Serverens svar på et specifikt ``CALL``.

**Matching**: Indeholder samme unikke ID som den oprindelige ``CALL``, så klienten kan matche svaret asynkront.

#### Message-envelope
**Message-envelope** er den tekniske "indpakning" af hver besked. Den sikrer, at modtageren kan afkode beskeden korrekt.

**Struktur for en pakke**:
```
{
    "id": "unikt_sekvnes_id",
    "type": "CALL",
    "token": 0,
    "payload": { ... }
}
```

**Vigtigt** skal sikre at envelope er korrekt formateret, da serveren ellers vil throw pakken som corrupt eller ufuldstændig.

#### Error-codes
Systemet benytter numeriske koder til hurtig fejlidentifikation.

|   Kode   |   Navn    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `400`   |   ERROR   |   Kommando ikke forstået  |
| `402`    |   ERROR   |   Parameter mangler   |
| `403`    |   ERROR   |   Ulovlig parameter   |
| `404`    |   ERROR   |   Medie fil ikke fundet  |
| `500`    |   FAILED   |   Intern server fejl   |
| `501`    |   FAILED   |   Medie fil ikke læsbar   |
| `503`    |   FAILED   |      Ingen adgang til filen eller ressourcen |


## Socket Architecture

### Compression (Pako)
For at sikre lav latency og minimal Bandwidth benytter systemet binær komprimering

**Format**: GZIP / Deflate via ``PAKO``
**Logik**: Alle beskeder dekomprimeres (inflates) ved modtagelse og komprimeres ``(deflate/gzip)`` før afsendelse. Vigtigt at man ikke kan læse rå pakker i en sniffer uden at dekomprimere dem først.

### Dual-Socket Streams
Klienten opretter to parallelle forbindelser for at separere kontrol instrukser fra tung data.

**1. Control Socket ``(/control)``**: Håndtere alle ``Client_Control_Commands`` og ``status_beskeder``. Det er her vores ``pipe-separerede protokol`` kører.

**2. Stream Socket ``(/stream)``**: Dedikeret til rå binære datastrømme, såsom video thumbnails eller frames, for at undgå at blokere kontrol kommandoer.

### Connection Lifecycle
Systemet er designet til at være fejl tolerant.

**Auto-Reconect**: Hvis forbindelsen skulle tabes, forsøger clienten automatisk at genoprette forbindelse hver 3. sekund.

**Offline Queue**: Hvis en kommando sendes, mens klienten er offline, gemmes den i en intern kø (``this.queue``) og sendes automatisk, så snart forbindelsen er aktiv igen (``flushQueue``).

### System / Auth

#### Auth.Hello()
**Hello** er det indledende digitale håndtryk.

**Formål**: Bekræfter at serveren er klar til at modtage instruktioner.

**Logik**: Hvis serveren ikke svare på ``Hello``, skal klienten afbryde forbindelsen og forsøge genopretning.

#### Auth.Identify(clientType, version, token)
**Identify** udveksler information om klientens identitet og version

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `clientType`   |   `string`   |   **Required**. Typen af klient (f.eks. "Admin" eller "Monitor")  |
| `version`    |   `string`   |   **Required**. Klientens nuværende versionnummer   |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Auth.Identify("Admin-control", "1.0.2");
```

#### Auth.Login(email, password);
**Login** håndtere adgangskontrollen. De fleste andre kommandoer afvises, indtil login er godkendt.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `email`   |   `string`   |   **Required**. Brugerens unikke email  |
| `password`    |   `string`   |   **Required**. Brugerens adgangskode eller token   |

**Eksempel**
```
Auth.Login("admin@test.com", "p@ssword123");
```

#### Auth.Logout(token);

**Logout** afslutter sessionen og frigiver ressourcer.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

### Responses

#### ACK
**ACK** er en positiv bekræftelse på, at en kommando er modtager og accepteret af serveren. 

**Eksempel**
```
>> PLAY 1-10
<< ACK 202 PLAY OK
```

#### NACK
**NACK** er en negativ bekræftelse på, at en kommando er afvist.

**Eksempel**
```
>> PLAY 1-10
<< NACK 404 PLAY ERROR
```

#### RESULT
**RESULT** følger typisk efter en `ACK` og indeholder de faktiske data, som klienten har anmodet om.

**Eksempel**
```
>> CALL VERSION
<< ACK 200
<< RESULT 200 VERSION 2.3.3.fba123
```

## Playlist Management

### Controller.LoadPlaylist(filename, token);

**LoadPlaylist** indlæser en prædefineret fil (JSON/XML) med medie-elementer og metadata. Den forbereder klientens kø uden at aktivere hardware outputs.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på playliste filen |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Controller.LoadPlaylist("playlist.json", 0)
```

### Controller.StartPlaylist(token)

**StartPlaylist** aktiverer afviklingen af den nuværende kø. Den trigger det første element og starter de nødvendige outputs på CasparCG og Blackmagic hardwaren.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

### Controller.StopPlaylist(token)

**StopPlaylist** afbryder al afvikling øjeblikkeligt. Den rydder hele afviklings-køen og fungerer som en "Panic Button", der bringer alle lag i sort eller til default state.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

### Controller.UpdatePlaylist(itemId, categoryId, timestamp, token)

**UpdatePlaylist** bruges når et eksisterende element i playlisten flyttes eller ændres.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `itemId`   |   `int`   |   **Required**. Unikt ID for elementet  |
| `categoryId`    |   `int`   |   **Required**. Kategoriens ID   |
| `timestamp`    |   `string`   |   **Required**. Tidspunkt for afvikling   |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Controller.UpdatePlaylist(101, 2, "18:30:00", "0")
```

### Controller.AddToPlaylist(itemId, categoryId, timestamp, token)

**AddToPlaylist** benyttes ved Drag & Drop fra biblioteket ind i playlisten.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `itemId`   |   `int`   |   **Required**. Unikt ID for elementet  |
| `categoryId`    |   `int`   |   **Required**. Kategoriens ID   |
| `timestamp`    |   `string`   |   **Required**. Tidspunkt for afvikling   |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Controller.AddToPlaylist(202, 2, "18:45:00", "0")
```

### Controller.RemoveFromPlaylist(itemId, categoryId, timestamp, token)

**RemoveFromPlaylist** fjerner et element fra playlist og opdaterer serverens status

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `itemId`   |   `int`   |   **Required**. Unikt ID for elementet  |
| `categoryId`    |   `int`   |   **Required**. Kategoriens ID   |
| `timestamp`    |   `string`   |   **Required**. Tidspunkt for afvikling   |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Controller.RemoveFromPlaylist(101, 2, "18.30.00", "0");
```

###  Transport & Playback Control

#### Controller.Next(token)

**Next** tvinger afviklingen videre til det næste element i playlisten øjeblikkeligt.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Bemærk** For at sikre et rent skift på Blackmagic outputtet, skal klienten sende en `STOP` til det nuværende lag og en `PLAY` til det næste simultant

**Eksempel**
```
>> Controller.Next(0);
<< Reponse: 200 OK NEXT [PLAYING: ITEM_02]
```

#### Controller.Seek(token, position, [unit]);

**Seek** flytter afspilningshovet (playhead) til et specifikt tidspunkt eller frame i det nuværende medie.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |
|   `position`   |   `int`   |   **Required**. Tidspunkt eller frame nummer |
|   `unit`   |   `string`   |   **Optional**. Definerer enheden: `"seconds"` eller `"frames"` (Default: `"seconds"`) |

**Eksempel**
```
Controller.Seek(0, 10, "seconds");
```

#### Controller.Current(token);

**Current** forespørger om status på det medie, der afspilles netop nu. Bruges primært til at holde interfacet og progress barer synkroniseret.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
>> Controller.Current(0);
<< Reponse: 200 OK ["FILE": "CLIP_A.mp4", "FRAME": 150, "TOTAL": 1000]
```
> Token eller 0 på alle vores interne kommandoer

## System Internals

### Logger

Systemets interne logger skriver til konsollen og til en logfil på disk.

#### Konfiguration
Logger skal konfigureres én gang ved opstart:
```javascript
Logger.configure({ logFile: "path/to/server.log" });
```

#### Log-niveauer

| Metode | Beskrivelse |
| :--- | :--- |
| `Logger.info(message)` | Generel information |
| `Logger.warning(message)` | Advarsler |
| `Logger.error(message)` | Fejl |

**Timestamps** formateres i `Europe/Copenhagen` timezone som `YYYY-MM-DDTHH:mm:ss.SSS`.

#### Log-rotation
Logfiler roteres automatisk ved midnat. Den gamle fil omdøbes med datoen (`server-2026-03-13.log`) og komprimeres til `.gz`. En ny tom logfil oprettes automatisk.

---

### Socket

WebSocket-serveren kører over **WSS (WebSocket Secure)** og kræver SSL-certifikat konfigureret i `global.config.socket`.

Alle beskeder komprimeres med **PAKO** (deflate/inflate). Se også [Compression](#compression-pako) sektionen.

Hver klient tildeles et **unikt numerisk ID** ved forbindelse og registreres i `global.variables.clients`:
```javascript
{
    socket: socket,
    id: "12345",
    page: null,
    connectedAt: Date.now()
}
```

---

### Command Handler

Command Handleren er den centrale router, der modtager beskeder fra klienter og videresender dem til det korrekte modul.

#### Protokolformat

Alle kommandoer sendes som **pipe-separerede strenge**:
```
KOMMANDO|DATA|TOKEN
```

For at kalde en specifik funktion i et modul:
```
KOMMANDO>FUNKTION|DATA|TOKEN
```

| Del | Beskrivelse |
| :--- | :--- |
| `KOMMANDO` | Navn på det modul der skal kaldes (f.eks. `auth`, `controller`) |
| `>FUNKTION` | **Valgfri**. Specifik funktion i modulet. Default er `f` |
| `DATA` | Data til kommandoen. Mellemste dele joinnes automatisk |
| `TOKEN` | Valideringstoken. Brug `0` for standard |

**Eksempel**
```
auth>login|admin@test.com|p@ssword123|0
controller|Play|1|1|Nyheder|0
```

---

### Cryptography

| Metode | Beskrivelse |
| :--- | :--- |
| `generateID()` | Genererer et tilfældigt numerisk ID |
| `passwordHash(password)` | Hasher en adgangskode med **SHA-256** |

---

### DB (PostgreSQL)

Databaseforbindelsen bruger en **connection pool** via `pg`-biblioteket.

Konfigureres via `global.config.database` eller miljøvariabler:

| Miljøvariabel | Beskrivelse |
| :--- | :--- |
| `PGHOST` | Database host |
| `PGPORT` | Port (default: `5432`) |
| `PGUSER` | Brugernavn |
| `PGPASSWORD` | Adgangskode |
| `PGDATABASE` | Database navn |

Standard max pool størrelse er **10 forbindelser**.

**Eksempel**
```javascript
const result = await db.query('SELECT * FROM users WHERE id = $1', [userId]);
```

---

### IO

IO-klassen håndterer port-tjek og frigivelse ved opstart.

| Metode | Beskrivelse |
| :--- | :--- |
| `IO.isPortFree(port)` | Returnerer `true` hvis porten er ledig |
| `IO.freePort(port)` | Tjekker porten — forsøger automatisk at frigøre den hvis den er i brug |

---

### SPL (Module Loader)

SPL indlæser automatisk alle moduler fra `modules/`-mappen rekursivt ved server-opstart.

**Eksempel — find et modul**
```javascript
const loginModule = await spl.find("auth/login");
```

Moduler addresseres med **skråstreg-separeret sti** relativt til `modules/`-mappen.

#### Server opstartrækkefølge

Serveren starter i følgende rækkefølge:

1. **Libs** — Eksterne biblioteker indlæses fra `global.config.libs` og registreres på `global.libs`
2. **Logger** — Logfilen initialiseres og registreres som `global.Log`
3. **Kernel** — Alle kernel-moduler indlæses parallelt og registreres som `global.[modulNavn]`
4. **SPL** — Alle applikationsmoduler indlæses fra `modules/`-mappen
5. **Port** — Den konfigurerede socket-port frigøres via `global.IO.freePort()`
6. **Socket** — WebSocket-serveren bygges og starter med at lytte

Hvis et trin fejler, afbrydes processen med `process.exit(1)`.