# TPA Viewer

Przeglądarka i edytor plików `.TCN` generowanych przez oprogramowanie TPA dla maszyn CNC (centra obróbcze do drewna i mebli). Aplikacja działa w całości w przeglądarce – jest pojedynczym plikiem HTML, nie wymaga instalacji ani serwera.

## Możliwości

### Podgląd operacji
- **Widok 3D** – przestrzenny model elementu z naniesionymi operacjami (Three.js, pełna orbita kamery)
- **Widoki 2D** – rzuty: TOP, FRONT, BACK, RIGHT, LEFT z etykietami wymiarów
- **Widok CODE** – surowy tekst pliku `.TCN`

### Obsługiwane operacje
| Typ               | Opis                               | Kolor     |
|-------------------|------------------------------------|-----------|
| Nawiert (W#81)    | Otwór przelotowy lub nieprzelotowy | zielony   |
| Frezowanie (W#89) | Ścieżka wielopunktowa freza        | fioletowy |
| Piłka (W#1050)    | Cięcie piłą tarczową               | turkusowy |

### Katalog plików
Lewy panel pozwala wskazać folder na dysku i przeglądać pliki `.TCN` jako miniatury (widok TOP). Wspierane jest sortowanie po nazwie lub dacie, skalowanie miniatur oraz wyszukiwanie – ręczne i przez skaner kodów kreskowych.

### Edycja i zapis
- Zmiana wymiarów elementu (długość × wysokość × grubość)
- Obrót elementu o 90° lub 180° (z przeliczeniem współrzędnych operacji)
- Dodawanie nowych operacji: nawiert, frezowanie, piłka
- Edycja parametrów istniejących operacji (średnica, głębokość, współrzędne)
- Zapis do oryginalnego pliku (File System Access API) lub „Zapisz jako…"
- Tworzenie nowego pliku od zera

### Pliki parametryczne (szablony)
Aplikacja otwiera pliki parametryczne zapisane w TpaCAD (`'code=unicode`, UTF-16):
- **Wzory we współrzędnych** – `x`, `y`, `z` (wymiary płaszczyzny wg konwencji TPA: TOP x=DL/y=DH, FRONT/BACK x=DL/y=DS, LEFT/RIGHT x=DH/y=DS), `l`, `h`, `s` (wymiary elementu) oraz arytmetyka, np. `x/2-9`, `y-37`, `x-(r0/2)-0.5`
- **Zmienne `r`** – sekcja `VAR{}` (`r0`, `r1`, …) z nazwami; edytowalne w panelu „Zmienne (VAR)" – po zmianie wszystkie wzory i warunki przeliczają się na żywo
- **Instrukcje logiczne IF/ELSE/ENDIF** (`W#2001`/`W#2005`/`W#2002`) – operacje w nieaktywnych gałęziach są ukrywane, np. rzędy otworów pod półki pojawiają się w zależności od wartości `r1` (ilość półek); warunek: do 3 termów `(e1) ? (e2)` z operatorami `< <= > >= = <>` łączonymi And/Or
- **Tryb szablonu** – plik z instrukcjami logicznymi jest przy zapisie zachowywany w oryginalnej strukturze (wzory i ukryte operacje nie giną); z poziomu TPA-Viewera koryguje się tylko wymiary elementu i wartości zmiennych `r`. Obrót i dodawanie operacji są w tym trybie zablokowane, a przy nazwie pliku pojawia się znacznik „⚙wzory"

### Generator szafek
Zakładka „Generator szafek" składa listę formatek z definicji szafki (wzory wymiarów z parametrami `H/W/D/T` + własnymi) i zapisuje pliki TCN o kolejnych numerach (hex/dec) do wskazanego katalogu, z etykietami i kodami kreskowymi:
- **Szablony TCN per element** – przy generowaniu w szablonie podmieniane są wymiary (`::UNm`) oraz **zmienne `r`** z sekcji `VAR{}`: przycisk **r** przy szablonie pokazuje jego zmienne (r0, r1, …) i pozwala przypisać im wzory szafki, np. `r0=T`, `r1=N` – wyliczona wartość trafia do pliku wynikowego
- **Parametry typu lista** – parametr szafki może być listą rozwijaną (np. „Bok widoczny / Bok niewidoczny"); wybrana opcja daje wartość liczbową równą pozycji na liście (0, 1, 2, …) do użycia we wzorach i zmiennych `r`
- Kodowanie szablonu (ANSI/UTF-16) jest zachowywane w plikach wynikowych
- Konfiguracja (szafki, parametry, narzędzia) zapisuje się do `ustawienia.json` + `szablony/` i przenosi gitem między komputerami

### Okleina krawędzi
Aplikacja odczytuje i wyświetla informacje o okleinowaniu krawędzi (EDGER, EDGEL, EDGET, EDGEB) z sekcji `SPEC{}` pliku.

### Ustawienia narzędzi
- Lista frezów: numer narzędzia (#205) → średnica (mm) – decyduje o szerokości ścieżki w rzutach 2D
- Lista piłek: numer narzędzia (#8516) → szerokość rowka (mm)
- Wykrywanie nowych narzędzi przy otwieraniu pliku z propozycją dodania do ustawień
- Konfiguracja skanera kodów kreskowych (maksymalny odstęp między znakami)

## Uruchomienie

Otwórz plik `index.html` bezpośrednio w przeglądarce – nie potrzeba serwera WWW ani żadnej instalacji.

### Wymagania przeglądarki

| Funkcja                                                  | Minimalna przeglądarka                             |
|----------------------------------------------------------|----------------------------------------------------|
| Rendering 3D (Three.js), widoki 2D                       | Chrome 80+, Edge 80+, Firefox 78+, Safari 14+      |
| Katalog plików + zapis do pliku (File System Access API) | **Chrome 86+ / Edge 86+** (Chromium)               |
| CSS `oklch()`, `color-mix()`                             | Chrome 111+, Edge 111+, Firefox 113+, Safari 15.4+ |

> Pełna funkcjonalność (w tym otwieranie katalogów i nadpisywanie plików) działa wyłącznie w **Chrome** lub **Edge** (i innych przeglądarkach opartych na Chromium). W Firefoksie i Safari zapis pliku ogranicza się do pobierania kopii przez „Zapisz jako…".

### Połączenie z internetem

Aplikacja ładuje dwie biblioteki z CDN przy pierwszym uruchomieniu:
- `cdnjs.cloudflare.com` – Three.js r128
- `cdn.jsdelivr.net` – OrbitControls
- `api.fontshare.com` – czcionka Satoshi (opcjonalna; bez niej używany jest font systemowy)

Po pierwszym załadowaniu strona działa offline (biblioteki mogą być w cache przeglądarki), o ile nie zostanie wyczyszczony cache.

## Format pliku

Aplikacja parsuje pliki `.TCN` w formacie TPA:
- `::UNm DL=… DH=… DS=…` – wymiary elementu
- `SIDE#N{…}SIDE` – bloki stron (1=TOP, 2=BOTTOM, 3=FRONT, 4=RIGHT, 5=BACK, 6=LEFT)
- `W#81{…}W` – nawiert
- `W#89{…}W` + `W#2201{…}W` – frezowanie (start + kolejne punkty)
- `W#1050{…}W` – piłka
- `W#2001{…}W` / `W#2005{…}W` / `W#2002{…}W` – IF / ELSE / ENDIF (pola: `#8070 ? #8073` z operatorem w `#8060`, termy 2–3: `#8071/#8074/#8061`, `#8072/#8075/#8062`, łączniki And/Or `#8065`/`#8066`)
- `VAR{…}VAR` – zmienne `r` (`#idx=wartość|nazwa|w|f|opis`)
- `SPEC{…}SPEC` – okleina krawędzi

Obsługiwane są pliki z separatorem dziesiętnym zarówno kropką, jak i przecinkiem – separator jest wykrywany automatycznie i zachowywany przy zapisie. Kodowanie pliku (ANSI/UTF-8 lub UTF-16 „unicode" z TpaCAD) jest wykrywane po BOM i zachowywane przy zapisie.

## Technologie

- Vanilla JS (ES2020, bez frameworków)
- [Three.js r128](https://threejs.org/) – rendering 3D
- File System Access API – zapis do pliku na dysku
- IndexedDB – trwałe przechowywanie ustawień narzędzi i ostatniego katalogu
- CSS custom properties z obsługą motywu jasnego i ciemnego
