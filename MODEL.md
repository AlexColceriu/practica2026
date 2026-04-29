# Modele de Date - Peviitor.ro

## Job Model

| Field          | Type     | Required | Description |
|----------------|----------|----------|-------------|
| url            | string   | Yes      | URL-ul complet către pagina jobului. Unic, valid HTTP/HTTPS |
| title          | string   | Yes      | Titlul exact al poziției. Max 200 caractere, fără HTML, DIACRITICE ACCEPTATE |
| company        | string   | No       | Numele companiei angajatoare. Nume legal complet, uppercase |
| cif            | string   | No       | CIF/CUI (8 cifre, fără prefix RO) |
| location       | string[] | No       | Orașe/adrese din România, DIACRITICE ACCEPTATE |
| tags           | string[] | No       | Skills/educație/experiență, lowercase, max 20 valori, FĂRĂ DIACRITICE |
| workmode       | string   | No       | "remote", "on-site", "hybrid" |
| date           | date     | No       | Data scrape/indexare (ISO8601 UTC) |
| status         | string   | No       | "scraped", "tested", "published", "verified" |
| vdate          | date     | No       | Data verificării (ISO8601) |
| expirationdate | date     | No       | Data expirării estimate (vdate + 30 zile max) |
| salary         | string   | No       | Interval salarial: "MIN-MAX CURRENCY" (ex: "5000-8000 RON") |

### Job Status Flow
```
scraped → (tested OR verified) → published
```

| Status | Descriere |
|--------|-----------|
| scraped | Nou scrape-uit, nevalidat |
| tested | URL funcțional dar date incomplete |
| verified | Complet scrape-uit cu toate detaliile |
| published | Importat în indexul principal |

## Company Model

| Field       | Type     | Required | Description |
|-------------|----------|----------|-------------|
| id          | string   | Yes      | CIF/CUI (exact 8 cifre) |
| company     | string   | Yes      | Nume legal complet, DIACRITICE OBLIGATORII, uppercase |
| brand       | string   | No       | Nume comercial (ex: "ORANGE", "EPAM") |
| group       | string   | No       | Compania părinte (ex: "Orange Group") |
| status      | string   | No       | "activ", "suspendat", "inactiv", "radiat" |
| location    | string[] | No       | Orașe/adrese, DIACRITICE ACCEPTATE |
| website     | string[] | No       | URL site oficial |
| career      | string[] | No       | URL pagină cariere |
| lastScraped | string   | No       | Data ultimei scrape (ISO8601) |
| scraperFile | string   | No       | Nume fișier scraper (ex: "epam.md") |

## Note
- Câmpurile `string[]` sunt array-uri multi-valoare
- SOLR/OpenSearch: analyzer "romanian" păstrează diacriticele ȘȚĂÂÎ
- Search: "Bucuresti" matchează automat "București"
- Companiile inactive/radiate: joburile asociate trebuie șterse