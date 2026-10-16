# BTOrganizer 2.0

Program do automatyzacji przygotowywania dokumentacji przeglądów BT.

## Pobieranie

Pobierz [instalator BTOrganizer 2.0 z modelem](https://github.com/delorian9/BTOrganizer-Releases/releases/download/v2.0/BTOrganizer-2.0-Setup.exe) z [wydania 2.0](https://github.com/delorian9/BTOrganizer-Releases/releases/tag/v2.0). Instalator zawiera program, lokalny model sortowania zdjęć i licencje zależności. Nie wymaga instalacji Python ani pobierania modelu. System: Windows 10/11, 64 bit.

Instalacja odbywa się dla bieżącego użytkownika. Dane aplikacji, licencja i szablony pozostają w AppData po odinstalowaniu. Sumę kontrolną instalatora podaje plik `SHA256SUMS.txt` dołączony do wydania.

Przed uruchomieniem 2.0 zamknij wcześniejsze okna BTOrganizer, aby dwie wersje nie przetwarzały jednocześnie tego samego przeglądu.

## Nowości 2.0

- **Sortuj zdjęcia**: lokalne przypisanie zdjęć z katalogu głównego do CW, SA, POMIARY, AC, EL i AZYMUTY. AWARIA pozostaje ręczna; przepusty kablowe należą do CW.
- Próg przenoszenia w ustawieniach, domyślnie 95%; oczekiwanie po kopiowaniu domyślnie 2 sekundy.
- Pytanie o nową partię zdjęć wyświetla się na środku ekranu.
- Rejestr i raporty znajdują się poza folderem przeglądu. Program przenosi własne metadane ze starego `.btorganizer`, zachowując historię potrzebną do wznowienia operacji i uniknięcia ponownej kompresji.
- Brak automatycznego skanowania poprzedniego przeglądu po restarcie.
- Sortowanie zachowuje zawartość zdjęć, nie nadpisuje plików o takich samych nazwach i pozostawia zdjęcia poniżej progu do ręcznego przypisania.

Wynik modelu nie jest gwarancją trafności. Opis treningu, wyniki porównania i ograniczenia znajdują się w [karcie modelu](MODEL_CARD.md). Zdjęcia i rejestry treningowe nie są publikowane.

## Funkcje

- generowanie dokumentów Excel,
- uzupełnianie placeholderów,
- zmiana nazw zdjęć,
- naprawa numeracji zdjęć,
- kompresja zdjęć przez ImageMagick,
- obsługa dokumentu Soll,
- obsługa obiektów z bateriami.

## Autor

Dorian Drzewiecki
