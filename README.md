# Demonstracija web sigurnosnih ranjivosti

Full-stack web aplikacija koja na interaktivan način demonstrira dvije česte sigurnosne ranjivosti u web aplikacijama. Svaka ranjivost može se uključiti ili isključiti putem prekidača u sučelju, čime se u stvarnom vremenu uspoređuje ponašanje ranjive i zaštićene verzije iste funkcionalnosti.

## Tehnologije

**Frontend**
- React 19
- Vite 7
- Tailwind CSS 4

**Backend**
- Node.js (ES moduli)
- Express 5
- PostgreSQL (`pg`)
- `sanitize-html` – sanitizacija HTML sadržaja
- Node.js `crypto` modul – AES-256-CBC enkripcija
- `dotenv`, `cors`

**Razvoj**
- `concurrently` – paralelno pokretanje frontend i backend dev servera

## Značajke

### 1. Cross-Site Scripting (Stored XSS)
Modul omogućava dodavanje komentara koji se trajno pohranjuju u bazu podataka. Prekidačem se bira između dva načina rada:
- **Ranjivo** – komentar se pohranjuje i prikazuje bez sanitizacije, pa umetnuti `<script>` ili `onerror` kod izvršava proizvoljni JavaScript u pregledniku svakog posjetitelja.
- **Zaštićeno** – sadržaj komentara se prije pohrane sanitizira (`sanitize-html`), čime se uklanjaju svi HTML tagovi i atributi.

### 2. Nesigurna pohrana osjetljivih podataka
Modul omogućava unos osobnih podataka (ime, prezime, korisničko ime, lozinka, broj kreditne kartice). Prekidačem se bira način pohrane:
- **Ranjivo** – lozinka i broj kreditne kartice pohranjuju se u bazu kao obični (plain text) podaci.
- **Zaštićeno** – isti podaci se prije pohrane šifriraju AES-256-CBC algoritmom.

Tablica s podacima iz baze prikazuje točan (sirovi) sadržaj zapisa te oznaku je li pojedini zapis pohranjen sigurno ili nesigurno, čime je razlika između dva pristupa vidljiva iz baze podataka, a ne samo iz sučelja.

## Arhitektura

Aplikacija se sastoji od dva dijela:
- `client/` – React aplikacija (Vite) koja komunicira s backendom preko REST API-ja.
- `server/` – Express server koji izlaže API rute, upravlja konekcijom prema PostgreSQL bazi i u produkcijskom načinu rada posužuje izgrađenu klijentsku aplikaciju (`client/dist`).

### API rute

| Metoda | Ruta | Opis |
|---|---|---|
| `GET` | `/api/comments` | Dohvat svih komentara |
| `POST` | `/api/comments` | Dodavanje komentara (s opcijom sanitizacije) |
| `DELETE` | `/api/comments` | Brisanje svih komentara |
| `POST` | `/api/sensitive-data` | Pohrana osjetljivih podataka (s opcijom enkripcije) |
| `GET` | `/api/sensitive-data` | Dohvat pohranjenih osjetljivih podataka |
| `GET` | `/api/health` | Provjera statusa servera i baze podataka |

## Pokretanje projekta

### Preduvjeti
- Node.js 18+
- Pristup PostgreSQL bazi podataka

### Postavljanje okruženja
Kopirati `.env.example` u `.env` i postaviti vrijednosti:

```
DATABASE_URL=postgresql://user:password@host/baza
PORT=3001
NODE_ENV=development
```

### Instalacija ovisnosti

```bash
npm install
cd client && npm install
```

### Razvojni način rada
Pokreće backend (port 3001) i frontend (port 5173) istovremeno, uz proxy `/api` zahtjeva prema backendu:

```bash
npm run dev
```

### Produkcijski način rada

```bash
npm run build
npm start
```

Struktura baze podataka (tablice `comments` i `sensitive_data`) automatski se inicijalizira prilikom pokretanja servera.

## Napomena

Ova aplikacija namjerno sadrži sigurnosne ranjivosti u edukativne svrhe. Ranjive verzije funkcionalnosti ni u kojem slučaju ne bi trebale biti korištene u produkcijskim aplikacijama koje obrađuju stvarne korisničke podatke.
