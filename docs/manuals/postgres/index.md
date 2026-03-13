[← Tilbage til forsiden](../../) | [CasparCG](../casparcg/) | [Media](../mediaManagement/)| [Node](../node/) | [Blackmagic](../blackmagic/)
### Postgres Kommandoer

### Reset DB
```
npm run db:setup
```
Køre .sql filer igennem, som drop.sql (For at droppe tables), scema.sql (For at bygge tables), seed.sql (For at fylde bestemte tabeller med data)

### Tabeller

#### Users
| Kolonne | Type | Beskrivelse |
| :--- | :--- | :--- |
| `user_id` | `SERIAL` / `PK` | Unikt bruger-ID (auto-genereret) |
| `email` | `string` | Brugerens email (bruges også som login) |
| `username` | `string` | Brugernavn |
| `password_hash` | `string` | SHA-256 hashet adgangskode |