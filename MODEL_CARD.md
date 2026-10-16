# BTOrganizer 2.0 — karta modelu

Model działa lokalnie na CPU i przypisuje fotografie przeglądów do sześciu kategorii: CW, SA, POMIARY, AC, EL i AZYMUTY. AWARIA pozostaje ręczna. Przepusty kablowe należą do CW. Fotografie z historycznych folderów AGREGAT/AGREGEGAT oraz SOLL/SOOL pominięto w danych modelu.

## Model i trening

- Enkoder obrazu: `timm/vit_base_patch16_siglip_224.v2_webli` (SigLIP2), Apache 2.0; dołączono licencję i oryginalne karty modelu.
- Dostrojono ostatnie 2 bloki enkodera, normalizację, pooling i głowicę sześciu klas. Rozmiar wejścia: 224 × 224; poprawka EXIF, RGB, resize i środkowy kadr.
- Trening wybranego modelu: 2 165 różnych zdjęć, wyłącznie z ręcznie zweryfikowanymi etykietami. Próbkowanie równoważyło klasy. Użyto łagodnych zmian jasności, kontrastu i nasycenia, bez odbić obrazu.
- 12 epok, około 45 minut. Najlepszy checkpoint: epoka 10. Wybór oparto na macro-F1 na 223 zdjęciach walidacyjnych; wynik wzrósł z 79,72% do 91,12%.
- Kalibrację temperatury wykonano na osobnych 150 zdjęciach walidacyjnych. Temperatura: 0,964106. Kategorie, identyfikatory stacji i nazwy plików nie są wejściami modelu; klasyfikator otrzymuje piksele obrazu.
- Train, walidacja i test rozdzielono według stacji, grup podobnych zdjęć i SHA-256. Ocena walidacji i testu używa wyłącznie ręcznie zatwierdzonych etykiet. Oryginały były otwierane tylko do odczytu, z weryfikacją zawartości.

Wcześniejsza próba trwała około 3 godziny 7 minut: 8 epok, ostatnie 4 bloki, 15 939 zdjęć, w tym 13 774 z niepotwierdzonymi etykietami historycznych folderów. Poprawiła ogólną klasyfikację, ale obniżyła trafność automatycznych przeniesień przy progu 90%; nie została wybrana do wydania. Zastąpiono ją opisanym wyżej treningiem wyłącznie na potwierdzonych etykietach. Łącznie eksperymenty trwały prawie 4 godziny.

## Ocena w ONNX Runtime CPU

| Model | Trafność ogólna | Macro-F1 |
|---|---:|---:|
| Bazowy 1.3.2 | 80,80% | 80,69% |
| Próba 1: etykiety mieszane (odrzucona) | 82,60% | 82,34% |
| 2.0: potwierdzone etykiety | 83,76% | 83,96% |

| Model | Próg | Przeniesione / 776 | Odsetek przeniesionych | Poprawność przeniesień | Błędne przypisania |
|---|---:|---:|---:|---:|---:|
| Bazowy 1.3.2 | 60% | 618 | 79,64% | 89,48% | 65 |
| Bazowy 1.3.2 | 90% | 350 | 45,10% | 95,71% | 15 |
| Bazowy 1.3.2 | 95% | 249 | 32,09% | 96,79% | 8 |
| Próba 1: etykiety mieszane (odrzucona) | 60% | 683 | 88,02% | 87,12% | 88 |
| Próba 1: etykiety mieszane (odrzucona) | 90% | 448 | 57,73% | 94,42% | 25 |
| Próba 1: etykiety mieszane (odrzucona) | 95% | 267 | 34,41% | 95,51% | 12 |
| 2.0: potwierdzone etykiety | 60% | 665 | 85,70% | 89,92% | 67 |
| 2.0: potwierdzone etykiety | 90% | 456 | 58,76% | 95,39% | 21 |
| 2.0: potwierdzone etykiety | 95% | 350 | 45,10% | 96,86% | 11 |

Odsetek przeniesionych oznacza udział zdjęć przekraczających wskazany próg. Poprawność przeniesień dotyczy tylko tych zaakceptowanych zdjęć. Zdjęcia poniżej progu pozostają w katalogu głównym do ręcznego przypisania. Wynik 90% lub 95% nie jest gwarancją trafności pojedynczego przypisania.

Porównanie używa tych samych 776 zdjęć testowych, tych samych etykiet i implementacji przetwarzania obrazu używanej przez aplikację. Zbiór ten był wcześniej oceniany w pierwszej próbie dostrajania; 195 zdjęć oceniano także przy wcześniejszych porównaniach modelu bazowego. Test nie służył do wyboru checkpointów ani dopasowania temperatury; użyto go do porównania kandydatów i decyzji o publikacji. Wyniki są diagnostyczne, nie stanowią nowego, wcześniej niewidzianego benchmarku. Próba pochodzi z ręcznej weryfikacji i ogranicza liczbę zdjęć jednej stacji/klasy; nie jest reprezentatywnym benchmarkiem pełnych przeglądów.

Pomiar 1 149 obrazów (walidacja i test) na komputerze treningowym: mediana 0,276 s/zdjęcie, 95. percentyl 0,334 s; 2 wątki CPU. Obejmuje odczyt, SHA-256, dekodowanie, EXIF, przygotowanie obrazu i inferencję; nie obejmuje pierwszego wczytania modelu, kompresji ani zmiany nazw. Nie jest to benchmark na docelowym i5.

Zgodność etykiet CPU z zapisanymi wynikami GPU: 1 148/1 149 (99,91%). Jedyna różnica dotyczyła niepewnego zdjęcia z wynikiem poniżej 36%. Dla progów 60%, 90% i 95% decyzje o przyjęciu zdjęć były zgodne. Maksymalna różnica wyniku wyniosła 0,24 punktu procentowego; przy progu 98% dwie decyzje graniczne różniły się. Tabela powyżej podaje rzeczywiste wyniki silnika CPU używanego w aplikacji.

## Ograniczenia i użytkowanie

Model może mylić zdjęcia zawierające kilka rodzajów urządzeń, podobne detale CW/SA/AC/EL oraz obrazy nietypowe dla przeglądów. Ręczne etykiety również mogą zawierać pomyłki lub subiektywne rozstrzygnięcia. Nie ma osobnej klasy „spoza domeny”, a AWARIA nie jest wykrywana automatycznie. Zdjęcia niejednoznaczne wymagają ręcznej kontroli.

Domyślny próg aplikacji to 95%; użytkownik może zmienić go w ustawieniach. To jawna polityka aplikacji. Pola `production_ready` i `automatic_sort_enabled` eksperymentalnego pakietu pozostają `false`; aplikacja stosuje wybrany przez użytkownika próg do pakietu z potwierdzoną kalibracją.

Zdjęcia, ich nazwy, identyfikatory, rejestry i checkpointy treningowe nie są publikowane. Instalator zawiera graf ONNX, konfigurację przetwarzania i licencje. Nie wymaga Python, PyTorch ani CUDA. Sortowanie nie kompresuje zdjęć i nie zmienia ich zawartości.

SHA-256 modelu: `d2fea492368de11a297f0487166029b8c91fb77024ddd0ceb99ae14bbb18e641`.
Źródła aplikacji: commit `f59b81afd30c4c195aac6aa39edf75099321ff5a`.
Narzędzia treningowe: commit `0cd87f54692e03bc037893b6093b7b199b28d388`.
