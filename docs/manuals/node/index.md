## CasparCG Controller Kommandoer

`Play, Stop, Clear, Resume, Pause, Info`

#### Controller.Play(channel, layer, clip);

**Play** sender et medie (Video, Billede eller Template) direkte til output-køen på et specifikt lag. Hvis noget allerede køre på laget, så vil **Play** som standard erstatte det gamle indhold.

|   Parameter   |   Type    |   Description                                                     |
| :------------ | :--------:| :---------------------------------------------------------------- |
|   `channel`   |   `int`   |   **Required**. Den kanal(1-9), som mediet skal afspilles på.     |
| `layer`       |   `int`   |   **Required**. Det lag, som mediet skal ligge på.                |
|   `clip`      |   `string`|   **Required**. Navnet på filen i medie mappen (Uden filendelse)  |
| `token`| `string`| **Required**. Token eller 0

Token eller 0 på alle vores interne kommandoer