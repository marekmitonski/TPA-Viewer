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
- `SPEC{…}SPEC` – okleina krawędzi

Obsługiwane są pliki z separatorem dziesiętnym zarówno kropką, jak i przecinkiem – separator jest wykrywany automatycznie i zachowywany przy zapisie.

## Technologie

- Vanilla JS (ES2020, bez frameworków)
- [Three.js r128](https://threejs.org/) – rendering 3D
- File System Access API – zapis do pliku na dysku
- IndexedDB – trwałe przechowywanie ustawień narzędzi i ostatniego katalogu
- CSS custom properties z obsługą motywu jasnego i ciemnego
