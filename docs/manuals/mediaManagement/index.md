[← Tilbage til forsiden](../../) | [CasparCG](../casparcg/) | [Postgres](../postgres/) | [Node](../node/) | [Blackmagic](../blackmagic/)
## Media Management

#### Media.Upload(filename, token)
**Upload** initierer overførsel af en mediefil fra klienten til serverens medie-mappe.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Bemærk**. Interfacet bør lytte på ´Progress´-events for at visualiserer upload-status for brugeren.

**Eksempel**
```
Media.Upload("INTRO_V2.mp4", "0");
```

#### Media.Delete(filename, token)
**Delete** fjerner en fil permanent fra serverens lager.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Bemærk**
Dette er en destruktiv handling. Det anbefales at implementere en "Er du sikker" dialog i UI'et

**Eksempel**
```
Media.Delete("INTRO_V2.mp4", "0");
```

#### Media.Move(source, destination, token)
**Move** omdøber eller flytter filer mellem mapper på serveren.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `source`   |   `string`   |   **Required**. Den nuværende sti/filnavn  |
| `destination`    |   `string`   |   **Required**. Den nye sti/filnavn    |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Vigtigt**. Flytning af filer kræver opdatering af playlister for at undgå `404 Media file not found`

**Eksempel**.
```
Media.Move("incoming/clip.mp4", "archive/clip.mp4", "0");
```

#### Media.Scan(token)
**Scan** tvinger serveren til at genindlæse medie-mapper og opdatere sin interne database.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Anvendelse**. Bør altid kaldes efter **Upload** eller **Move** for at sikre, at filerne er synlige for klienten.

#### Media.List(token)
**List** returnerer en liste over alle registrerede mediefiler. Bruges til at populere **Media Browser** eller **FilePicker** interfacet.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
>> Media.List()
<< Returnerer: ["video1.mp4", "logo.png", "folder/clip.mov"]
```

#### Media.Info(filename, token)
**Info** henter dybdegående metadata (codec, framerate, opløsning) om en specifik fil. Bruges fx til at tjekke for mismatch mellem filens framerate og hardwarens output.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

#### Media.Search(query, token)
**Search** søger efter filer ved hjælp af nøgleord eller wildcards.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `query`   |   `string`   |   **Required**. Søgestreng  |
|   `token`   |   `string`   |   **Required**. Validerings token (0 for standard) |

**Eksempel**
```
Media.Search("INTRO", "0")
```