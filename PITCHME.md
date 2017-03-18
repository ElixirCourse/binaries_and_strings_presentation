#HSLIDE
## Низове и binaries
![Image-Absolute](assets/title.jpg)

#HSLIDE
## За какво ще си говорим днес?

1. Как да да работим с двоичните структури - binaries
2. Как са представени двоичните структури
3. Низове
4. Обхождане на низове

#HSLIDE
## Binaries
![Image-Absolute](assets/binary_code.jpg)

#HSLIDE
### Конструкция
```elixir
<<>>

<< 123, 23, 1 >>

iex> << 0b01111011, 0b00010111, 0b00000001 >> == << 123, 23, 1 >>
true

iex> << 280 >>
<<24>>
```

#HSLIDE
```elixir
iex> << 280::size(16) >> # Същото като << 280::16 >>
<< 1, 24 >>

iex> << 0b00000001, 0b00011000 >> == << 280::16 >>
true

<< 128::7 >>
1 # 10000001 - 1/0000001
```

#HSLIDE
* Интересно е съхранението на числа с плаваща запетая като `binaries`.
* Винаги се представят като `binary` от 8 байта:

```elixir
iex> << 5.5::float >>
<<64, 22, 0, 0, 0, 0, 0, 0>>
```

#HSLIDE
### Конкатенация
![Image-Absolute](assets/concatenate.jpg)

#HSLIDE
```elixir
iex> << 83, 79, 83 >> <> << 24, 4 >>
<<83, 79, 83, 24, 4>>

<< 83, 79, 83, << 24, 4 >> >>
```

#HSLIDE
### Размер

```elixir
iex> byte_size(<< 83, 79, 83 >>)
3

byte_size(<< 34::5, 23::2, 12::2 >>)
2

# Между другото:
<< 34::5, 23::2, 12::2 >> == << 22, 0::1 >>
true

```elixir
iex> bit_size(<< 34::5, 23::2, 12::2 >>)
9
```

#HSLIDE
### Проверки
1. `is_bitstring` - Винаги е истина за каквато и да е валидна поредица от данни между `<<` и `>>`. Няма значение дължината върната от `bit_size`.
2. `is_binary` - Истина е само ако `bit_size` връща число кратно на `8` - тоест структурата е поредица от байтове.

#HSLIDE
### Sub-binaries
![Image-Absolute](assets/sub_binary.jpg)

#HSLIDE
С функцията `binary_part`, можем да боравим с части от дадена `binary` структура:

```elixir
iex> binary_part(<< 83, 79, 83, 23, 21, 12 >>, 1, 3)
<<79, 83, 23>>
```

#HSLIDE
## Pattern matching

Като всичко друго в Elixir и с `binaries` можем да съпоставяме:

```elixir
<< x, y, x >> = << 83, 79, 83 >>

x
83

y
79
```

#HSLIDE
```elixir
iex> << x, y, z::binary >> = << 83, 79, 83, 43, 156 >>
<<83, 79, 83, 43, 156>>
iex> z
<< 83, 43, 156 >>

iex> << x, y, z >> = << 83, 79, 83, 43, 156 >>
** (MatchError)
```

#HSLIDE
```elixir
<<
  sign::size(1),
  exponent::size(11),
  mantissa::size(52)
>> = << 4815.162342::float >>
<<64, 178, 207, 41, 143, 62, 204, 196>>

iex> sign
0
```

#HSLIDE
```elixir
:math.pow(-1, sign) *
(1 + mantissa / :math.pow(2, 52)) *
:math.pow(2, exponent - 1023)

4815.162342
```

#HSLIDE
Модификатори:
* `float`
* `binary` или `bytes`
* `integer` модификатор, който се прилага по подразбиране
* `bits` или `bitstrig`
* `utf8`
* `utf16`
* `utf32`

#HSLIDE
```elixir
iex> << x::5, y::bits >> = << 225 >>
<<225>>
iex> x
28
iex> y
<<1::size(3)>>
```

#HSLIDE
* `size` задава дължина в битове, когато работим с `integer`.
* Aко работим с `binary` модификатор, `size` е в байтове.

#HSLIDE
```elixir
iex> << x::binary-size(4), _::binary >> =
  << 83, 222, 0, 345, 143, 87 >>
<<83, 222, 0, 89, 143, 87>>
iex> x # 4 байта
<<83, 222, 0, 89>>
```

#HSLIDE
## Binaries - имплементация
![Image-Absolute](assets/implementation.jpg)

#HSLIDE
* Всеки процес в Elixir има собствен heap.
* За всеки процес различни структури от данни и стойности са съхранени в този heap <!-- .element: class="fragment" -->
* Когато два процеса си комуникират, съобщенията се копират между heap-овете им. <!-- .element: class="fragment" -->

#HSLIDE
### Heap Binary
* Ако `binary`-то е `64` **байта** или по-малко, то е съхранявано в `heap`-a на процеса си.
* Такива `binary` структурки наричаме **heap binaries**.

#HSLIDE
### Refc Binary
* Когато структурката ни е по-голяма от `64` **байта**.
* Пази се в обща памет за всички process-и на даден `node`.
* В `porcess heap`-a се пази малко обектче - **ProcBin**.
* `binary` структурка може да бъде сочена от множество `ProcBin` указатели от множество процеси.

#HSLIDE
* Пази се _reference counter_ за всеки от тези указатели.
* Когато той стане `0`, Garbage Collector-ът ще може да изчисти `binary`-то от общия `heap`.
* Такива `binary` структури наричаме **refc binaries**.

#HSLIDE
### Sub binaries
* Указател към съществуващо `binary` пазещ по-малка от неговата дължина.
* Няма копиране.
* Броячът на референции се увеличава.
* Функцията `binary_part` да речем създава такива.

#HSLIDE
### Matching context
* Подобен на `sub binary`, но оптимизиран за `binary pattern matching`.
* Държи указател към двоичните данни в паметта.
* Когато нещо е `match`-нато, указателят се придвижва напред.
* Компилаторът избягва създаването на `sub binary` за всяка `match`-ната променлива.
* Ако е възможно и преизползва един и същ `match context`.

#HSLIDE
```elixir
x = << 83, 222, 0, 89 >>
y = << x, 225, 21 >>
z = << y, 125, 156 >>
a = << y, 15, 16, 19 >>
```

#HSLIDE
## Низове
![Image-Absolute](assets/strings.jpg)

#HSLIDE
Низовете в Elixir използват `unicode` - предтавляват `binary` структури -
поредица от `unicode codepoint`-и в `utf8` енкодинг.

#HSLIDE
* При `UTF-8` даден `codepoint` може да се съхранява в от един до четири байта.
* Първите `128` кодпойнта съвпадат с `ASCII` `codepoint`-ите.
* Te се пазят в един байт, който започва с `0`:

```
0XXXXXXX
```

#HSLIDE
```elixir
iex> << 0b00110000 >>
"0"
iex> << 0b00110001 >>
"1"
iex> << 0b00110010 >>
"2"
iex> << 0b00110101 >>
"5"
iex> << 0b00111001 >>
"9"
```

#HSLIDE
* След като `128`-възможности за `1` байт се изчерпат, започват да се ползват по `2` байта:
* В този шаблон за `2`-байтови `codepoint`-и, виждаме че първият байт винаги започва с `110`.
* Двете единици значат - `2` байта, байт започващ с `10`, означава че е част от поредица от байтове.

```
110XXXXX 10XXXXXX
```

#HSLIDE
```elixir
iex> ?Ъ
1066
```

```
Префиксваме с 110 и допълваме до байт с битовете от числото
(110)10000

Префиксваме с 10 и допълваме до байт с битовете, които останаха
(10)101010
```

#HSLIDE
```elixir
iex> Integer.to_string(1066, 2)
"10000101010"
```

```elixir
iex> << 0b11010000, 0b10101010 >>
"Ъ"
```

#HSLIDE
Шаблонът за `3`-байтови `codepoint`-и е:

```
1110XXXX 10XXXXXX 10XXXXXX
```

#HSLIDE
А шаблонът за `4`-байтовите:

```
11110XXX 10XXXXXX 10XXXXXX 10XXXXXX
```

#HSLIDE
Можем да заключим че има три типа байтове:
1. Единичен - започва с `0` и представлява първите `128` ASCII-compatible `codepoint`-a.
2. Байт-продължение - започва с `10`, следва други байтове в `codepoint` от повече от `1` байт.
3. Байт-начало - започва с `110`, `1110`, `11110`, първи байт от серия байтове представляващи `codepoint`.

#HSLIDE
### Графеми и codepoint-и
![Image-Absolute](assets/graphemes.jpg)

#HSLIDE
* Символите или графемите не винаги са точно един `codepoint`.
* Има графеми, които могат да се представят и като един `codepoint` и като няколко.

#HSLIDE
```elixir
iex> name = "Николай"
"Николай"
iex> String.codepoints(name)
["Н", "и", "к", "о", "л", "а", "й"]
iex> String.graphemes(name)
["Н", "и", "к", "о", "л", "а", "й"]
iex> String.graphemes(name) == String.codepoints(name)
true
```

#HSLIDE
```elixir
iex> name = "Николаи\u0306"
"Николай"
iex> String.codepoints(name)
["Н", "и", "к", "о", "л", "а", "и", ̆"<не може да се представи>"]
iex> String.graphemes(name)
["Н", "и", "к", "о", "л", "а", "й"]
```

#HSLIDE
* Функции като `String.reverse/1`, работят правилно, защото ползват `String.graphemes/1`.

```elixir
iex(288)> String.reverse(name2)
"йалокиН"
# Аналогично:
iex(290)> String.graphemes(name2) |> Enum.reverse |> Enum.join("")
"йалокиН"
```

#HSLIDE
### Дължина на низ
```elixir
iex> Kernel.bit_size "Николаи\u0306"
128
iex> Kernel.byte_size "Николаи\u0306"
16
iex> String.length "Николаи\u0306"
7
```

#HSLIDE
### Под-низове
```elixir
iex> String.at("Искам бира!", 6)
"б"
iex> String.at("Искам бира!", -1)
"!"
iex> String.at("Искам бира!", 12)
nil
```

#HSLIDE
```elixir
iex> << _::binary-size(11), x::utf8, _::binary >> = "Искам бира!"
"Искам бира!"
iex> x
1073
iex> << x::utf8 >>
"б"
```

#HSLIDE
```elixir
iex> << "Искам ", x::utf8, _::binary >> = "Искам бира!"
"Искам бира!"
```

#HSLIDE
```elixir
iex> String.next_codepoint("Николаи\u0306")
{"Н", "иколай"}
iex> String.next_codepoint("и\u0306")
{"и", "<нещо което скапва hilighting-a на кода>"}
iex> String.next_grapheme("и\u0306")
{"й", ""}
iex> String.next_grapheme_size("и\u0306")
{4, ""}
iex> String.next_grapheme_size("\u0306")
{2, ""}
```

#HSLIDE
```elixir
poem = """
  Ще строим завод,
  огромен завод,
  със яки
        бетонни стени!
  Мъже и жени,
  народ,
  ще строим завод
  за живота!
"""

iex> large_factory = String.slice(poem, 18..30)
"огромен завод"
iex> String.slice(poem, 18, 13)
"огромен завод"
```

#HSLIDE
## Обхождане на низове
![Image-Absolute](assets/iterate_path.jpg)

#HSLIDE
```elixir
defmodule ACounter do
  def count_it_with_slice(str) do
    count_with_slice(str, 0)
  end

  defp count_with_slice("", n), do: n
  defp count_with_slice(str, n) do
    the_next_n = next_n(String.starts_with?(str, "a"), n)
    count_with_slice(String.slice(str, 1..-1), the_next_n)
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

#HSLIDE
* Това е около `3.18` секунди за низ с дължина над 2_000_000

```elixir
iex> :timer.tc(ACounter, :count_it_with_slice, [str])
{3176592, 589251}
```

#HSLIDE
```elixir
defmodule ACounter do
  def count_it_with_next_grapheme(str) do
    count_with_next_grapheme(str, 0)
  end

  defp count_with_next_grapheme("", n), do: n
  defp count_with_next_grapheme(str, n) do
    {next_grapheme, rest} = String.next_grapheme(str)
    count_with_next_grapheme(
      rest,
      next_n(next_grapheme == "a", n)
    )
  end

  defp next_n(true, n), do: n + 1
  defp next_n(false, n), do: n
end
```

#HSLIDE
* Добре същият отговор, но сега за `0.8` секунди

```elixir
iex> :timer.tc(ACounter, :count_it_with_next_grapheme, [str])
{792885, 589251}
```

#HSLIDE
```elixir
defmodule ACounter do
  def count_it_with_match(str) do
    count_with_match(str, 0)
  end

  defp count_with_match("", n), do: n

  defp count_with_match(<< c::utf8, rest::binary >>, n)
  when c == ?a do
    count_with_match(rest, n + 1)
  end

  defp count_with_match(<< _::utf8, rest::binary >>, n) do
    count_with_match(rest, n)
  end
end
```

#HSLIDE
## Край
![Image-Absolute](assets/end.jpg)
