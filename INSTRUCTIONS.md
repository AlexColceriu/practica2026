# INSTRUCTIONS.md - Web Scraper pentru Locuri de Muncă

## Scop
Acest document oferă instrucțiuni generale pentru dezvoltarea unui web scraper Node.js/JavaScript care extrage job-uri conform Job Model și Company Model din proiectul peviitor.ro.

## Tech Stack
- **Runtime:** Node.js (LTS)
- **Language:** JavaScript
- **HTTP Client:** `axios` sau `node-fetch`
- **HTML Parsing:** `cheerio`
- **Browser Automation (opțional):** `puppeteer` pentru site-uri cu JavaScript
- **Data Validation:** `zod` sau `joi`
- **Logging:** `winston` sau `console` standard
- **Output:** JSON

## Structura Proiectului
```
scraper/
├── package.json
├── config.js          # Configurații (URL, delay, timeout)
├── scraper.js         # Logică principală
├── parser.js          # Parsing HTML → Job/Company model
├── validator.js       # Validare date conform model.md
├── output/            # Fișiere JSON rezultate
└── README.md
```

## Instrucțiuni de Implementare

### 1. Configurare
```javascript
// config.js
module.exports = {
  baseUrl: 'https://company-cariere-url.com',
  delay: 1000,              // Delay între cereri (ms)
  timeout: 10000,           // Timeout request (ms)
  userAgent: 'Mozilla/5.0 ...',
  outputDir: './output',
  scraperFile: 'nume-companie.md'  // Nume fișier scraper
}
```

### 2. HTTP Request
- Folosește `axios` cu retry logic
- Implementează rate limiting (delay între request-uri)
- Respectă `robots.txt`
- Setează User-Agent adecvat
- Implementează timeout-uri

### 3. Parsing HTML
- Identifică structura paginii de job-uri (lista job-uri + paginare)
- Extrage date conform Job Model:
  - `url` - URL canonical job
  - `title` - Titlu exact, max 200 chars, diacritice acceptate
  - `company` - Nume legal, UPPERCASE
  - `location` - Array orașe, diacritice acceptate
  - `tags` - Array skills, lowercase, FĂRĂ diacritice, max 20
  - `workmode` - "remote", "on-site", "hybrid"
  - `salary` - String format: "MIN-MAX CURRENCY"
  - `date` - ISO8601 UTC (momentul scrape)
  - `status` - Default "scraped"

### 4. Validare Date
```javascript
// Exemplu cu Zod
const jobSchema = z.object({
  url: z.string().url(),
  title: z.string().max(200),
  company: z.string().optional(),
  cif: z.string().regex(/^\d{8}$/).optional(),
  location: z.array(z.string()).optional(),
  tags: z.array(z.string()).max(20).optional(),
  workmode: z.enum(['remote', 'on-site', 'hybrid']).optional(),
  date: z.string().datetime().optional(),
  status: z.enum(['scraped', 'tested', 'published', 'verified']),
  salary: z.string().optional()
})
```

### 5. Output
- Output JSON conform structurii de mai jos
- Salvare în fișier `output/nume-companie.json`
- Setează `lastScraped` și `scraperFile` în Company model

### 6. Error Handling
- Loghează erorile cu `winston`
- Retry pentru request-uri eșuate (max 3 încercări)
- Continuă la job-ul următor dacă un job nu poate fi extras
- Job-urile cu date incomplete → status `tested`

### 7. Rulare
```bash
npm install
npm start
# sau
node scraper.js
```

## Format Output

### Job Model JSON
```json
{
  "url": "https://company.com/job/123",
  "title": "Developer Java",
  "company": "NUME COMPANIE SRL",
  "cif": "12345678",
  "location": ["București", "Cluj-Napoca"],
  "tags": ["java", "spring", "sql"],
  "workmode": "hybrid",
  "date": "2026-04-29T15:00:00Z",
  "status": "scraped",
  "salary": "5000-8000 RON"
}
```

### Company Model JSON
```json
{
  "id": "12345678",
  "company": "NUME COMPANIE SRL",
  "brand": "BRAND",
  "group": "PARENT GROUP",
  "status": "activ",
  "location": ["București"],
  "website": ["https://company.ro"],
  "career": ["https://company.ro/careers"],
  "lastScraped": "2026-04-29",
  "scraperFile": "nume-companie.md"
}
```

## Reguli și Conventii
- **DIACRITICE:** Acceptate pentru `title`, `location`, `company`. FĂRĂ diacritice pentru `tags`
- **UPPERCASE:** Numele companiilor întotdeauna uppercase
- **CIF/CUI:** Exact 8 cifre, fără prefix "RO"
- **URL:** Valid HTTP/HTTPS, fără trailing slash
- **Status:** Toate job-urile noi încep cu `scraped`
- **Tags:** Lowercase, valori standardizate
- **Workmode:** Doar "remote", "on-site", "hybrid"

## Testare
- Rulează scraper-ul pe câteva pagini înainte de a-l lansa complet
- Verifică output-ul JSON cu un validator JSON
- Verifică că toate câmpurile respectă modelul din `model.md`