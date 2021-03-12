# Rozdział 04: Currying

## Can't Live If Livin' Is without You

Ojciec kiedyś wyjaśnił mi, że są pewne rzeczy, bez których można żyć tylko dopóki się ich nie ma. Mikrofalówka jest jedną z takich rzeczy. Smartfony, też. Starszyzna pamięta satysfakcjonujące życie w erze przed Internetem. Na mojej liście znajduje się currying.

Zamysł jest prosty: Wywołujesz funkcję z mniejszą liczbą argumentów, niż ta potrzebuje. Zwrócona zostaje funkcja, która przyjmie brakujące argumenty.

Możesz wywołać funkcję ze wszystkimi argumentami, albo karmić ją po kawałeczku.

```js
const add = x => y => x + y;
const increment = add(1);
const addTen = add(10);

increment(2); // 3
addTen(2); // 12
```

Stworzyliśmy funkcję `add`, która przyjmuje jeden argument i zwraca kolejną funkcję. Po wywołaniu jej, zwrócona funkcja „pamięta” pierwszy argument dzięki domknięciu. Wywołanie `add` z dwoma argumentami na raz trochę boli, jednak możemy wykorzystać specjalną funkcję o nazwie `curry`, która znacząco ułatwi nam deklarowanie takich i podobnych funkcji.

Stwórzmy kilka funkcji, na których zastosowano `curry` – od teraz jako `curry` będziemy przyjmować funkcję zdefiniowaną w [Dodatek A - Essential Function Support](./appendix_a-pl.md). 

```js
const match = curry((what, s) => s.match(what));
const replace = curry((what, replacement, s) => s.replace(what, replacement));
const filter = curry((f, xs) => xs.filter(f));
const map = curry((f, xs) => xs.map(f));
```

Zwróć uwagę na wzorzec, który tutaj zastosowałem, gdyż jest on istotny. Strategicznie umieściłem dane, na których chcemy operować (string, tablica) jako ostatni argument. Za chwilę stanie się jasne, dlaczego to ważne.

(Zapis `/r/g` to wyrażenie regularne, które pasuje do każdej litery "r". Więcej o [wyrażeniach regularnych](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Regular_Expressions).)

```js
match(/r/g, 'hello world'); // [ 'r' ]

const hasLetterR = match(/r/g); // x => x.match(/r/g)
hasLetterR('hello world'); // [ 'r' ]
hasLetterR('just j and s and t etc'); // null

filter(hasLetterR, ['rock and roll', 'smooth jazz']); // ['rock and roll']

const removeStringsWithoutRs = filter(hasLetterR); // xs => xs.filter(x => x.match(/r/g))
removeStringsWithoutRs(['rock and roll', 'smooth jazz', 'drum circle']); // ['rock and roll', 'drum circle']

const noVowels = replace(/[aeiou]/ig); // (r,x) => x.replace(/[aeiou]/ig, r)
const censored = noVowels('*'); // x => x.replace(/[aeiou]/ig, '*')
censored('Chocolate Rain'); // 'Ch*c*l*t* R**n'
```

Co się tu dzieje? Otóż pokazuję możliwość „załadowania” funkcji argumentem albo dwoma, aby otrzymać nową funkcję, która zapamięta te argumenty.

I encourage you to clone the Mostly Adequate repository (`git clone
https://github.com/MostlyAdequate/mostly-adequate-guide.git`), copy the code above and have a
go at it in the REPL. The curry function, as well as actually anything defined in the appendixes,
are available in the `support/index.js` module.

Alternatively, have a look at a published version on `npm`:

```
npm install @mostly-adequate/support
```

## More Than a Pun / Special Sauce

Currying is useful for many things. We can make new functions just by giving our base functions some arguments as seen in `hasLetterR`, `removeStringsWithoutRs`, and `censored`.

We also have the ability to transform any function that works on single elements into a function that works on arrays simply by wrapping it with `map`:

```js
const getChildren = x => x.childNodes;
const allTheChildren = map(getChildren);
```

Giving a function fewer arguments than it expects is typically called *partial application*. Partially applying a function can remove a lot of boiler plate code. Consider what the above `allTheChildren` function would be with the uncurried `map` from lodash (note the arguments are in a different order):

```js
const allTheChildren = elements => map(elements, getChildren);
```

We typically don't define functions that work on arrays, because we can just call `map(getChildren)` inline. Same with `sort`, `filter`, and other higher order functions (a *higher order function* is a function that takes or returns a function).

When we spoke about *pure functions*, we said they take 1 input to 1 output. Currying does exactly this: each single argument returns a new function expecting the remaining arguments. That, old sport, is 1 input to 1 output.

No matter if the output is another function - it qualifies as pure. We do allow more than one argument at a time, but this is seen as merely removing the extra `()`'s for convenience.


## In Summary

Currying is handy and I very much enjoy working with curried functions on a daily basis. It is a tool for the belt that makes functional programming less verbose and tedious.

We can make new, useful functions on the fly simply by passing in a few arguments and as a bonus, we've retained the mathematical function definition despite multiple arguments.

Let's acquire another essential tool called `compose`.

[Rozdział 05: Coding by Composing](ch05-pl.md)

## Exercises

#### Note about Exercises

Throughout the book, you might encounter an 'Exercises' section like this one. Exercises can be
done directly in-browser provided you're reading from [gitbook](https://mostly-adequate.gitbooks.io/mostly-adequate-guide) (recommended).

Note that, for all exercises of the book, you always have a handful of helper functions
available in the global scope. Hence, anything that is defined in [Dodatek A](./appendix_a-pl.md),
[Dodatek B](./appendix_b-pl.md) and [Dodatek C](./appendix_c-pl.md) is available for you! And, as
if it wasn't enough, some exercises will also define functions specific to the problem
they present; as a matter of fact, consider them available as well.

> Hint: you can submit your solution by doing `Ctrl + Enter` in the embedded editor!

#### Running Exercises on Your Machine (optional)

Should you prefer to do exercises directly in files using your own editor:

- clone the repository (`git clone git@github.com:MostlyAdequate/mostly-adequate-guide.git`)
- go in the *exercises* section (`cd mostly-adequate-guide/exercises`)
- install the necessary plumbing using [npm](https://docs.npmjs.com/downloading-and-installing-node-js-and-npm) (`npm install`)
- complete answers by modifying the files named *exercise\_\** in the corresponding chapter's folder 
- run the correction with npm (e.g. `npm run ch04`)

Unit tests will run against your answers and provide hints in case of mistake. By the by, the
answers to the exercises are available in files named *solution\_\**.

#### Let's Practice!

{% exercise %}  
Refactor to remove all arguments by partially applying the function.  
  
{% initial src="./exercises/ch04/exercise_a.js#L3;" %}  
```js  
const words = str => split(' ', str);  
```  
  
{% solution src="./exercises/ch04/solution_a.js" %}  
{% validation src="./exercises/ch04/validation_a.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}  


---


{% exercise %}  
Refactor to remove all arguments by partially applying the functions.  
  
{% initial src="./exercises/ch04/exercise_b.js#L3;" %}  
```js  
const filterQs = xs => filter(x => match(/q/i, x), xs);
```  
  
{% solution src="./exercises/ch04/solution_b.js" %}  
{% validation src="./exercises/ch04/validation_b.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}  


---


Considering the following function:

```js  
const keepHighest = (x, y) => (x >= y ? x : y);  
```  

{% exercise %}  
Refactor `max` to not reference any arguments using the helper function `keepHighest`.  
  
{% initial src="./exercises/ch04/exercise_c.js#L7;" %}  
```js  
const max = xs => reduce((acc, x) => (x >= acc ? x : acc), -Infinity, xs);  
```  
  
{% solution src="./exercises/ch04/solution_c.js" %}  
{% validation src="./exercises/ch04/validation_c.js" %}  
{% context src="./exercises/support.js" %}  
{% endexercise %}  
