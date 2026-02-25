## Media Management

#### Media.Upload(filename)
**Upload** initierer overførsel af en mediefil fra klienten til serverens medie-mappe.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |

**Bemærk**. Interfacet bør lytte på ´Progress´-events for at visualiserer upload-status for brugeren.

**Eksempel**
```
Media.Upload("INTRO_V2.mp4")
```

#### Media.Delete(filename)
**Delete** fjerner en fil permanent fra serverens lager.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |

> [!WARNING]
> Dette er en destruktiv handling. Det anbefales at implementere en "Er du sikker" dialog i UI'et

**Eksempel**
```
Media.Delete("INTRO_V2.mp4");
```

#### Media.Move(source, destination)
**Move** omdøber eller flytter filer mellem mapper på serveren.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `source`   |   `string`   |   **Required**. Den nuværende sti/filnavn  |
| `destination`    |   `string`   |   **Required**. Den nye sti/filnavn    |

**Vigtigt**. Flytning af filer kræver opdatering af playlister for at undgå `404 Media file not found`

**Eksempel**.
```
Media.Move("incoming/clip.mp4", "archive/clip.mp4");
```

#### Media.Scan()
**Scan** tvinger serveren til at genindlæse medie-mapper og opdatere sin interne database.

**Anvendelse**. Bør altid kaldes efter **Upload** eller **Move** for at sikre, at filerne er synlige for klienten.

#### Media.List()
**List** returnerer en liste over alle registrerede mediefiler. Bruges til at populere **Media Browser** eller **FilePicker** interfacet.

**Eksempel**
```
>>Media.List()
<< Returnerer: ["video1.mp4", "logo.png", "folder/clip.mov"]
```

#### Media.Info(filename)
**Info** henter dybdegående metadata (codec, framerate, opløsning) om en specifik fil. Bruges fx til at tjekke for mismatch mellem filens framerate og hardwarens output.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `filename`   |   `string`   |   **Required**. Navnet på filen inkl filendelse  |

#### Media.Search(query)
**Search** søger efter filer ved hjælp af nøgleord eller wildcards.

|   Parameter   |   Type    |   Description |
| :------------ | :--------: | :------------ |
|   `query`   |   `string`   |   **Required**. Søgestreng  |

**Eksempel**
```
Media.Search("INTRO")
```