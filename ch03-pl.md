# Rozdział 03: Czysta radość z czystych funkcji

## Mam czystą buzię, jestem uczesany, na koszulce nie mam żadnej tłustej plamy

Jedna rzecz, którą natychmiast musimy sobie wyjaśnić to idea czystych funkcji.

> Czysta funkcja to taka funkcja, która, gdy otrzyma ten sam zestaw danych wejściowych, zawsze zwróci dokładnie ten sam wynik oraz nie posiada żadnych obserwowalnych skutków ubocznych.

Weźmy `slice` i `splice`. Są to dwie funkcje, które z grubsza robią to samo, co prawda w zasadniczo inny sposób, no ale jednak. Mówi się, że `slice` jest *czysta* ponieważ zwraca ten sam rezultat dla tego samego argumentu, kropka. Natomiast `splice` przeżuwa podaną tablicę i wypluwa na zawsze odmienioną – to właśnie jest obserwowalny skutek uboczny.

```js
const xs = [1,2,3,4,5];

// czysta
xs.slice(0,3); // [1,2,3]

xs.slice(0,3); // [1,2,3]

xs.slice(0,3); // [1,2,3]


// nieczysta
xs.splice(0,3); // [1,2,3]

xs.splice(0,3); // [4,5]

xs.splice(0,3); // []
```

W programowaniu funkcyjnym nie lubimy się z funkcjami takimi jak `splice`, które *mutują* dane. Dążymy do rzetelnych funkcji, na których można polegać i po których zawsze wiemy czego się spodziewać, a nie funkcji jak `splice`, które wiecznie zostawiają za sobą bałagan.

Spójrzmy na kolejny przykład:

```js
// nieczysta
let minimum = 21;
const checkAge = age => age >= minimum;

// czysta
const checkAge = (age) => {
  const minimum = 21;
  return age >= minimum;
};
```

W nieczystym kodzie, `checkAge` korzysta z mutowalnej wartości `minimum`, aby zwrócić wynik. Innymi słowy, funkcja ta zależy od stanu systemu (od kontekstu), co jest niezadowalające, gdyż znacząco zwiększa [obciążenie poznawcze](https://en.wikipedia.org/wiki/Cognitive_load).

W powyższym przykładzie może to wyglądać na błahy problem, jednak uzależnienie od stanu jest jednym z najważniejszych elementów przyczyniających się do wzrostu złożoności systemu (http://curtclifton.net/papers/MoseleyMarks06a.pdf). `checkAge` w pierwszym wariancie może zwracać różne wyniki w zależności od czynników zewnętrznych, a to nie tylko dyskwalifikuje ją z określenia mianem „czysta”, ale też przepycha nasz mózg przez wyżymaczkę ilekroć chcemy w myślach ustalić rezultat wywołania tej funkcji.

Natomiast czysta postać `checkAge` jest całkowicie samowystarczalna.

## Skutki uboczne mogą…

Przyjrzyjmy się bliżej tym „skutkom ubocznym”, aby poprawić naszą intuicję. Czymże jest jest ten nikczemny *skutek uboczny* wspomniany w definicji *czystej funkcji*? Skutkiem (albo efektem) nazwiemy wszystko, co nie jest bezpośrednio związane z wyliczeniem wyniku funkcji.

Nie ma nic inherentnie złego w samych skutkach i będziemy ich używać na każdym kroku w nadchodzących rozdziałach. To słówko *uboczny* niesie ze sobą negatywne skojarzenia. Woda sama w sobie nie zawsze jest siedliskiem komarów. Dopiero „stojąca woda” staje się idealnym środowiskiem dla milionów krwiopijców. Zapewniam cię, że efekty *uboczne* są podobnym terenem lęgowym dla wszelkiego robactwa w Twoich programach.

> *Skutek uboczny* to zmiana stanu systemu lub *obserwowalna interakcja* ze światem zewnętrznym, która ma miejsce podczast obliczania wyniku funkcji.

Skutki uboczne mogą zawierać, ale nie ograniczają się do:

  * modyfikowania plików
  * zapisywania rekordów w bazie danych
  * wykonania żądania HTTP
  * mutacji
  * wypisywania na ekranie (logowania)
  * pobierania danych od użytkownika
  * szukania elementów DOM
  * odczytawania stanu systemu

Lista jest długa. Jakakolwiek interakcja ze światem poza ciałem funkcji jest skutkiem ubocznym i jest to fakt, który może skłonić Cię do podważania praktyczności programowania bez skutków ubocznych w ogóle. Filozofia programowania funkcyjnego postuluje, że skutki uboczne są głównym źródłem błędów w aplikacjach.

To jednak nie jest tak, że mamy całkowity zakaz używania ich. Raczej chcielibyśmy umieć kontrolować skutki uboczne i z nich korzystać. Nauczymy się jak to zrobić, gdy dotrzemy do funktorów i monad w przyszłych rozdziałach, a na razie postarajmy się trzymać te podstępne funkcje ze skutkami ubocznymi z dala od naszych czystych funkcji.

Skutki uboczne dyskwalifikują funkcję z bycia *czystą*. Ma to sens: czyste funkcje, z definicji, muszą zwracać zawsze ten sam wynik dla takich samych danych, a tego po prostu nie można zagwarantować, jeśli funkcja dodatkowo korzysta ze stanu dookoła niej.

Skupmy się na tym, dlaczego tak bardzo zależy nam na tym, żeby funkcja zwracała ten sam wynik dla tego samego wejścia. Nakładaj mundurek, rzucimy okiem na matematykę na poziomie gimnazjum (gimby znajo).

## Matematyka z gimnazjum

Za epodreczniki.pl:

> Funkcją f ze zbioru X w zbiór Y nazywamy przyporządkowanie
> w którym każdemu elementowi zbioru X przyporządkowany jest dokładnie jeden element zbioru Y.

Innymi słowy, funkcja to relacja pomiędzy dwoma wartościami: wejściem a wyjściem. Każda dana wejściowa na pewno ma dokładnie jeden wynik, to jednak ten sam wynik może się powtarzać dla wielu różnych argumentów. Poniżej diagram całkowicie poprawnej funkcji z `x` do `y`:

<img src="images/function-sets.gif" alt="function sets" />(https://www.mathsisfun.com/sets/function.html)

Natomiast kolejny diagram przedstawia relację, która *nie* jest funkcją, gdyż wejściu `5` przyporządkowane jest kilka wyników:

<img src="images/relation-not-function.gif" alt="relation not function" />(https://www.mathsisfun.com/sets/function.html)

Funkcje można również zapisać jako zbiór par (wejście, wyjście): `[(1,2), (3,6), (5,10)]` (wygląda na to, że akurat ta funkcja podwaja podany argument).

Można też przedstawić ją jako tabelkę:
<table> <tr> <th>Wejście</th> <th>Wyjście</th> </tr> <tr> <td>1</td> <td>2</td> </tr> <tr> <td>2</td> <td>4</td> </tr> <tr> <td>3</td> <td>6</td> </tr> </table>

A nawet jako wykres, w którym `x` jest wejściem, a `y` rezultatem:

<img src="images/fn_graph.png" width="300" height="300" alt="function graph" />

Nie ma nawet potrzeby zagłębiania się w szczegóły implementacyjne jeśli to dane wejściowe determinują rezultat. Skoro funkcja to zwykłe mapowanie z wejścia na wyjście, to moglibyśmy równie dobrze użyć literałów obiektów i `[]` zamiast `()`:

```js
const toLowerCase = {
  A: 'a',
  B: 'b',
  C: 'c',
  D: 'd',
  E: 'e',
  F: 'f',
};
toLowerCase['C']; // 'c'

const isPrime = {
  1: false,
  2: true,
  3: true,
  4: false,
  5: true,
  6: false,
};
isPrime[3]; // true
```

Oczywiście, możesz chcieć wyliczyć wynik zamiast zapisywać ręcznie wszystkie możliwości, ale chciałem Ci pokazaż inny sposób myślenia o funkcjach. (Jeśli zastanawiasz się teraz „a co z funkcjami, które przyjmują wiele argumentów?”, to już odpowiadam. Rzeczywiście może to rodzić dla nas pewne problemy, gdy będziemy myśleć o funkcjach na gruncie matematyki. Na tym etapie załóżmy, że funkcje wielu argumentów przyjmują po prostu tablicę wartości (albo obiekt `arguments`). Gdy poznamy pojęcie *rozwijania funkcji* (ang. _currying_ na cześć matematyka Haskella Curry'ego), to stanie się jasne, w jaki sposób można matematycznie opisać takie funkcje)

Ujawnię teraz dramatyczną prawdę: czyste funkcje *są* matematycznymi funkcjami i to wszystko o co chodzi w programowaniu funkcyjnym. Programowanie z wykorzystaniem tych słodkich aniołków przynosi nam ogromne benefity. Zastanówmy się, jakie są powody, dla których jesteśmy w stanie wkładać tyle wysiłku w zachowanie czystości.

## Sprawa czystości

### Zapamiętywnanie (cache)

Na początek zwróćmy uwagę, że w przypadku czystych funkcji zawsze można _cache'ować_ wyniki na podstawie argumentów. Zwyczajowo nazywa się to memoizacją:

```js
const squareNumber = memoize(x => x * x);

squareNumber(4); // 16

squareNumber(4); // 16, cache dla 4

squareNumber(5); // 25

squareNumber(5); // 25, cache dla 5
```

Poniżej prosta implementacja. Łatwo znaleźć też znacznie bardziej rozbudowane.

```js
const memoize = (f) => {
  const cache = {};

  return (...args) => {
    const argStr = JSON.stringify(args);
    cache[argStr] = cache[argStr] || f(...args);
    return cache[argStr];
  };
};
```

Warto też zanotować, że nieczyste funkcje można zamienić w czyste funkcje przez odwleczenie ich wykonania i zapakowanie w jeszcze jedną funkcję:

```js
const pureHttpCall = memoize((url, params) => () => $.getJSON(url, params));
```

Nie wykonujemy tutaj żadnego zapytania HTTP. Zamiast tego zwracamy funkcję, która dopiero wykona żądanie, gdy ją wywołamy. Ta funkcja jest czysta, gdyż zawsze zwróci ten sam wynik dla tych samych argumentów: zmemoizowaną funkcję, która wykona odpowiednie zapytanie HTTP używając `url` i `params`.

Nasze `memoize` działa w porządku, chociaż nie zapamiętuje wyników żądań HTTP, tylko tworzone funkcje.

Nie jest to jeszcze jakoś superużyteczne, ale wkrótce poznamy triki, które sprawią, że ta wiedza nam się przyda. Na razie zapamiętaj jedno: możemy memoizować każdą funkcję, niezależnie od tego, co robi.

### Przenośna / Samodokumentująca

Czyste funkcje są całkowicie samowystarczalne. Wszystko, czego funkcja potrzebuje zostaje jej dostarczone na srebrnej tacy. Moment, ale w jaki sposób to może być korzystne dla nas? No na przykład, wszystkie zależności danej funkcji są jasne, klarowne i od razu widoczne, a dzięki temu łatwiej ją zrozumieć – żadnych królików z kapelusza.

```js
// nieczysta
const signUp = (attrs) => {
  const user = saveUser(attrs);
  welcomeUser(user);
};

// czysta
const signUp = (Db, Email, attrs) => () => {
  const user = saveUser(Db, attrs);
  welcomeUser(Email, user);
};
```

Przykład powyżej pokazuje, że czyste funkcje zawsze są szczere odnośnie tego, na czym im zależy i mówią dokładnie co mają zamiar robić. Tylko na podstawie samej deklaracji, wiemy, że ta funkcja będzie korzystała z `Db`, `Email` i `attrs`, a to jest, wydaje mi się, dość wymowne.

Nauczymy się, jak zamieniać nieczyste funkcje w czyste bez śmiesznego opóźniania wywołania, ale mam nadzieję, że już teraz jest dobrze widoczny fakt, że czysta funkcja jest znacznie bardziej czytelna i niesie ze sobą więcej informacji, niż chytry i nieczysty odpowiednik, który ma nie wiadomo co w planach.

Inna warta uwagi kwestia to fakt, że jesteśmy niejako zmuszeni do „wstrzyknięcia” zależności (przekazania ich jako argumenty) co sprawia, że nasza aplikacja staje się znacznie bardziej elastyczna. Możemy parametryzować połączenie z bazą danych, albo klienta poczty, albo cokolwiek innego przed przekazaniem ich do funkcji (nie martw się, tylko na razie brzmi to jak kompletna nuda). Chcemy skorzystać z innej bazy danych? Wystarczy wywołać naszą funkcję z innym argumentem. Chcemy wykorzystać tę samą rzetelną funkcję w innym projekcie? No to po prostu przekażemy jej inne `Db` i inny `Email`.

Przenośność w JavaScripcie można oznaczać na przykład możliwość serializowania funkcji i przesłania ich w żądaniu. Albo wywołanie tego samego kodu w Web Workers. Przenośność to potężny przymiot.

W odróżnieniu od „typowych” metod i procedur w programowaniu imperatywnym, które są głęboko zakorzenione w swoim środowisku poprzez stan, zależności, skutki uboczne – czyste funkcje możemy przenosić i wywoływać gdzie nam w duszy gra.

Kiedy ostatni raz zdarzyło Ci się skopiować jakąś metodę do nowej aplikacji? Jeden z moich ulubionych cytatów pochodzi od twórcy języka Erlang, Joe Armstronga: „Problem z językami obiektowymi jest taki, że mają to całe ukryte środowisko, które noszą ze sobą. Chcesz banana, ale dostajesz goryla trzymającego banana… i całą dżunglę.” (ang. _"The problem with object-oriented languages is they’ve got all this implicit environment that they carry around with them. You wanted a banana but what you got was a gorilla holding the banana... and the entire jungle"_)

### Testowalna

Następnie, dochodzimy do wniosku, że testowanie czystych funkcji jest znacznie proste. Nie musimy _mockować_ żadnej bramki płatności albo tworzyć całego globalnego stanu przed każdym testem. Po prostu dajemy coś funkcji i sprawdzamy, czy rezultat jest poprawny.

Faktem jest, że to społeczność funkcyjna jest pionierem jeśli chodzi o nowe narzędzia do testowania, które mogą wymłócić nasze funkcje losowymi danymi i sprawdzić, czy wszystkie własności się zgadzają. To zdecydowanie wychodzi poza zakres niniejszej książki, ale bardzo mocno polecam Ci poszukać i spróbować *Quickcheck* – narzędzia do testowania, które zostało stworzone pod funkcyjne środowisko (nota od tłumacza: istnieje również wersja tej biblioteki do JavaScript i nazywa się *fast-check*).

### Sensowna

Wiele osób sądzi, że najważniejsza przy pracy z czystymi funkcjami jest *referential transparency* (nota od tłumacza: nie podejmuję się próby przetłumaczenia, gdyż przyniosłoby to więcej szkody niż pożytku). Fragment kodu nazywamy _referentially transparent_, gdy możemy go wykonać i podmienić na wynik bez zmiany zachowania aplikacji.

Skoro czyste funkcje nie mają żadnych skutków ubocznych, to jedyny wpływ na program jaki mogą mieć, to poprzez swoje wyjście. Ponadto, skoro ich wyjście jest zawsze takie samo dla danego wejścia, to czyste funkcje zawsze zachowują cechę _referential transparency_. Przykładzik:


```js
const { Map } = require('immutable');

// Aliasy: p = player, a = attacker, t = target
const jobe = Map({ name: 'Jobe', hp: 20, team: 'red' });
const michael = Map({ name: 'Michael', hp: 20, team: 'green' });
const decrementHP = p => p.set('hp', p.get('hp') - 1);
const isSameTeam = (p1, p2) => p1.get('team') === p2.get('team');
const punch = (a, t) => (isSameTeam(a, t) ? t : decrementHP(t));

punch(jobe, michael); // Map({name:'Michael', hp:19, team: 'green'})
```

`decrementHP`, `isSameTeam` i `punch` są czystymi funkcjami, a więc są _referentially transparent_. Możemy skorzystać z techniki zwanej *śledzeniem równania* (ang. _equantional reasoning_), w której podstawia się „równe za równe”, aby łatwiej rozumować na temat kodu. To trochę jakbyśmy w głowie próbowali wykonać kod bez brania pod uwagę dziwactw kompilatora. Korzystając z _referential transparency_, spróbujemy się pobawić i zastosować tę technikę na kodzie powyżej.

Najpierw otwórzmy funkcję `isSameTeam` i wstawmy jej kod bezpośrednio w `punch`:

```js
const punch = (a, t) => (a.get('team') === t.get('team') ? t : decrementHP(t));
```

Wiemy, że nasze dane są niemutowalne, więc możemy podmienić `.get('team')` na wartości:

```js
const punch = (a, t) => ('red' === 'green' ? t : decrementHP(t));
```

Widzimy, że warunek nie jest spełniony, więc możemy pozbyć się całej pierwszej części funkcji:

```js
const punch = (a, t) => decrementHP(t);
```

A teraz jeśli otworzymy i podmienimy `decrementHP` to zobaczymy, że, w tym przypadku, `punch` to tylko zmiana wartości `hp` o 1 w dół:

```js
const punch = (a, t) => t.set('hp', t.get('hp') - 1);
```

Możliwość myślenia o kodzie w taki sposób jest niesamowicie przydatna do refaktoryzacji albo ogólnie przy próbie zrozumienia programu. Użyliśmy tej samej techniki, aby zrefaktorować naszą mewią funkcję w pierwszym rozdziale. Tam użyliśmy śledzenia równania, aby okiełznać własności dodawania i mnożenia. Tej techniki będziemy używać często dalej w książce.

### Kod równoległy

Wreszcie, oto _coup de grâce_, wszystkie czyste funkcje można uruchamiać równolegle, gdyż nie mają dostępu do współdzielonej pamięci i z definicji nie mogą mieć hazardów z powodu jakiegoś skutku ubocznego.

Jest to jak najbardziej możliwe po stronie serwera w wątkach (NodeJS) albo w przeglądarkach przez Web Workers, chociaż obecnie społeczność JS zdaje się tego tematu unikać ze względu na złożoność aplikacji przy równoległym uruchamianiu nieczystych funkcji.

## Podsumowując

Wiemy czym są czyste funkcje i dlaczego my, funkcyjni programiści i funkcyjne programistki, wierzymy, że są wisienką na torcie programowania. Odtąd będziemy dążyć do pisania wyłącznie czystych funkcji. Potrzebujemy jeszcze kilku narzędzi, aby móc ten cel zrealizować w stu procentach, ale póki ich nie posiadamy, to po prostu będziemy oddzielać nieczyste funkcje od reszty czystego kodu.

Pisanie całych programów z czystymi funkcjami jest ździebko męczące bez odpowiednich narzędzi pod ręką. Musimy żonglować danymi przekazując wszędzie wszystkie argumenty, zakazano nam używania stanu, o skutkach ubocznych nie wspominając. To wy tu tak żyjecie? Jak można pisać tak masochistyczny kod? Poznajmy nową zabawkę nazwaną curry.

[Rozdział 04: Currying](ch04-pl.md)
