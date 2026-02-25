[← Tilbage til forsiden](../../) | [CasparCG](../casparcg/) | [Media](../mediaManagement/) | [Postgres](../postgres/) | [Node](../node/)

## Blackmagic Kommandoer

### System / Auth (Hardware status)

#### Hardware.Status(token)

**Status** returnerer den nuværende tilstand af hardware komponenterne (f.eks. DeckLink outputs). Den bruges til at overvåge systemets "helbred".

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Anvendelse**. Bruges til "Health monitors" i interfacet, for at se CPU load og frame drops.

**Eksempel**
```
>> Hardware.Status(0);
<< Response: 201 STATUS OK [CPU: 20%] [FRAMEDROPS: 1]
```

#### Hardware.Ping(token)

**Ping** er en low level netværkstest, der måler latency mellem klient og server.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Vigtigt** Høj svartid (jitter) kan få fjernstyret grafik på Blackmagic outputtet til at hakke eller køre usynkront.

**Eksempel**
```
>> Hardware.Ping(0);
<< Reponse: 200 PONG
```

#### Hardware.Time(token);

**Time** returnerer serverens interne systemtid eller den aktuelle tidskode direkte fra Blackmagic kortet.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
>> Hardware.Time(0);
<< Reponse: 200 TIME 12:00:00:00
```

### Audio Control (SDI Embedded Audio)

#### Audio.SetVolume(token, channel, volume)

**SetVolume** justerer lydstyrken på den aktive kanal

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |
|   `channel`   |   `int`   |   **Required**. Kanal (1-9) |
|   `volume`   |   `float`   |   **Required**. Værdi mellem 0.0 (Stil) og 1.0 (Fuld) |

**Eksempel**
```
Audio.SetVolume(0, 1, 0.5) // Sætter volumen til 50% på kanal 1
```

#### Audio.Mute(token, channel, state)

**Mute** arbryder øjeblikkeligt lyd-outputtet på en kanal uden at stoppe videoafspilning.

|   Parametre   |   Type    |   Beskrivelse |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |
|   `channel`   |   `int`   |   **Required**. Kanal (1-9) |
|   `state`   |   `bool`   |   **Required**. 1 for Mute, 0 for Unmute |

**Eksempel**
```
Audio.Mute(0, 1, 1); // Afbryder lyden på kanal 1
```