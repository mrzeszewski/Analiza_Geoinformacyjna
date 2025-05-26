## Analiza wielokryterialna

Celem wykonywania przestrzennej analizy wielokryterialnej jest **porównanie wielu czynników** wpływających na interesujące nas zjawisko np. lokalizacje przedszkola. W jej rezultacie otrzymujemy informacje, które obszary są najmniej/najbardziej korzystne z punktu widzenia lokalizacji. 

W tym celu identyfikujemy co wpływa na zjawisko np. liczba ludności, odległość od istniejącego przedszkola, odległość od terenów zielonych etc. Każdemy **wskaźnikowi przypisujemy wagę** w ten sposób aby suma zamykała się w 1. np. 0,1a+0,3b+0,6c. Po zebraniu danych musimy je sprowadzić do wspólnego mianownika przestrzennego - bywa, że jest to jednostak administracyjna ale znacznie lepiej posłużyć się jednostkami mniej arbitralnymi np. **komórkami rastra** lub **polami siatki wektorowej**. 

## Przykład

Mamy **dane**:

1. Siatkę liczby ludność 200x200 m
2. Lokalizację terenów zieleni - plik wektorowy z OSM

Chcemy zobaczyć które miejsca na danym obszarze są najbardziej nam przydatne. **Zakładamy** że:

1. Im więcej ludzi tym lepiej, i jest to ważniejsza więc **waga** będzie wynosić **0,6**
2. Im bliżej terenów zieleni tym lepiej  ale jest to nieco mniej ważne więc **waga** to **0,4**

Potrzebujemy "wspólny mianownik przestrzenny" - jakoś musimy porównać oba czynniki pod względem lokalizacji. Możemy obie te warstwy zamienić na raster ale jak istnieje siatka to jest często dobrym pomysłem jej wykorzystanie lub stworzenie nowej (np. heksagonalnej). Tutaj **wykorzystamy siatkę liczby ludności** jako nasze "miejsca". 

Mamy liczbę ludności w siatce. Pozostaje nam przypisanie dla każdego oczka siatki w jakiej odległości znajduje się od obszarów zieleni. Można to zrobić liczbowo za pomocą bufora ale znacznie lepiej posłużyć się tzw. **rastrem sąsiedztwa (proximity raster)**. W tym celu obszary zielone musimy przekonwertować do postaci rastrowej:

1. *Raster/Konwersja/Rasteryzacja*
2. Wybieramy warstwę tereny zielone (jeśli ma układ EPSG:4326 musimy najpierw zapisać jako układ płaski np. EPSG:2180)
3. Wpisujemy wartość w polu *Określona wartość do nadania pikselom* np. 1
4. Wybieramy jako *Jednostki rozmiaru* opcję *Jednostki z georeferencją*
5.  *Szerokość i wysokość* ustawiamy na jednostki mniejsze niż oczka siatki (im mniej tym lepiej ale wielkość rastra i czaas analiz będzie szybko rosły) np. 100x100\
6. *Wyjściowy zasięg *ustawimy na zgodny z warstwą siatki
7. Uruchamiamy

Otrzymany raster posłuży nam do wykonania **rastra sąsiedztwa**:

1. *Raster/Analiza/Rastrowa mapa sąsiedztwa*
2. Jako warstwę wejściową wybieramy *raster z terenami zieleni*
3. W *liscie wartości pikseli* wpisujemy wartość którą wybraliśmy w punkcie 3 poprzedniego etapu (u nas było to 1).
4. Jako *jednostki odległości* wybieramy współrzędne z georeferencją
5. Uruchamiamy

Wartości z mapy sąsiedztwa trzeba przypisać do oczek siatki. Dokonujemy tego za pomocą narzędzia **Statystyki strefowe**. 

1. Szukamy funkcji *Statystyki strefowe* (w oknie w lewym rogu lub Ctrl+K)
2. Jako* Warstwę wejściową* wybieramy siatkę a jako *rastrową* raster sąsiedztwa.
3. *Przedrostek kolumny wynikowej* to będzie nazwa nowej kolumny w formacie np. NAZWA_mean. Warto to nazwać odpowiednio!
4. *Statystyki do obliczenia* - wybieramy *średnią* (w różnych przypadkach możemy chcieć różne statystki)
5. Uruchamiamy

Dostajemy nową warstwę, gdzie w tabeli atrybutów mamy zarówno liczbę ludności jak i wskaźnik średniej odległości od terenów zieleni. Jednak aby porównać wartości należy je **znormalizować** czyli doprowadzić do postaci 0-1, gdzie 1 to wartość pożądana.

Liczba ludności to wskaźnik pozytywny (im więcej tym lepiej) czyli wzór to
```
("liczba_ludnosci" - minimum("liczba_ludnosci")) / (maximum("liczba_ludnosci") - minimum("liczba_ludnosci"))
```
gdzie **liczba_ludnosci** to nazwa kolumny w tabeli atrybutów. Należy utworzyć *nowe pole w tabeli atrybutów* (**koniecznie z wartościami dziesiętnymi!**) przy pomocy **kalkulatorza pól** i wpisać tam powyższy wzór.

Odległość od terenów zieleni to wskaźnik negatywny więc musimy wynik odwrócić. Wzór będzie wyglądał następująco:
Liczba ludności to wskaźnik pozytywny (im więcej tym lepiej) czyli wzór to
```
1-("zielen" - minimum("zielen")) / (maximum("zielen") - minimum("zielen"))
```

Mając **obie wartości znormalizowane** wystarczy je zsumować za pomocą sumy ważonej. Czyli tworzymy nowe pole w tabeli atrybutów za pomocą wzoru (pamiętaj użyj nazwa kolumn z wartościami znormalizowanymi!):

("norm_liczba_ludnosci"*0.6)+("norm_zielen"*0.4)


**Otrzymana wartośc to wynik analizy.**









