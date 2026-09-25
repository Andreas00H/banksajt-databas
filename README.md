# Banksajt med databas

**Publicerad sajt:** http://13.61.15.216:3002

En enkel bank byggd med Next.js (frontend) och Express (backend). I den här versionen sparas all data i en **MySQL-databas** istället för i arrayer, så att användare, konton och saldon finns kvar när servern startas om.

## Databasen

Backend skapar databasen `bank` och tabellerna automatiskt när den startar (`backend/db.js`):

| Tabell     | Kolumner                               | Innehåll                                            |
| ---------- | -------------------------------------- | --------------------------------------------------- |
| `users`    | `id`, `username`, `password`           | Användare. `username` är unikt.                     |
| `accounts` | `id`, `user_id`, `amount`              | Ett bankkonto per användare, kopplat via `user_id`. |
| `sessions` | `id`, `user_id`, `token`, `created_at` | Engångslösenord som skapas vid inloggning.          |

Alla SQL-frågor använder `?`-platshållare (prepared statements) via `mysql2`, så att användarens input aldrig klistras in direkt i SQL-koden. Det skyddar mot SQL-injektion.

## Endpoints (CRUD)

| Endpoint                         | SQL                 | Beskrivning                                          |
| -------------------------------- | ------------------- | ---------------------------------------------------- |
| `POST /users`                    | `INSERT`            | Skapar användare och ett konto med 0 kr              |
| `POST /sessions`                 | `SELECT` + `INSERT` | Loggar in och skapar ett sexsiffrigt engångslösenord |
| `POST /me/accounts`              | `SELECT`            | Visar saldot för den som äger engångslösenordet      |
| `POST /me/accounts/transactions` | `UPDATE`            | Sätter in pengar på kontot                           |

En ogiltig token ger status 401, och ett upptaget användarnamn ger status 409.

## Köra lokalt (med MAMP)

1. Starta **MAMP** (MySQL på port 8889 med användare `root` och lösenord `root`, som är standardinställningarna).
2. Installera och starta:

```bash
npm install --prefix backend
npm install --prefix frontend
npm start --prefix backend      # backend på http://localhost:3001
npm run dev --prefix frontend   # frontend på http://localhost:3000
```

Databasinställningarna kan ändras med miljövariablerna `DB_HOST`, `DB_PORT`, `DB_USER`, `DB_PASSWORD` och `DB_NAME`, till exempel i en fil `backend/.env` (som inte laddas upp till GitHub).

Testerna i `tests/` kan köras lokalt medan MAMP är igång:

```bash
npm ci --prefix tests
npm run build --prefix frontend
npm test --prefix tests
```

## VG: Databasen på AWS

Sajten och databasen körs på samma EC2-instans (Ubuntu, Europe/Stockholm):

1. Installerade MySQL på servern med `sudo apt install mysql-server`.
2. Skapade databasen `bank` och en egen databasanvändare `bankuser` med ett slumpat lösenord (istället för `root`), med behörighet bara till databasen `bank`.
3. Sparade inställningarna i `backend/.env` på servern. Filen finns bara på servern och ligger i `.gitignore`, så lösenordet hamnar aldrig på GitHub.
4. MySQL lyssnar bara på `127.0.0.1`, alltså kan databasen bara nås inifrån servern, inte från internet.
5. Överförde projektet med `rsync`, byggde frontend med `BACKEND_URL=http://127.0.0.1:3003` och startade backend (port 3003) och frontend (port 3002) med **pm2**, så att sajten fortsätter köra när terminalen stängs och startar igen om servern startas om. Port 3002 öppnades i säkerhetsgruppen.

## Förbättringar i en riktig bank

- Lösenorden sparas i klartext, precis som i lektionsexemplet. I en riktig bank skulle de hashas, till exempel med `bcrypt`.
- Engångslösenorden går aldrig ut. De borde få en giltighetstid (kolumnen `created_at` finns redan för det).
- Sajten använder `http`. En riktig bank måste använda `https`.
