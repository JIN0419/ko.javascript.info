# Оператор typeof и утиная типизация

В этой главе мы рассмотрим, как создавать *полиморфные* функции, то есть такие, которые по-разному обрабатывают аргументы, в зависимости от их типа. Например, функция вывода может по-разному форматировать числа и даты.

Для реализации такой возможности нужен способ определить тип переменной. 

[cut]
Как мы знаем, существует несколько *примитивных типов*:
<dl>
<dt>`null`</dt>
<dd>Специальный тип, содержит только значение `null`.</dd>
<dt>`undefined`</dt>
<dd>Специальный тип, содержит только значение `undefined`.</dd>
<dt>`number`</dt>
<dd>Числа: `0`, `3.14`, а также значения `NaN` и `Infinity`</dd>
<dt>`boolean`</dt>
<dd>`true`, `false`.</dd>
<dt>`string`</dt>
<dd>Строки, такие как `"Мяу"` или пустая строка `""`.</dd>
</dl>

Все остальные значения, включая даты и массивы, являются объектами.

## Оператор typeof [#type-typeof]

Оператор `typeof` возвращает тип аргумента. У него есть два синтаксиса: со скобками и без:
<ol>
<li>Синтаксис оператора: `typeof x`.</li>
<li>Синтаксис функции: `typeof(x)`.</li>
</ol>

Работают они одинаково, но первый синтаксис короче.

**Результатом `typeof` является строка, содержащая тип:**

```js
typeof undefined // "undefined" 

typeof 0 // "number" 

typeof true // "boolean" 

typeof "foo" // "string" 

typeof {} // "object" 

*!*
typeof null // "object" 
*/!*

function f() { /* ... */ }
typeof f // "function" 
*/!*
```

Последние две строки помечены, потому что `typeof` ведет себя в них по-особому.

<ol>
<li>Результат `typeof null == "object"` -- это официально признанная ошибка в языке, которая сохраняется для совместимости.

На самом деле `null` -- это не объект, а примитив. Это сразу видно, если попытаться присвоить ему свойство:

```js
//+ run
var x = null;
x.prop = 1; // ошибка, т.к. нельзя присвоить свойство примитиву
```

</li>
<li>Для функции `f` значением `typeof f` является `"function"`. На самом деле функция не является отдельным базовым типом в JavaScript, все функции являются объектами, но такое выделение функций на практике удобно, так как позволяет легко определить функцию.</li>
</ol>

**Оператор `typeof` надежно работает с примитивными типами, кроме `null`, а также с функциями. Но обычные объекты, массивы и даты для `typeof` все на одно лицо, они имеют тип `'object'`:**

```js
//+ run
alert( typeof {} ); // 'object'
alert( typeof [] ); // 'object'
alert( typeof new Date ); // 'object'
```

Поэтому различить их при помощи `typeof` нельзя.

## Утиная типизация

Основная проблема `typeof` -- неумение различать объекты, кроме функций. Но есть и другой способ проверки типа.

Так называемая "утиная типизация" основана на одной известной пословице: *"If it looks like a duck, swims like a duck and quacks like a duck, then it probably is a duck (who cares what it really is)"*.

В переводе: *"Если это выглядит как утка, плавает как утка и крякает как утка, то, вероятно, это утка (какая разница, что это на самом деле)"*. 

Смысл утиной типизации -- в проверке необходимых методов и свойств.

Например, у нас функция работает с массивами. Мы можем проверить, что объект -- массив, уточнив наличие метода `splice`, который, как известно, есть у всех массивов:

```js
//+ run
var something = [1, 2, 3];

if (something.splice) {
  alert( 'Это утка! То есть, массив!' );
}
```

Обратите внимание -- в `if` мы не вызываем метод `something.splice()`, а пробуем получить само свойство `something.splice`. Для массивов оно всегда есть и является функцией, т.е. даст в логическом контексте `true`.

Проверить на дату можно, определив наличие метода `getTime`:

```js
//+ run
var x = new Date();

if (x.getTime) {
  alert( 'Дата!' );
}
```

С виду такая проверка хрупка, ее можно "сломать", передав похожий объект с тем же методом. 

Но как раз в этом и есть смысл утиной типизации: если объект похож на массив, у него есть методы массива, то будем работать с ним как с массивом (какая разница, что это на самом деле). 

## Полиморфизм

Пример полимофрной функции -- `sayHi(who)`, которая будет говорить "Привет" своему аргументу, причём если передан массив -- то "Привет" каждому:

```js
//+ run
function sayHi(who) {

  if (who.forEach) { // проверка на массив (или что-то похожее)
    who.forEach(sayHi);
  } else {
    alert( 'Привет, ' + who );
  }
}

// Вызов с примитивным аргументом
sayHi("Вася"); // Привет, Вася

// Вызов с массивом
sayHi(["Саша", "Петя"]); // Привет, Саша... Петя

// Вызов с вложенными массивами
sayHi(["Саша", "Петя", ["Маша", "Юля"]]); // Привет Саша..Петя..Маша..Юля
```

Здесь вместо `splice` проверяется наличие `forEach`. Так надёжнее, поскольку именно его мы собираемся использовать.

Обратите внимание, получилась даже поддержка вложенных массивов. Да здравствует рекурсия! 

## Итого

Для написания полиморфных (это удобно!) функций нам нужна проверка типов.

Для примитивов с ней отлично справляется оператор `typeof`. 

У него две особенности:
<ol>
<li>Он считает `null` объектом, это внутренняя ошибка в языке.</li>
<li>Для функций он возвращает `function`, по стандарту функция не считается базовым типом, но на практике это удобно и полезно.</li>
</ol>

Там, где нужно различать объекты, обычно используется утиная типизация, то есть мы смотрим, есть ли в объекте нужный метод, желательно -- тот, который мы собираемся использовать.
