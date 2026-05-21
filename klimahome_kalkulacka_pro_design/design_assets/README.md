# Podklady pro PDF nabídku — inventura

Stažené ze stránek `klimahome.cz` (vlastní obsah Klimahome, licence OK). Strukturováno ve složce `design_assets/` v projektu Klimahome.

## ✅ Co je k dispozici

### 1. Hero foto titulky (1× hotové, alternativy)

| Soubor | Použití | Rozměr |
|---|---|---|
| `montaze/hero_realizace_main.jpg` | **Doporučeno na titulku** — celkový hero shot Klimahome | 1024×617 |
| `montaze/montaz_1.jpg` až `montaz_4.jpg` | Alternativy pro hero nebo na další strany | rozlišení web |

### 2. Vnitřní jednotky (3 značky — splněno)

| Soubor | Značka | Kvalita |
|---|---|---|
| `produkty/beko_vnitrni_jednotka.png` | Beko Evolutio Pro | dobrá, transparentní pozadí |
| `produkty/samsung_windfree.png` | Samsung WindFree | dobrá, větší rozlišení |
| `produkty/toshiba_haori.webp` | Toshiba Haori | dobrá, webp formát — designer převede |

⚠️ **Tip:** všechno jsou produktové fotky z webu Klimahome (`/wp-content/uploads/`). Pro tiskovou kvalitu (300 DPI) ideálně dotáhnout vyšší rozlišení od výrobců — Samsung, Toshiba i Beko mají pressroom s vysokorozlišovými fotkami. Pro digitální PDF jdoucí mailem stačí.

### 3. Loga partnerů (3/5 — chybí 2)

| Logo | Status | Soubor |
|---|---|---|
| Samsung | ✅ máme (jako header banner) | `loga_partneru/loga_header_samsung.png` |
| Toshiba | ✅ máme | `loga_partneru/loga_header_toshiba.png` |
| Beko | ✅ máme | `loga_partneru/loga_header_beko.png` |
| Acond | ❌ chybí — Klimahome web má vlastní stránku, ale logo není samostatně dostupné | — |
| Daikin | ❌ chybí — Klimahome web ho zmiňuje, ale samostatné logo nemá | — |

**Řešení:** Acond a Daikin si stáhne designer z oficiálních partner pressroom (acond.cz, daikin.cz) nebo brandfetch.com. Pokud nechceš čekat, lze pro v1 vyhodit a nechat jen "Partner: Samsung, Toshiba, Beko" + sekci "a další".

### 4. Logo Klimahome ✅

`logo_klimahome/klimahome_logo.svg` — **vektorové oficiální logo** přímo z webu (328×50). Plně škálovatelné, designer s ním pracuje bez omezení. Plus `favicon.jpg` jako bonus.

### 5. QR kód na telefon ✅

`qr/qr_telefon_606872526.png` — QR kód s `tel:+420606872526`. Po naskenování smartphone otevře dialer s předvyplněným číslem. Vygenerováno v barvě Klimahome modré (#0F4C75).

### 6. Reference fotky ✅ (bonus, na stranu „proč my")

`reference/Dolni_Lukavice_1-3.jpg` — fotky z hotové reference rodinného domu (Dolní Lukavice, paní Iva, Acond multisplit). Použitelné pro stránku „naše realizace".

### 7. Logo Google / Firmy.cz pro social proof ✅

V `loga_partneru/google_review_logo.png` a `firmy_cz_logo.png` — k použití v patičce („4,8/5 na Google · 4,9/5 na Firmy.cz").

---

## 📋 Personalizační proměnné (z formuláře kalkulačky → PDF)

Vstupují do PDF jako placeholdery vyplněné serverem. Designer připraví zástupné textové vrstvy s default hodnotami:

| Proměnná | Příklad | Zdroj | Default v PDF mockupu |
|---|---|---|---|
| `{jmeno}` | "Pan Novák" | formulář, krok 3 | "Pan Vzorový Zákazník" |
| `{okres}` | "Praha-západ" | formulář, krok 1 | "Praha-západ" |
| `{pocet_mistnosti}` | "2" | formulář, krok 2 | "2" |
| `{vybrana_varianta}` | "Samsung WindFree" | klik na „Chci tuto" v kroku 4 | "Samsung WindFree" (default jako prostřední) |
| `{cislo_nabidky}` | "2026-05-002847" | server-side auto-increment (rok-měsíc-pořadí) | "2026-05-000001" |
| `{datum_vygenerovani}` | "21. 5. 2026" | server-side, datum odeslání formuláře | dnešní datum |

**Doporučení formátu čísla nabídky:** `RRRR-MM-NNNNNN` (rok-měsíc-6cifr). Důvod: telefonistka i obchodník při hovoru hned vidí, ze kterého období lead je, a unikátní ID pro Raynet.

Pokud nemáš auto-increment v současném systému, alternativa: UUID prvních 8 znaků (např. `KH-A3F7E2B1`) — také unikátní, ale méně čitelné pro člověka po telefonu.

---

## 📦 Struktura, jak to předáš designerovi

```
design_assets/
├── logo_klimahome/
│   ├── klimahome_logo.svg     ← oficiální vektor, použij vždy
│   └── favicon.jpg
├── produkty/
│   ├── beko_vnitrni_jednotka.png
│   ├── samsung_windfree.png
│   └── toshiba_haori.webp
├── montaze/
│   ├── hero_realizace_main.jpg     ← na titulku
│   └── montaz_1-4.jpg              ← do těla
├── loga_partneru/
│   ├── loga_header_samsung.png
│   ├── loga_header_toshiba.png
│   ├── loga_header_beko.png
│   ├── google_review_logo.png
│   └── firmy_cz_logo.png
├── reference/
│   └── Dolni_Lukavice_1-3.jpg
└── qr/
    └── qr_telefon_606872526.png
```

---

## ❗ Co je potřeba ještě sehnat (zhruba 30 min práce)

1. **Logo Acond** — z `acond.cz/o-nas` nebo brandfetch.com
2. **Logo Daikin** — z `daikin.cz/cs_cz/about-us.html` nebo brandfetch.com
3. **(Volitelné) Vyšší rozlišení produktových fotek** — Samsung pressroom, Toshiba EU media, Beko press portal. Pokud PDF jde jen mailem, není nutné.

---

## Souhrn

| Požadavek z briefu | Status |
|---|---|
| 4× foto (1× hero, 3× vnitřní jednotky) | ✅ |
| Logo Klimahome (vektor) | ✅ |
| 5× loga partnerů | ⚠️ 3/5 (chybí Acond, Daikin) |
| QR kód na telefon | ✅ |
| Personalizační proměnné — definice | ✅ |
| Bonus: hero foto reference | ✅ |
| Bonus: review badges (Google, Firmy.cz) | ✅ |
