# Jak stavíme web firmě

Destilát ze dvou hotových projektů — **Klimahome** (Astro 5, Cloudflare Pages) a **Voltimo**
(Next.js 16, statický export, GitHub Pages). Není to referát o nich. Je to návod pro **třetí web**.

Co je tady, je vyzkoušené. Co je označené **⚠️ chyba, která stála čas**, se v jednom z projektů
skutečně stalo a zaplatilo se za to předěláváním.

---

## 0. Pořadí, které funguje

Pořadí není kosmetika. Oba projekty se zdržely tím, že se některý krok udělal pozdě.

| # | Krok | Proč právě tady |
|---|---|---|
| 1 | **Fakta o firmě** do jednoho souboru | Bez něj si agent vymýšlí a texty se musí přepisovat. |
| 2 | **Pravidla textů + pravidla vzhledu** | Jinak každá stránka vypadá a mluví jinak a sjednocuje se potom. |
| 3 | **Přepínač staging ↔ produkce** a nasazení naprázdno | Nasazení se musí vyzkoušet, dokud je web prázdný. |
| 4 | **Design template** — tokeny a 15 hotových vzorů sekcí | Teprve pak jde stránka postavit za hodinu, ne za den. |
| 5 | **Čísla do dat** (ceník, akce, termíny) | Jakmile je číslo ve dvou stránkách, rozejde se. |
| 6 | Obsah — stránka po stránce | |
| 7 | **Měření, souhlasy, formuláře → CRM** | ⚠️ V obou projektech nejpozdější a nejvíc blokující krok. Patří dřív. |
| 8 | Právní stránky s **reálnými** firemními údaji | Rejstřík se mezitím změní častěji, než si myslíš. |
| 9 | Přesměrování, sitemap, Search Console | |
| 10 | Spuštění | |

**Krok 7 se nedá dohnat.** Člověk, který spravuje kampaně, potřebuje termín přepnutí domény
**minimálně 2 týdny dopředu** — musí si vzít srovnávací data a přepsat cílové adresy ve všech
kampaních. Web může být hotový a spuštění přesto čeká na něj. Ptej se na to v prvním týdnu.

---

## 1. Stack — co se osvědčilo

Oba projekty stojí na jiném frameworku a **oba fungují**. Shodné je to, co rozhoduje:

- **Statický výstup.** Žádný server, žádná databáze pro běžný provoz. Rychlé, levné, nerozbije se.
- **Jeden přepínač prostředí** v konfiguraci: staging / produkce. Nikde jinde se prostředí neřeší.
- **Nasazení z gitu.** Push do větve = nasazení. Žádné ruční kopírování souborů.
- **Design tokeny** (barvy, písmo, mezery, stíny, rádiusy) v jednom CSS souboru. Ve stránkách se
  nikdy nepíše natvrdo barva ani mezera.
- **Minimum závislostí.** Klimahome má v `package.json` dva balíčky a pravidlo „žádné další bez
  domluvy“ — animace a interakce jsou vanilla CSS/JS. Výsledek: build se za rok nerozbije.
- **Česky všechno** — kód, komentáře, commity, dokumentace. Kdokoli to pak přebere.

### Čím se liší a co z toho plyne

| | Klimahome (Astro) | Voltimo (Next.js) |
|---|---|---|
| Interaktivita | vanilla JS na stránce | React komponenty |
| Formulář | serverová funkce v tomtéž projektu | samostatný worker mimo repo |
| Dynamické cesty | ze souborů v `src/pages/` a z dat | generátor cest + MDX |
| Hosting | Cloudflare Pages (umí serverové funkce) | GitHub Pages (jen statika) |

**Poučení:** Astro bylo rychlejší na obsahový web s desítkami stránek. Next.js se vyplatil tam, kde
jsou komponenty s logikou (vícekrokové formuláře, A/B test, kalkulačka). Rozhodující není framework,
ale **jestli hosting umí serverovou funkci**: pokud ne, formulář musí jít přes samostatný worker a to
je o jeden deploy pipeline víc.

### Doporučení pro třetí web

**Varianta A — Astro + Cloudflare Pages.** Pro obsahový web (prezentace, deník, případovky)
nejmenší režie. Formulář běží jako funkce ve stejném projektu, žádný druhý deploy. Build
nejjednodušší, závislostí minimum.

**Varianta B — Next.js + GitHub Pages.** Když se počítá s komponentami s logikou a chci zůstat
na GitHubu (vše v jednom účtu, nasazení přes Actions, zdarma). Cena: formuláře potřebují
externí worker a `basePath` komplikuje odkazy a obrázky.

Pro web studia (vizitka + deník) **doporučuji A**, ale rozhoduješ ty — řekni které.

---

## 2. Hosting a nasazení

### Jeden přepínač, dvě prostředí

Voltimo to má nejčistší a je to vzor k převzetí. V konfiguraci se čte **jediná proměnná prostředí**,
a ta odvodí všechno ostatní:

```
isProd = SITE_ENV === "production"

basePath   = isProd ? ""                      : "/<nazev-repa>"
siteOrigin = isProd ? "https://<domena>"      : "https://<ucet>.github.io"
noindex    = isProd ? ""                      : "true"
analytics  = isProd ? "<id kontejneru>"       : ""   ← staging nikdy neměří do ostrých dat
```

Odtud si to berou: kanonické adresy, Open Graph, sitemap, robots, odkazy, cesty k obrázkům.
**V kódu se nikde jinde neřeší, na jaké doméně web běží.**

Produkční repo a staging repo jsou pak **bitově skoro identické** — liší se v jednom řádku
workflow (`SITE_ENV: production`). To je zdravý stav: co prošlo na stagingu, projde i na produkci.

### Staging se nesmí indexovat — dvakrát

1. `noindex` meta a `robots.txt` s `Disallow: /` (dělá přepínač výš),
2. **hlavička na úrovni hostingu** — `X-Robots-Tag: noindex` pro všechny náhledové adresy
   (`*.pages.dev`, `*.github.io`). Pojistka pro případ, že se někde zapomene meta.

### Vlastní doména

Na GitHub Pages se soubor `CNAME` generuje **jen při produkčním buildu** malým skriptem — jinak
by si staging převzal ostrou doménu. Jeden z těch detailů, které se objeví až v provozu.

### Build-gated: nepushuj, co se nesestaví

Závazně, v tomto pořadí:

1. build musí projít **bez chyb a bez varování**,
2. kontrola v prohlížeči na **1440 px i 390 px** — nic nepřetéká, nic nezajíždí pod lepivou hlavičku,
3. `grep` ve výstupu na zakázané výrazy a na **nedoplněné placeholdery** (`DOPLNIT`),
4. teprve pak commit a push.

⚠️ **Chyba, která stála čas:** náhled se nasazoval nahráním lokální složky `dist/`. Kdo nasadil
staging ze své větve, ve které chyběly commity kolegy, ten jeho práci na náhledu „smazal“ —
v gitu zůstala, ale nikdo ji neviděl a hledalo se, co se stalo. **Řešení: každý má vlastní
náhledovou adresu pojmenovanou po své větvi; adresa `staging` se nasazuje jen z aktuální
větve `staging` po `pull`.** Na GitHub Pages tenhle problém nevzniká, protože nasazuje CI z větve.

---

## 3. Struktura repa a dokumenty

### Co má repo mít od prvního dne

```
CLAUDE.md                 mapa repa + pevná omezení (co se nikdy nedělá)
README.md                 stav projektu — tabulka „oblast × stav“, aktualizuje se
docs/
  FAKTA.md                jediný zdroj faktů o firmě
  OBSAH-PRAVIDLA.md       tón, terminologie, šablony stránek, SEO, zákazy
  DESIGN-PRAVIDLA.md      vzory vzhledu — roste s projektem
  DENIK-PRACE.md          zápis na konci každého pracovního dne, nejnovější nahoře
  ROZPRACOVANO.md         předávka mezi sezeními — kde navázat
  OTAZKY.md               co agent nesmí rozhodnout sám
  NAVOD-NOVA-STRANKA.md   jak postavit stránku ze hotových vzorů
zadani/                   jeden .md = jedna stránka; stav draft | k-realizaci | hotovo
src/data/                 ceny, akce, termíny, recenze — čísla, ne texty
src/styles/tokens/        design tokeny
```

### Které dokumenty se vyplatily od začátku a které vznikly pozdě

**Hned a bez diskuse:** `FAKTA.md`, `OBSAH-PRAVIDLA.md`, `CLAUDE.md`.
Bez nich agent píše texty, které se pak přepisují. To je nejdražší druh práce — vypadá hotově.

**Vznikly pozdě a byla to škoda:**

- **`OTAZKY.md`** (u Klimahome `otazky-pro-jakuba.md`) — **nejcennější dokument z obou projektů.**
  Vznikl až v průběhu. Kdyby byl od začátku, nevznikla by polovina práce, která se předělávala.
  Jak funguje, viz kapitola 5.
- **`DENIK-PRACE.md`** a **`ROZPRACOVANO.md`** — přišly, když už se nevědělo, kde se skončilo.
  Nejsou pro čtení, jsou pro **předávku mezi sezeními**. Agent začíná den jejich přečtením.
- **Jeden soubor s čísly** (`src/data/cenik.ts`) — vznikl, až když se ceny rozešly. Kapitola 4.
- **Onboarding pro člověka, který to přebírá** (Voltimo `JAKUB_ONBOARDING.md`) — nainstaluj,
  naklonuj, nastav identitu v gitu, otestuj build, tady jsou příklady zadání, tohle dělej opatrně,
  tady je kontakt. Krátký, konkrétní, s příkazy k vložení do terminálu. Má vzniknout v okamžiku,
  kdy projekt přebírá druhý člověk — ne později.

### `CLAUDE.md` není dokumentace, je to seznam zákazů

Co v něm u obou projektů reálně fungovalo:

- 3–5 odkazů „tohle si přečti, než cokoli uděláš“,
- **pevná omezení**: na co se nesahá (měření, DNS, běžící starý web), co se nepřidává (nové
  závislosti), co se nikdy nepíše (zakázané zkratky a formulace),
- **pracovní postup** v číslovaných krocích, včetně kontrol před pushem,
- povinnost zapsat zápis do deníku na konci dne.

Čím konkrétnější zákaz, tím lépe. „Piš dobré texty“ nefunguje. „Zkratka X se na webu nepoužívá
nikde, ani v menu, tlačítkách a alt textech“ funguje.

---

## 4. Čísla: jediný zdroj a vlastní datum spotřeby

Tohle je největší technické poučení z obou projektů.

### Pravidlo: číslo žije v datech, ne ve stránce

Ceny, slevy, termíny, kapacity, statistiky → **jeden datový soubor**, ze kterého si to berou
stránky i kalkulačka. Když se číslo mění, mění se na jednom místě.

⚠️ **Co se stalo bez tohoto pravidla:** interní ceník byl **bez DPH**, web u cen z něj tvrdil
„cena je konečná, včetně DPH“. Web tedy **podstřeloval ceny o 12 %** — a nikdo to půl roku
nenašel, protože číslo bylo opsané ručně do devatenácti stránek a do kalkulačky. Opravovalo se
na pěti stránkách značek, v kalkulačce a v cenových pruzích zároveň.

Jak to vypadá správně:

- v datech se drží **ta hodnota, která se nemění** (cena bez daně), a převod dělá funkce,
- u každé položky je **komentář, kdo ji potvrdil a kdy**,
- co není potvrzené, je v komentáři označené jako nepotvrzené — a nepočítá se s tím dál,
- dopočítávané hodnoty se **nikdy nepřepisují ručně**.

### Pravidlo: nabídka s koncem musí zmizet sama

⚠️ **Co se stalo:** letní akce skončila 30. 6. Statický web o tom nevěděl. Stránka ji ukazovala
dál a odpočet tikal na samé nuly. Našlo se to až při migraci, měsíce po konci.

Řešení je deset řádků kódu:

```
AKCE = { …, platiDo: 'RRRR-MM-DD' | null }

akceBezi(dnes) → vrátí akci, jen když platiDo ještě neprošlo
```

Stránky se ptají `akceBezi()`, ne `AKCE`. Když datum projde, akce **sama zmizí**.
Radši žádná akce než prošlá. `null` = běží do odvolání, ale pak to musí být vědomé rozhodnutí.

**Totéž platí pro termíny kurzů, kapacity, „platí do vyprodání zásob“ a ceny s platností od data.**

### Pravidlo: čísla se kontrolují strojem, ne okem

U Klimahome vznikl malý skript, který porovná, co web tvrdí, proti zdroji pravdy (technický
katalog v CSV), a vypíše, co chybí nebo nesedí. Našel věci, které člověk nenašel: dvě řady z osmi
na stránce značky, výrobcem už neprodávané řady nabízené na šesti místech. **Jeden skript nad
tabulkou nahradí procházení katalogů.**

---

## 5. Jak se řeší, že agent nesmí vymýšlet fakta

Nejde to zařídit pokynem „nevymýšlej si“. Musí z toho být mechanismus.

### a) `FAKTA.md` — jediný zdroj pravdy o firmě

Jeden soubor, který **každý model čte jako první**, než napíše řádek textu. Co v něm není, se
do textu nedostane.

Co v něm je: název a právní údaje, adresy a čím která je (fakturační ≠ korespondenční ≠ místo,
kam může přijít zákazník), telefony, působnost, služby v pořadí důležitosti, značky a partnerství,
důvěryhodnostní fakta v **přesném znění** (záruka, hodnocení, délka praxe), co se právě nabízí a
co už ne, a — zásadní sekce — **„čísla, která se nesmí objevit bez zdroje“**: počty realizací,
úspory v procentech, ceny, návratnost, rok založení.

Dvě věci, které dělají rozdíl:

- **u každého faktu je datum a kdo ho potvrdil.** „Více než 12 let praxe (ověřeno 6. 10.)“.
  Za rok je vidět, co je třeba znovu ověřit.
- **i vyvrácená tvrzení zůstávají zapsaná** jako zákaz: „tohle se už nenabízí, do textů nepatří“.
  Jinak se to vrátí při příští úpravě.
- **zakázané formulace** rovnou v seznamu: psát takto, ne takto. Včetně toho, co by bylo
  obchodně nepřesné (uvádět provozovnu jako prodejnu, slibovat působnost, kterou firma nemá).

### b) Chybějící fakt se neodhaduje, ale označí

Není-li zdroj: do textu `[DOPLNIT]` (chybí údaj) nebo `[OVĚŘIT]` (údaj existuje, ale nikdo ho
nepotvrdil), řádek do `OTAZKY.md`, **a pokračuje se jinde.** Build nebo grep pak hlídá, že žádný
`DOPLNIT` neodejde na produkci.

### c) `OTAZKY.md` — soubor, na kterém to celé stojí

Nejcennější dokument z obou projektů. Jak funguje:

- otázky mají **stabilní označení** (`TC-16`, `KL-8`, `AUD-3`) a odkazují se na sebe,
- u otázky je **náhledová adresa stránky**, o které je řeč, a **citace sporného tvrzení**,
- odpovídá se **přímo v souboru** na GitHubu (tužka → napsat → commit). Žádný e-mail, žádný chat.
  Odpověď je tím zapsaná v historii u věci, které se týká.
- stav otázky je vidět značkou: ⏳ čeká · ✅ zapracováno,
- pod odpovědí je **řádek „zapracováno kdy a co se změnilo“** — audit trail,
- fakt z odpovědi se **propíše do `FAKTA.md`**. Odpověď v otázkách je doklad, `FAKTA.md` je zdroj.

**Pravidlo, které z toho vzešlo a je to to nejdůležitější:** hraniční tvrzení se **neopravují,
jen evidují**. Agent, který „vylepší“ marketingové tvrzení, ho ve skutečnosti změnil bez
oprávnění. Tvrzení zůstane, jak je, a jde do otázek. Majitel pak řekne „nechat“ nebo „pryč“ —
a v obou projektech řekl „nechat“ častěji, než by agent čekal.

### d) Kontrolu dělá druhý, úzce vymezený agent

Klimahome má na to malého levného agenta (`auditor`), který **nic neopravuje**. Přečte `FAKTA.md`
a `OBSAH-PRAVIDLA.md` a vrátí jen seznam odchylek v pevném formátu
`soubor:řádek — kategorie — co je špatně`. Kategorie: fakta, terminologie, formulace, SEO, odkaz.
Nic nenašel → vrátí přesně „Bez odchylek.“

Oddělení rolí je záměrné: **kdo píše, nekontroluje; kdo kontroluje, nepřepisuje.** Druhý agent
(`extraktor`) jen stahuje a kopíruje, taky nic nepřepisuje. Agent, který smí všechno, zamete
problém pod koberec.

### e) Nezávislý audit před spuštěním

Klimahome si nechalo udělat externí audit 61 stránek: 16 chyb, 11 doporučení, 5 bodů k obchodnímu
potvrzení. Nálezy šly do `AUDIT-UKOLY.md` **seřazené podle toho, co blokuje spuštění**, s odškrtáváním
a datem. Audit našel věci, které vlastní kontrola najít nemohla (odkazy vedoucí jinam, redakční
pokyn `[DOPLNIT]` viditelný na živé stránce, dvě stránky tvrdící o téže věci opak).

Ale pozor na jeho hranici: **audit porovnával nový web proti starému.** Staré firemní údaje
odsouhlasil jako správné, protože starý web je měl stejně staré. Rejstřík nekontroloval nikdo.
**Firemní údaje si ověř ve veřejném rejstříku, ne proti starému webu.**

---

## 6. Obsahový model

### Co funguje

- **Jeden soubor = jedna stránka.** Adresa stránky = název souboru. Žádná mapa cest.
- **Zadání stránky je taky soubor** v repu, se stavem `draft` / `k-realizaci` / `hotovo`.
  Realizuje se **jen** `k-realizaci`. Hotová zadání se nemažou — jsou historie „proč to tak vypadá“.
- **Články v Markdownu s bohatým frontmatterem.** Voltimo to má nejlépe: kromě titulku a popisu
  i `seoTitle` zvlášť, `kicker`, kategorie, doba čtení, datum vydání a aktualizace, **krátká
  odpověď na otázku z titulku** (pro AI a pro výpis), hero s povinným alt textem, **vlastní CTA
  pro ten článek**, blok FAQ a odkazy na související. Obsah článku je pak jen text pod tím.
- **Obsah stránek jako data.** Klimahome drží převedené stránky jako JSON (sekce, odstavce, fotky,
  tlačítka) a šablona je vykresluje. Obsah a vzhled jde měnit nezávisle.

### ⚠️ Chyba, která stála čas: velké přegenerování

Když obsah vyrábí skript, **další spuštění přepíše ruční opravy**, které mezitím někdo udělal.
Řešení, které v Klimahome vzniklo: skript píše do jedné složky („co bylo stahnuto“), ruční opravy
žijí ve **druhé složce** (`opravy/<slug>.json`) a sestavení je spojí. Přegenerování pak ruční
práci nepřepíše.

Pravidlo: **generovaný soubor a editovaný soubor nejsou nikdy tentýž soubor.** Platí i pro export
design tokenů: zdroj je v `src/styles/tokens/`, export je generovaný a v hlavičce má „needituj“.

### Kontrola úplnosti strojem

U převodu starého webu porovnával skript starou a novou stránku odstavec po odstavci a fotku po
fotce, a musel hlásit 0 chybějících. **Falešné nálezy se zapisovaly jako známé** („dekorativní
ikony“, „odstavec rozložený do tabulky“), aby se znovu nekontrolovaly.

---

## 7. SEO a AEO

Co měly oba projekty a co stojí převzít:

- **Titulek a popis zvlášť pro každou stránku**, s pevnými délkami (titulek 50–60 znaků, popis
  120–155) a povinnou unikátností. Kontroluje to auditor.
- **Šablona titulku** v layoutu (`%s | Firma`), aby se neopisovala.
- **Open Graph a Twitter card** z jednoho místa v layoutu, obrázek 1200 × 630.
- **Kanonická adresa** odvozená z přepínače prostředí — nikdy natvrdo.
- **JSON-LD organizace** v layoutu (jméno, logo, adresa, telefon, e-mail).
  ⚠️ Pozor: **Google čte adresu právě odsud** a ukazuje ji ve vyhledávání. Když se firmě změní
  sídlo, je to jedno z míst, na které se zapomene. V Klimahome bylo staré sídlo na 61 stránkách
  v patičce **a navíc** ve strukturovaných datech.
- **Sitemap generovaná z kódu** — statické cesty se prioritami + cesty článků z jejich souborů.
  Nikdy ručně udržovaný seznam.
- **`robots.txt` z přepínače**: staging zakázat celý, produkce povolit s výjimkou interních stránek.
- **Verifikace Search Console** přes meta v layoutu (přežije změnu hostingu, na rozdíl od souboru).
- **Jeden H1 na stránku**, H2 sekce, H3 uvnitř. Alt texty česky, popisné, bez slova „obrázek“.
- **Prolinkování jako pravidlo, ne jako nápad**: každá stránka služby odkazuje na kalkulačku
  a na poptávku, podstránky na nadřazenou. Auditor kontroluje, že odkaz vede na existující cíl.
- **AEO**: `answer` ve frontmatteru článku (dvě až tři věty, které odpovídají na otázku z titulku)
  a blok FAQ přímo u obsahu. Píše se pro to, aby to šlo citovat.
- **Vyhledávání na webu** nad indexem vyrobeným při sestavení — žádná služba, žádný server.

### Přesměrování starých adres

Když web nahrazuje starý, je to nejvíc podceněná položka. Konkrétně:

- mapu starých adres **vytáhni crawlem živého webu**, ne z hlavy,
- rozsah: jen **aktivní indexované** stránky (starý web polovinu adres sám přesměrovával),
- **ověř každou adresu curlem**, že opravdu přesměrovává. Audit našel 26 přesměrování, která
  končila 404,
- pozor na **adresu se lomítkem i bez** — hosting je sám neztotožňuje,
- pozor na **limit počtu pravidel** hostingu: co je nad limit, se tiše ignoruje a vrací 404.
  Po přidání pravidel vždy ověřit **první i poslední** adresu v souboru.
- stažená služba **nesmí skončit 404**, když na ni vedou odkazy a kampaně — zůstane krátká
  stránka s jednou větou a rozcestníkem.
- PDF a soubory ke stažení **přenes na vlastní doménu** před vypnutím starého webu.

---

## 8. Právní a konverzní vrstva

### Souhlasy a měření

Oba projekty mají stejné řešení a stojí ho převzít celé:

1. **Všechno měření jde přes jeden kontejner** (GTM). V kódu nejsou žádné měřicí skripty —
   žádný gtag, pixel, retargeting. Tagy se konfigurují v kontejneru a tam má každý podmínku na souhlas.
   **Pravidlo: inline měřicí skript ve stránce = chyba.**
2. **Consent Mode v2 se nastaví dřív než kontejner.** Výchozí stav „zamítnuto“ pro EU, uloženou
   volbu uživatele se aplikuje ještě před načtením kontejneru. **Pořadí v layoutu se nemění** —
   jinak tagy startují bez správného stavu souhlasu.
3. Volba se ukládá do **first-party cookie** (ne localStorage — vidí ji i subdomény a server)
   a nese **verzi politiky**; zvýšení verze zobrazí lištu všem znovu. Platnost 180 dní.
4. **Kontrakt na události v dataLayer** je dokument, ne dohoda v hlavě: tabulka
   událost → kdy se pošle → jaké údaje. Konvence pro nové stránky jsou pak triviální
   (`data-cta="…"` na tlačítku, `data-form="…"` na formuláři).
5. **Volba „nepřeji si být kontaktován“** posílá vlastní událost **místo** konverzní, nikdy vedle ní.
6. **Audit log souhlasů** (kdo, kdy, jakou verzi politiky) jako serverová funkce + databáze.
   Do logů nejde nic osobního.
7. **Staging nikdy neměří do ostrých dat** — na stagingu je ID kontejneru prázdné.

⚠️ **Co se našlo v existujícím kontejneru:** tagy, které i při „zamítnuto“ střílely dál —
retargeting s natvrdo zapsaným souhlasem, dvě nahrávací služby bez podmínky, pixel s natvrdo
uděleným souhlasem. Web byl v pořádku, kontejner ne. **Když přebíráš měření, kontejner se musí
projít, ne jen kód.**

### Formuláře a poptávky

- **Formulář nikdy nepíše do CRM z prohlížeče.** Jde na serverovou funkci (nebo worker),
  ta sestaví lead a pošle ho do CRM. Klíč zůstává na serveru.
- **Formát poznámky v leadu je kontrakt.** Prefixy (`Form ID:`, `Url:`) parsuje analytika —
  mění se jen po domluvě.
- **Jedno ID pro CRM i pro měření.** Formulář vyrobí UUID, pošle ho do CRM i do dataLayer.
  Bez toho se lead v CRM nespáruje s konverzí v reklamě a nikdo neví, co kampaň vydělala.
- **Zdroj poptávky patří do leadu.** Předmět podle formuláře nebo kalkulačky, adresa stránky
  v poznámce. Jediné, co obchod chtěl, a kdyby to chybělo, dodělávalo by se potom.
- **Suchý běh.** Bez přístupových údajů (nebo s přepínačem) se lead sestaví a nepošle.
  Staging tak nezakládá falešné poptávky obchodníkům. **Nutnost, ne luxus.**
- **Ochrana proti spamu** bez externí služby: skryté pole pro roboty, kontrola `Origin`,
  limit velikosti, validace polí.
- **Záloha při selhání CRM.** Když CRM lead nepřijme, poptávka je pryč a návštěvník vidí chybu.
  Při selhání (a jen při selhání) uložit lead na stranu a poslat notifikaci. Za normálního běhu
  se neukládá nic — žádná druhá kopie osobních údajů.
- **Do měření nikdy nejde volný text od uživatele.** Jméno, telefon, poznámka = osobní údaje,
  patří do CRM, ne do analytiky.

### Právní stránky

- Údaje správce osobních údajů (název, IČO, sídlo, kontakt) **na jednom místě** a odtud do VOP,
  zásad cookies a zpracování osobních údajů.
- **Fakturační, korespondenční a návštěvní adresa jsou tři různé věci** a web si je plete.
  Do VOP patří adresa, kam reálně chodí reklamace.
- ⚠️ **Co se stalo:** v dokumentu o ochraně osobních údajů byl roky uvedený jako zastupující
  jednatel člověk, který už ve firmě nebyl. Nepravdivý údaj přesně v tom dokumentu, kde to vadí
  nejvíc. Web to opsal 1:1 ze starého a nikdo to nekontroloval.
- **Rozpor mezi marketingem a VOP se musí vyřešit, ne obejít.** Když stránky slibují pětiletou
  záruku a VOP uvádějí dva roky, je to problém, i když to tak bylo vždycky. Zapiš to jako otázku,
  nech rozhodnout majitele a **rozhodnutí si nech napsat.**

---

## 9. Konvence a spolupráce

### Větve

- Společný základ je **jedna integrační větev** (`staging`), ne `main`.
  `main` je ostrý web a sahá se na něj jen při spuštění.
- Každý úkol **krátká větev s předponou podle typu** (`lp/…`, `migrace/…`, `fix/…`).
- Vlastní **náhledová adresa pojmenovaná po větvi**, ta se posílá ke schválení.
- Po schválení slučení do integrační větve, build, až pak nasazení.
- Před pushem vždy `pull` a **znovu build**.

### Commity

Česky, krátce, **co a proč**. Ne „oprava“ — „ceny klimatizací přepočítané na ceny s DPH 12 %“.

### Sdílené soubory

⚠️ **Chyba, která stála čas:** na webu pracovali tři lidé, každý se svým agentem. Nejčastější
střet bylo **menu a patička** v layoutu — oba se z principu mění při každé nové stránce.

Pravidla, která z toho vzešla:

- **Nová stránka = nové soubory. To je bezpečná zóna.** Nikomu nic nepřepíše.
- Sdílené soubory (layout, komponenty, tokeny, přesměrování, pravidla vzhledu) se mění
  **malým samostatným commitem** a dá se o tom vědět. Do layoutu přidávej **jen svůj řádek**.
- Potřebuješ v komponentě něco navíc? **Volitelný parametr s výchozí hodnotou**, ne přepsání.
  Stávající stránky se tím nezmění.
- **Hotové stránky kolegy se nepředělávají bez domluvy.** Seznam „hotovo, nesahat“ je v dokumentaci.
- Komu patří která část repa, je **v tabulce**. Měření měl na starost jeden člověk a platil zákaz
  sahat na jeho soubory i na jejich pořadí v layoutu.

### Design template roste

Nový opakovatelný vzor (komponenta, animace, typ sekce) se **zapíše do pravidel vzhledu** —
jednou větou a jedním odstavcem: co to je, kde je kód, kdy se to používá.
**Cíl: další stránka vzniká rychleji než předchozí.** Po pár týdnech se stránka skládá
z patnácti hotových vzorů a nevymýšlí se od nuly.

Vedle toho se vyplatilo mít **„výdejní okénko“ design systému**: jedna složka s vyexportovanými
tokeny (CSS i strojově čitelný JSON) a jednou samostatnou HTML stránkou s živými ukázkami prvků.
Pro práci mimo repo (návrhy, landing pages, jiný nástroj) stačí ta složka. Generuje se skriptem.

### Tajné údaje

Nikdy do repa. Ani v komentáři, ani „dočasně“. Patří do proměnných prostředí hostingu nebo do
secrets v CI. Předávají se **správcem hesel nebo osobně**, ne mailem ani chatem.
ID měřicího kontejneru tajné není (je vidět ve zdrojáku) — to v repu být může.

⚠️ **Co se našlo:** starý WordPress plugin měl přístupové údaje k CRM **natvrdo v kódu**.
Správný postup, který se použil: vygenerovat **nový** klíč, nasadit ho do proměnných hostingu
a **starý po přepnutí zrušit**.

---

## 10. Předání třetí straně — co musí být dopředu

Spuštění nedrží obsah webu. Drží ho tohle:

| Co | Kdo to potřebuje | Jak dlouho dopředu |
|---|---|---|
| **Termín přepnutí domény** | správce kampaní | **min. 2 týdny** |
| Srovnávací data (14 dní před přepnutím) | správce kampaní | před termínem |
| Přepsání cílových adres ve všech kampaních | správce kampaní | v den přepnutí |
| ID měřicího kontejneru + úklid tagů podle souhlasu | správce měření | před testy |
| Nový klíč do CRM předaný správcem hesel | kdo spravuje hosting | před testy |
| Kdo má přístup ke správě domény (DNS) | ten, kdo přepíná | **zjisti to v prvním týdnu** |

### Dokument k předání

Vyplatilo se napsat **jeden dokument pro toho člověka**, který obsahuje:
co je hotové (tabulkou), co od něj potřebujeme (odškrtávací seznam), proč mu nedáváme přístup tam,
kam ho nepotřebuje, a **plán testů**. Odpovědi si pak dopisuje přímo do toho dokumentu —
stejný princip jako `OTAZKY.md`.

### Plán testů před přepnutím

1. **Staging s ID kontejneru**, v náhledovém režimu projít souhlas (přijmout i odmítnout)
   a všechny formuláře. Ověřit, že při „nepřeji si být kontaktován“ nestřílí konverze.
   Data ze stagingu se v analytice odfiltrují podle hostname.
2. **CRM:** z každého formuláře jeden lead se jménem „TEST — nevolat“, zařazený do kategorie,
   kterou report vylučuje. Obchod zkontroluje předmět, poznámku a zdroj a smaže je.
   Pak se staging vrátí do suchého běhu.
3. **Po přepnutí:** jeden ostrý test z každého formuláře, kontrola konverzí do 24 h
   a pak znovu po 3 a 7 dnech proti srovnávacím datům.

### Pozor na starý web

DNS se nemění, dokud není rozhodnuto. Starý web **běží dál a neupravuje se ani nemaže**,
dokud nový web neběží na ostré doméně a neověří se. To je pevné omezení, které patří do `CLAUDE.md`.

---

## 11. Kde se utopil čas — souhrn

| Co se stalo | Co tomu šlo předejít |
|---|---|
| Cena opsaná do 19 stránek a kalkulačky, ceník bez DPH, web „včetně DPH“ → **ceny o 12 % nižší** | jeden datový soubor s čísly a jeden vzorec na převod, od začátku |
| Akce skončila 30. 6., web ji ukazoval dál s odpočtem na nulách | pole `platiDo` a funkce, která prošlou akci sama skryje |
| Dvě různé verze téhož obchodního slibu na dvou stránkách | fakta v `FAKTA.md`, auditor kontroluje odchylky |
| Staré sídlo na 61 stránkách a navíc ve strukturovaných datech | firemní údaje na jednom místě; **ověřit v rejstříku, ne proti starému webu** |
| Nepravdivý jednatel v dokumentu o osobních údajích | právní stránky se nepřepisují 1:1 bez kontroly údajů |
| Nasazení náhledu „smazalo“ práci kolegy | vlastní náhledová adresa na větev; nasazovat z CI, ne z lokálního stroje |
| Konflikty v menu a patičce | nová stránka = nové soubory; sdílené soubory malým commitem |
| Přegenerování přepsalo ruční opravy | generovaný a editovaný soubor nejsou tentýž soubor |
| 26 přesměrování končilo 404, PDF viselo na staré doméně | crawl starého webu + ověření každé adresy curlem |
| Redakční pokyn `[DOPLNIT]` viditelný na živé stránce | grep na placeholdery před nasazením, jako součást build-gate |
| Měření a napojení CRM se řešilo až na konci | patří do prvního týdne, včetně termínu přepnutí |
| Stránka značky nabízela řady, které výrobce už neprodává | skript, který porovná web proti tabulce parametrů |
| Agent „vylepšil“ marketingové tvrzení | hraniční tvrzení se **neopravují, jen evidují** |

---

## 12. Checklist od nuly k nasazení

### A. Než se napíše řádek kódu

- [ ] **Potvrdit účet a jméno repa**, viditelnost (začít privátně).
- [ ] **Potvrdit stack a hosting** — nevolit sám, nabídnout dvě varianty s důvody.
- [ ] **Potvrdit doménu**: existuje? kdo spravuje DNS? kdy se smí přepnout?
- [ ] Založit repo, `CLAUDE.md`, `README.md` se stavovou tabulkou.
- [ ] `docs/FAKTA.md` — i kdyby byla z poloviny `[DOPLNIT]`.
- [ ] `docs/OBSAH-PRAVIDLA.md` — tón, terminologie, šablony stránek, SEO délky, zákazy.
- [ ] `docs/OTAZKY.md` — prázdný, s návodem jak odpovídat.
- [ ] `.gitignore` (včetně `.env*` a `.DS_Store`).

### B. Skelet a nasazení naprázdno

- [ ] Projekt se sestaví a spustí lokálně.
- [ ] **Jeden přepínač prostředí** → basePath, origin, noindex, ID měření.
- [ ] CI: push do větve = build + nasazení. **Build musí být podmínkou nasazení.**
- [ ] Staging nasazený, prázdný, **noindex ověřený dvakrát** (meta + hlavička hostingu).
- [ ] Design tokeny v jednom souboru. Layout: hlavička, patička, skip-link, SEO meta, JSON-LD.
- [ ] `sitemap` a `robots` generované z kódu.
- [ ] `docs/DENIK-PRACE.md` — první zápis.

### C. Design template

- [ ] 10–15 vzorů sekcí jako komponenty s volitelnými parametry.
- [ ] `docs/DESIGN-PRAVIDLA.md` — co je závazné, co je nejčastější chyba.
- [ ] `docs/NAVOD-NOVA-STRANKA.md` + šablona stránky s místy `DOPLNIT`.
- [ ] Pohyb umlká při `prefers-reduced-motion`.
- [ ] Zkontrolováno na 1440 px a 390 px.

### D. Data a obsah

- [ ] `src/data/` — čísla, akce s `platiDo`, termíny. Komentář kdo a kdy potvrdil.
- [ ] Obsahový model: frontmatter článku včetně `answer`, FAQ, vlastního CTA.
- [ ] Zadání stránek v `zadani/` se stavy. Realizuje se jen `k-realizaci`.
- [ ] Stránka po stránce. Po každém bloku: build, kontrola, náhled, zápis do deníku.
- [ ] Nedoložené tvrzení → `[OVĚŘIT]` + řádek do `OTAZKY.md`, **pokračuj jinde**.

### E. Konverze a právo

- [ ] Souhlasy: Consent Mode před kontejnerem, first-party cookie s verzí politiky.
- [ ] Kontrakt na události v dataLayer jako dokument.
- [ ] Formulář → serverová funkce → CRM. UUID do CRM i do dataLayer. Zdroj poptávky v leadu.
- [ ] Suchý běh na stagingu. Ochrana proti spamu. Záloha při selhání CRM.
- [ ] Právní stránky s **ověřenými** údaji z rejstříku. Rozpory marketing × VOP vyřešené.
- [ ] Zásady cookies, zpracování osobních údajů, VOP — správce na jednom místě.

### F. Před spuštěním

- [ ] Přesměrování: crawl starého webu, **každá adresa ověřená curlem**, pozor na limit pravidel.
- [ ] Soubory ke stažení přenesené na vlastní doménu.
- [ ] `grep` ve výstupu: žádné `DOPLNIT`, žádné zakázané výrazy.
- [ ] Titulky a popisy unikátní, délky v rozsahu.
- [ ] Každý odkaz vede na existující cíl.
- [ ] Nezávislý audit, nálezy seřazené podle toho, co blokuje spuštění.
- [ ] **Termín přepnutí domluvený ≥ 2 týdny dopředu.** Srovnávací data odebraná.
- [ ] Testovací leady odeslané, zkontrolované obchodem, smazané.
- [ ] Vím, kdo spravuje DNS.

### G. Spuštění a po něm

- [ ] Produkční build s produkčním přepínačem. `CNAME` / doména. HTTPS.
- [ ] Cílové adresy kampaní přepsané. Uklizený kontejner publikovaný.
- [ ] Ostrý test z každého formuláře. Konverze do 24 h.
- [ ] Search Console: ověření, sitemap odeslaná.
- [ ] Kontrola leadů a konverzí D+1, D+3, D+7 proti srovnávacím datům.
- [ ] Starý web vypnut **až** po ověření.
- [ ] `README.md` a deník aktualizované.

---

## 13. Co bych u třetího webu udělal jinak

1. **`OTAZKY.md` od prvního dne**, i prázdný. Je to nejcennější dokument obou projektů a v obou
   vznikl pozdě.
2. **Čísla do dat hned**, i když je zatím jedno. Dvě ceny na dvou místech se rozejdou vždy.
3. **Měření a napojení CRM do prvního týdne**, aspoň rozhovorem. Terminuje spuštění.
4. **Firemní údaje z rejstříku, ne ze starého webu.** Stálo to opravu na 61 stránkách
   plus strukturovaná data plus právní texty.
5. **Datum spotřeby u všeho, co ho má.** Akce, termín, kapacita, cena platná od. Statický web
   nepozná, že je po termínu — musí to být v datech.
6. **Fakt bez data potvrzení je polovina faktu.** Za rok není vidět, co ověřit znovu.
7. **Agent, který kontroluje, nesmí opravovat.** A kdo opravuje, nekontroluje.
8. **Hraniční tvrzení neopravovat, jen evidovat.** Rozhodnutí je obchodní, ne technické.
9. **Rozsah určit na začátku a napsat ho do souboru.** U Klimahome se zjistilo až v průběhu,
   že z 122 adres je aktivních 71 — zbytek starý web sám přesměrovával.
10. **Z jednoho dokumentu udělat výdejní okénko design systému.** Vyplatilo se víc, než to vypadalo:
    práce mimo repo (návrhy, landing pages) pak nepotřebuje celý projekt.
11. **Nasazovat z CI, ne z lokálního stroje.** Ruční nahrávání složky je nejrychlejší cesta
    k „zmizelé“ práci kolegy.
12. **Deník psát každý den.** Ne pro čtení — pro předávku mezi sezeními. Bez něj se další den
    začíná zjišťováním, kde se skončilo.

---

_Sepsáno 9. 10. 2026 z repozitářů `klimahome-web` (větev `staging`) a `voltimo-staging` / `voltimo-web`.
Zdrojová repa byla jen čtena — nic se v nich nezměnilo. Převzaty jsou postupy a architektura,
žádné texty, brand, fotky, domény ani údaje klientů._
