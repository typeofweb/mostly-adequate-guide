# Rozdział 02: Funkcje pierwszej klasy

## Szybki przegląd
Kiedy mówimy, że funkcje są „pierwszej klasy” oznacza to, że są zwykłymi ludźmi, zawsze mówią „dzień dobry”… Innymi słowy, „funkcja pierwszej klasy” oznacza „normalna funkcja”. Chodzi o to, że funkcje możemy traktować tak samo, jak każdy inny typ danych i nie ma w nich nic szczególnie wyjątkowego. Możemy je trzymać w tablicy, przekazywać jako parametry, przypisywać do zmiennych i co tam jeszcze chcesz.

To są podstawy JavaScriptu, ale warto o tym wspomnieć, gdyż szybkie wyszukanie na GitHubie pokazuje, że istnieje jakiś zbiorowy wykręt, albo może rozpowszechniona ignorancja, odnośnie tego konceptu. Może przykład?

```js
const hi = name => `Hi ${name}`;
const greeting = name => hi(name);
```

Powyżej widzimy, że tworzenie kolejnej funkcji dla `greeting` jest kompletnie zbędne. Dlaczego? Bo funkcje w JavaScripcie są *wywoływalne* (ang. _callable_). Oznacza, to, że gdy po `hi` pojawi się `()`, to funkcja zostanie wykonana i zwróci wartość. Natomiast, jeśli `()` nie będzie, to dostaniemy funkcję przypisaną do zmiennej o nazwie `hi`. Aby się upewnić, sprawdźmy:


```js
hi; // name => `Hi ${name}`
hi("jonas"); // "Hi jonas"
```

Skoro jedyne co robi `greeting` to przyjęcie parametrów i wywołanie z nimi `hi`, to możemy uprościć:

```js
const greeting = hi;
greeting("times"); // "Hi times"
```

Mówiąc inaczej, skoro `hi` jest już funkcją, która oczekuje jednego argumentu, po co tworzyć kolejną funkcję dookoła niej, która tylko wywoła `hi` z tym samym cholernym argumentem?! To nie ma żadnego sensu. To tak, jakby ubrać najgrubszą kurtkę w sierpniu tylko po to, żeby domagać się loda na patyku.

Jak to zwykle bywa, nie tylko jest to okropnie rozwlekłe, ale to też po prostu zła praktyka, aby otaczać funkcję kolejną funkcją tylko po to, żeby odwlec jej wykonanie (za moment wyjaśni się dlaczego, ale chodzi o trudność w utrzymaniu kodu).

Dogłębne zrozumienie powyższych akapitów jest niezbędne, abyśmy mogli iść dalej, więc przyjrzyjmy się kilku innym przykładom, które wykopałem z czeluści bibliotek w NPM.

```js
// ignorancja
const getServerStuff = callback => ajaxCall(json => callback(json));

// oświecenie
const getServerStuff = ajaxCall;
```

Świat został zaśmiecony kodem dokładnie takim jak ten. Śpieszę wyjaśnić czemu obie funkcje powyżej oznaczają dokładnie to samo:

```js
// ta linijka
ajaxCall(json => callback(json));

// to to samo co po prostu
ajaxCall(callback);

// więc wracając do getServerStuff
const getServerStuff = callback => ajaxCall(callback);

// ...możemy zrobić o tak
const getServerStuff = ajaxCall; // <-- patrz mamo, programuję bez nawiasów!
```

Tak to się robi! Jeszcze raz, żeby było jasne, dlaczego jestem z tym tak natrętny:

```js
const BlogController = {
  index(posts) { return Views.index(posts); },
  show(post) { return Views.show(post); },
  create(attrs) { return Db.create(attrs); },
  update(post, attrs) { return Db.update(post, attrs); },
  destroy(post) { return Db.destroy(post); },
};
```

Ten niedorzeczny kontroler składa się w 99% z wody. Wiecie, tej od lania wody. Możemy go przepisać:

```js
const BlogController = {
  index: Views.index,
  show: Views.show,
  create: Db.create,
  update: Db.update,
  destroy: Db.destroy,
};
```

…albo zupełnie zaorać bo i tak nie robi nic mądrego poza powiązaniem ze sobą View i Db.

## Dlaczego wolimy pierwszą klasę?

Okej, no to przejdźmy do powodów, dla których preferujemy pracować z funkcjami pierwszej klasy. Jak widać w `getServerStuff` i `BlogController`, bardzo łatwo jest dodawać warstwy abstrakcji, które niczego nie wnoszą i tylko zwiększają ilość niepotrzebnego kodu, który trzeba utrzymywać.

Dodatkowo, jeśli taka niepotrzebnie zapakowana funkcja w funkcji zostanie zmodyfikowana, to będziemy zmuszeni zmienić również tę zewnętrzną funkcję.

```js
httpGet('/post/2', json => renderPost(json));
```

Jeśli `httpGet` miałoby się zmienić i przekazywać opcjonalny błąd `err`, musielibyśmy wrócić do tego kodu i poprawić nasze spoiwo.

```js
// wracamy do każdego wywołania httpGet w całej aplikacji tylko po to, żeby dodać err
httpGet('/post/2', (json, err) => renderPost(json, err));
```

Gdybyśmy od początku zapisali to jako funkcję pierwszej klasy, znacznie mniej musiałoby się zmienić:

```js
// renderPost zostanie wywołane przez httpGet z iloma parametrami chce
httpGet('/post/2', renderPost);
```

Poza tworzeniem niepotrzebnych funkcji, mamy też problem z samymi nazwami argumentów. Widzisz, nazywanie rzeczy jest trudne. Istnieje ryzyko, że coś zaczniemy błędnie określać i jest ono tym wyższe, im starszy jest kod i im bardziej go zmieniamy.

Wiele nazw dla tego samego konceptu to częste źródło zagubienia w projektach. Poza tym jest też problem generycznego kodu. Przykładowo, te dwie funkcje poniżej robią dokładnie to samo, ale jednak pierwsza z nich wydaje się być zdecydowanie mniej ogólna i reużywalna:

```js
// związany z naszym blogiem
const validArticles = articles =>
  articles.filter(article => article !== null && article !== undefined),

// zdecydowanie bardziej przydatne w innych miejscach
const compact = xs => xs.filter(x => x !== null && x !== undefined);
```

Używając specyficznych nazw związaliśmy się z konkretnymi danymi (tutaj artykułami `articles`). To dzieje się w zasadzie codziennie i jest źródłem duplikacji i wymyślania koła na nowo.

Muszę jeszcze wspomnieć, że, podobnie jak w kodzie obiektowym, musimy być świadomi, że `this` przyjdzie niczym _rak nieborak i jak uszczypnie, to będzie znak_. Jeśli funkcja, której używamy korzysta z `this` i potraktujemy ją jak funkcję pierwszej klasy, to spotka nas gniew przeciekającej abstrakcji. Wrrr!

```js
const fs = require('fs');

// straszne
fs.readFile('freaky_friday.txt', Db.save);

// mniej straszne
fs.readFile('freaky_friday.txt', Db.save.bind(Db));
```

`Db.save` na zawsze związane z `Db` może teraz dowolnie używać swojego syfiastego kodu z `prototype`. Unikam `this` jak osób bez maseczek w czasie pandemii. Naprawdę obejdziemy się bez `this` pisząc funkcyjny kod. Jednakże, gdy używa się również innych bibliotek, to trzeba zgodzić się na to szaleństwo.

Niektórzy twierdzą, że `this` jest niezbędne do optymalizowania prędkości działania kodu. Jeśli jesteś z mikrooptymalizującego sortu, to proszę, zamknij tę książkę. Jeśli nie możesz już dostać zwrotu pieniędzy, to może chociaż uda Ci się ją wymienić na coś bardziej niskopoziomowego.

To powiedziawszy, jesteśmy teraz gotowi aby ruszać dalej!

[Rozdział 03: Czysta radość z czystych funkcji](ch03-pl.md)
