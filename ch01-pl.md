# Rozdział 01: Co my tu w ogóle robimy?

## Wprowadzenie

Cześć! Jestem profesor Franklin Frisby i jest mi niezmiernie miło Cię poznać. Spędzimy razem trochę czasu, bo podobno mam Cię nauczyć nieco programowania funkcyjnego. No, to tyle o mnie, teraz skupmy się na Tobie. Mam nadzieję, że znasz chociaż trochę JavaScript, masz zdeczko doświadczenia z programowaniem orientowanym obiektowo i szczycisz się zawodem programisty bądź programistki. Nie musisz mieć doktoratu z entomologii, ale musisz wiedzieć jak znaleźć i zabić niektóre _bugi_.

Nie będą zakładać, że posiadasz jakiekolwiek doświadczenie z programowaniem funkcyjnym, bo wszyscy wiemy co się dzieje, gdy się za dużo założy, no nie? Natomiast spodziewam się, że na pewno przydarzyło Ci się znaleźć w wielu nieprzyjemnych sytuacjach związanych z mutowalnym stanem, niczym nieograniczonymi efektami ubocznymi i złą architekturą kodu. Skoro już się poznaliśmy, to zaczynajmy!

Celem tego rozdziału jest uświadomienie Ci, jaki jest powód pisania kodu funkcyjnie i czego poszukujemy. Aby zrozumieć kolejne rozdziały, konieczne jest rozumienie co to w ogóle oznacza, że program jest *funkcyjny*. Bez tego możemy łatwo znaleźć się w sytuacji, w której piszemy kod bez głębszego przemyślenia, po prostu unikając używania obiektów, a to nie będzie najmądrzejsze przedsięwzięcie. Musimy widzieć jasno i wyraźnie cele i powody pisania kodu w stylu funkcyjnym.

Są pewne ogólne zasady programowania; różne skrótowce, które mają nas prowadzić przez meandry tworzenia aplikacji: DRY (_don't repeat yourself_, czyli nie powtarzaj kodu), YAGNI (_ya ain't gonna need it_, czyli nie twórz rzeczy niepotrzebnych),  _loose coupling high cohesion_ (mało silnych powiązań, ale dużo spójności), _the principle of least surprise_ (zasada tworzenia kodu, który nie zaskakuje), SRP (_single responsibility principle_, czyli nigdy nie powinno być więcej niż jednego powodu do zmiany danego kodu) i tak dalej…

Nie będę Cię zanudzał próbując wypisać każdą złotą zasadę, o której słyszałem przez te wszystkie lata… Chodzi mi tylko o to, że te wszystkie pojęcia nadal mają sens w programowaniu funkcyjnym, mimo że nie są one naszym głównym celem.

Zanim przejdziemy dalej, chciałbym aby było jasne, jaka jest nasza intencja, gdy klepiemy w klawiaturę; nasza funkcyjna bursztynowa komnata.

<!--BREAK-->

## Pierwsze spotkanie

Zacznijmy od źdźbła absurdu: mamy tutaj aplikację dla mew. Gdy stada ptaków się łączą, to tworzą większe stada, a gdy się rozmnażają, to ich liczba rośnie proporcjonalnie do liczby biorących w tym udział. Ważne! To co za chwilę zobaczysz bynajmniej nie ma być przykładem wspaniałego obiektowego kodu. Chcę tylko podkreślić najważniejsze aspekty współczesnego popularnego programowania. Spójrz:

```js
class Stado {
  constructor(n) {
    this.mewy = n;
  }

  połącz(inne) {
    this.mewy += inne.mewy;
    return this;
  }

  rozmnażaj(inne) {
    this.mewy = this.mewy * inne.mewy;
    return this;
  }
}

const stadoA = new Stado(4);
const stadoB = new Stado(2);
const stadoC = new Stado(0);
const result = stadoA
  .połącz(stadoC)
  .rozmnażaj(stadoB)
  .połącz(stadoA.rozmnażaj(stadoB))
  .mewy;
// 32
```

Po kiego grzyba ktoś miałby stworzyć taką upiorną obrzydliwość?! Nadążanie za zmianami w mutowalnym stanie w środku tej klasy jest niezwykle trudne. I do chrzanu z taką robotą, bo przecież nawet wynik jest błędny! Powinno wyjść `16`, ale `stadoA` zostało po drodze zmutowane. Biedne `stadoA`. To jest anarchia w IT! Dzika zwierzęca arytmetyka!

Jeśli nie rozumiesz powyższego kodu, to nic, bo ja też go nie rozumiem. Sednem jest to, żeby pamiętać, że mutowalny stan jest bardzo trudny do ogarnięcia, nawet w tak małej aplikacji.

Spróbujmy jeszcze raz, ale tym razem w bardziej funkcyjny sposób:

```js
const połącz = (stadoX, stadoY) => stadoX + stadoY;
const rozmnażaj = (stadoX, stadoY) => stadoX * stadoY;

const stadoA = 4;
const stadoB = 2;
const stadoC = 0;
const result =
    połącz(rozmnażaj(stadoB, połącz(stadoA, stadoC)), rozmnażaj(stadoA, stadoB));
// 16
```

Cóż, tym razem wynik jest prawidłowy. I kodu znacznie mniej. To zagnieżdżenie funkcji jest jednak nieco mylące (ale poradzimy coś na to w rozdziale 5). Na pewno to rozwiązanie jest lepsze od poprzedniego, ale przyjrzyjmy się temu nieco bardziej. Są powody, aby nazywać rzeczy po imieniu, a gdybyśmy zbadali nasze funkcje bliżej, to odkrylibyśmy, że to, co tutaj robimy, to zwykłe dodawanie (`połącz`) i mnożenie (`rozmnażaj`).

Nie ma absolutnie nic szczególnego w tych dwóch funkcjach poza ich nazwami. Przemianujmy je na `add` i `multiply`, aby odsłonić ich prawdziwe tożsamości (nota od tłumacza: od tej pory będziemy używać raczej angielskich nazw zmiennych i funkcji).

```js
const add = (x, y) => x + y;
const multiply = (x, y) => x * y;

const stadoA = 4;
const stadoB = 2;
const stadoC = 0;
const result =
    add(multiply(stadoB, add(stadoA, stadoC)), multiply(stadoA, stadoB));
// 16
```

Tym sposobem zdobywamy wiedzę starożytnych:

```js
// łączność (associative)
add(add(x, y), z) === add(x, add(y, z));

// przemienność (commutative)
add(x, y) === add(y, x);

// element neutralny (identity)
add(x, 0) === x;

// rozdzielność (distributive)
multiply(x, add(y,z)) === add(multiply(x, y), multiply(x, z));
```

Ah tak, stare poczciwe zasady matematyki mogą nam się przydać. Nie martw się, jeśli nie masz ich wykutych na blachę. Dla większości z nas będzie to dobrych kilka lat od ostatniego spotkania z prawami arytmetyki. Sprawdźmy, czy uda nam się jakoś wykorzystać tę wiedzę do uproszczenia naszego mewiego programu:

```js
// Oryginalnie
add(multiply(stadoB, add(stadoA, stadoC)), multiply(stadoA, stadoB));

// Element neutralny (stadoC) sprawia, że możemy usunąć zbędne add
// (add(stadoA, stadoC) == stadoA)
add(multiply(stadoB, stadoA), multiply(stadoA, stadoB));

// Rozdzielność mnożenia względem dodawania
multiply(stadoB, add(stadoA, stadoA));
```

Genialnie! Nie musieliśmy pisać ani kawałeczka nowego kodu poza wywołaniami naszych funkcji. Zdefiniowaliśmy wcześniej `add` i `multiply` dla spójności tego przykładu, ale biblioteki standardowe większości języków programowania na pewno mają te funkcje już wbudowane.

Możesz teraz myśleć „przeinaczyłeś argumenty przeciwnika, aby łatwiej było ci go zaatakować i pokazałeś taki matematyczny przykład” albo „prawdziwe programy tak nie wyglądają i nie da się ich tak łatwo przekształcać”. Wybrałem ten przykład celowo, bo większość z nas rozumie dodawanie i mnożenie, więc prosto było dostrzec, że matematyka jest tutaj dla nas bardzo przydatna.

Nie rozpaczaj! W tej książce znajdziesz fragmenty teorii kategorii, teorii zbiorów, rachunku lambda i osiągniesz podobny poziom prostoty w prawdziwych, z życia wziętych programach, jak w aplikacji dla mew. Ale to nie oznacza, że konieczna jest magisterka z matematyki! Wszystko będzie się wydawało naturalne i proste, zupełnie jak korzystanie z nowego „normalnego” frameworka lub API.

Dla wielu będzie zaskoczeniem, że da się pisać rozbudowane aplikacje w sposób funkcyjny podobny do tego powyżej. Aplikacje, które są poprawne (ang. _sound_) . Aplikacje, które są zwięzłe, ale proste do zrozumienia. Aplikacje, które nie wymyślają koła na nowo na każdym rogu. Samowolka jest dobra dla przestępców, ale w tej książce będziemy chcieli poznać i skrupulatnie przestrzegać praw matematyki.

Będziemy korzystać z teorii, w których każdy element pasuje do siebie jak ulał. Będziemy przedstawiać konkretne problemy w postaci uniwersalnych, małych i komponowalnych kawałków, a później wykorzystywać ich własności do cna tylko dla naszych niecnych celów. Będzie to wymagało nieco więcej dyscypliny niż typowe „zrób żeby działało” z programowania imperatywnego (dokładną definicję słowa „imperatywny” omówimy dalej, a na razie przyjmijmy, że imperatywne jest wszystko to, co nie jest funkcyjne). Praca z prawami matematyki przyniesie Ci tak gigantyczny zwrot z inwestycji, że zamienisz się w słup soli.

Widzieliśmy migotanie gwiazdy polarnej programowania funkcyjnego. Teraz musimy dokładnie omówić kilka teoretycznych pojęć, aby móc wyruszyć w dalszą podróż.

[Rozdział 02: Funkcje pierwszej klasy](ch02-pl.md)
