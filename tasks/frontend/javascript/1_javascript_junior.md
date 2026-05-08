# JavaScript — Junior Tasks

## Задачи

- [Перевернуть каждое второе слово в строке](#перевернуть-каждое-второе-слово-в-строке)
- [Функция capitalize](#функция-capitalize)
- [Проверка подпоследовательности (needleInHaystack)](#проверка-подпоследовательности-needleinhaystack)
- [Поиск подстроки (fuzzySearch)](#поиск-подстроки-fuzzysearch)

---

## Перевернуть каждое второе слово в строке

Напишите функцию `evenBack(str)`, которая принимает строку, разбивает её на слова по пробелам и переворачивает каждое **чётное по порядку** слово (2-е, 4-е, 6-е, считая с 1), остальные слова оставляет без изменений.

```
evenBack("hello world foo bar") // "hello dlrow foo rab"
evenBack("one two three four five") // "one owt three ruof five"
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
function evenBack(str) {
  return str
    .split(' ')
    .map((word, i) =>
      (i + 1) % 2 === 0
        ? word.split('').reverse().join('')
        : word
    )
    .join(' ');
}

console.log(evenBack("hello world foo bar")); // "hello dlrow foo rab"
```

</details>

---

## Функция capitalize

Напишите функцию `capitalize(str)`, которая принимает строку и возвращает её с первой заглавной буквой, остальные буквы переводятся в нижний регистр.

```
capitalize("hELLO wORLD") // "Hello world"
capitalize("javaScript")   // "Javascript"
```

**Связанные вопросы:**

- [Что такое высшие функции (higher-order functions)?](../../../interviews/frontend/javascript/1_javascript_junior.md#что-такое-высшие-функции-higher-order-functions)

<details>
<summary>Решение</summary>

```javascript
function capitalize(str) {
  if (!str) return str;
  return str[0].toUpperCase() + str.slice(1).toLowerCase();
}

// Через деструктуризацию
function capitalize2(str) {
  const [first, ...rest] = str;
  return first.toUpperCase() + rest.join('').toLowerCase();
}

console.log(capitalize("hELLO wORLD")); // "Hello world"
```

</details>

---

## Проверка подпоследовательности (needleInHaystack)

Напишите функцию `needleInHaystack(needle, haystack)`, которая проверяет, является ли строка `needle` **подпоследовательностью** строки `haystack`. Символы needle должны встречаться в haystack в том же порядке, но необязательно подряд.

```
needleInHaystack("ace", "abcde") // true
needleInHaystack("aec", "abcde") // false
needleInHaystack("",    "abc")   // true
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
function needleInHaystack(needle, haystack) {
  let i = 0;
  for (const char of haystack) {
    if (char === needle[i]) i++;
    if (i === needle.length) return true;
  }
  return needle.length === 0;
}

console.log(needleInHaystack("ace", "abcde")); // true
console.log(needleInHaystack("aec", "abcde")); // false
```

</details>

---

## Поиск подстроки (fuzzySearch)

Напишите функцию `fuzzySearch(query, text)`, которая проверяет, содержится ли строка `query` как подстрока в `text` (без учёта регистра). В отличие от `needleInHaystack`, символы должны идти **подряд**.

```
fuzzySearch("foo", "barfoobar") // true
fuzzySearch("Foo", "barFOObar") // true (без учёта регистра)
fuzzySearch("baz", "barfoobar") // false
```

**Связанные вопросы:**

<!-- Связанных вопросов нет -->

<details>
<summary>Решение</summary>

```javascript
function fuzzySearch(query, text) {
  return text.toLowerCase().includes(query.toLowerCase());
}

// Альтернатива через indexOf
function fuzzySearchAlt(query, text) {
  return text.toLowerCase().indexOf(query.toLowerCase()) !== -1;
}

console.log(fuzzySearch("Foo", "barFOObar")); // true
console.log(fuzzySearch("baz", "barfoobar")); // false
```

</details>
