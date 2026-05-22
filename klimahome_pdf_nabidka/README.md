# Klimahome — PDF nabídka po kalkulačce

První master verze PDF dokumentu, který Klimahome posílá zákazníkovi e-mailem po vyplnění webové kalkulačky.

## Soubory

- `Klimahome PDF nabidka.html` — **hlavní design**, 5× A4 stránka. Otevři v prohlížeči.
- Složky `logo_klimahome/`, `montaze/`, `produkty/`, `loga_partneru/`, `qr/`, `reference/` — všechny assety použité v PDF.

## Jak prohlížet

1. Stáhni celou složku (`Code → Download ZIP` na GitHubu nebo `git clone`).
2. Otevři `Klimahome PDF nabidka.html` v prohlížeči (Chrome, Safari, Firefox).
3. Tlačítkem **„Tisk / Uložit jako PDF"** vlevo nahoře vyexportuješ skutečné PDF (A4, bez okrajů).

## Jak komentovat

Otevři PDF nabídku, projdi 5 stránek a v Issues k tomuto repu zakládej komentáře po stránkách (např. „Strana 3, box 4 — text upravit takto…").

## Jak editovat

Soubor `Klimahome PDF nabidka.html` je čistý HTML + CSS, žádný framework. Otevři v jakémkoliv editoru (VS Code, Sublime, Notepad). Texty hledej přes Ctrl+F, barvy jsou centrálně v `:root { --orange: ... }` na začátku `<style>` bloku.

## Brand

- Barvy: oranžová `#ff4227` (akcent), tmavě šedá `#2a2c33` (text), krémová `#faf6f1` (sekundární)
- Font: Poppins (Google Fonts, načítá se přes CDN)
- Logo: oficiální SVG z `logo_klimahome/klimahome_logo.svg`

## Personalizační proměnné

V PDF jsou vyplněny vzorové hodnoty (Pan Vzorový Zákazník · Praha-západ · 2 místnosti · č. nabídky 2026-05-000001 · datum 21. 5. 2026). Při napojení na kalkulačku tyto stringy nahradíš proměnnými serverového template engine.
