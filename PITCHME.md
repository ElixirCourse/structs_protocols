---
## Структури и протоколи
![Image-Absolute](assets/title.jpg)

---
## Структури
![Image-Absolute](assets/structures.jpg)

---
Една структура всъщност много прилича на `Map`, с няколко разлики:
- Ключовете ѝ са атоми. |
  - Задължително атоми! |
- Ключовете ѝ са предефинирани.|
---
Дефиниране на структура:
- Структурите се дефинират в модул.
- Структурата взема името на модула в който е дефинирана. |
- Може да даваме стойности по-подразбиране на някoe поле на структурата. |
- Можем да направим някое поле задължително с модул атрибута @enforce_keys. |
---
#### Пример
```elixir
defmodule Person do
  @enforce_keys [:name]
  defstruct [:name, :location, children: []]
end
```
---

#### Създаването на инстанция на структура, също е подобно на създаването на `Map`

---
Разликите са:
- Траява да дадем името на структурата

```elixir
iex> %Person{name: "Пешо", location: "Некаде", children: "Нема"}
%Person{children: "Нема", location: "Некаде", name: "Пешо"}
```
---
Разликите са:
- Ако пропуснем да дадем стойност на даден дефиниран ключ той ще получи стойността си по подразбиране
- Ако няма дефинирана стойност по подразбиране, то стойността му ще е `nil`
```elixir
iex> %Person{name: "Пешо"}
%Person{children: [], location: nil, name: "Пешо"}
```
---
Разликите са:
- Ако не дадем стойност на ключ, който е обявен за задължителен получаваме грешка
```elixir
iex> %Person{children: "Пет или шес'"}
** (ArgumentError) the following keys must also be given
                   when building struct Person: [:name]
```
---
Разликите са:
- Не можем да даваме ключове, които не са дефинирани в структурата
```elixir
iex> %Person{name: "Пешо", drinks: "ВодKа"}
** (KeyError) key :drinks not found in: %Person{...}
    (stdlib) :maps.update(:drinks, "ВодKа", %Person{...})
```
@[3]

---
Нека пробваме няколко неща

```elixir
iex> pesho = %Person{name: "Пешо", children: "Нема",
...>  lotaction: "НикАде"}
%Person{children: "Нема", location: "НикАде", name: "Пешо"}
iex> is_map(pesho)
true
iex> map_size(pesho)
4
```
@[1-3]
@[4-5]
@[6-7]
@[1-3,6-7]

---
Структурите всъщност са `Map`-ове със специалния ключ `__struct__`
```elixir
iex> inspect pesho, structs: false
"%{
  __struct__: Person,
  children: \"Нема\",
  location: \"НикАде\",
  name: \"Пешо\"
}"
```
---
Хубава новина е, че `Map` модулът работи със структури.

```elixir
iex> Map.put(pesho, :name, "Стойчо")
%Person{
  children: Нема, location: "НикАде", name: "Стойчо"
}
```
---
Операторът за обновяване също работи.

```elixir
iex> %{pesho | children: "Жената ги брои"}
%Person{
  children: "Жената ги брои", location: "НикАде", name: "Пешо"
}
```
---

### Можем да съпоставяме структури с речници и други структури

---

#### Но първо, бързо припомняне.

```elixir
iex> %{age: x} = %{name: "Гошо", age: 25}
iex> x
???
```
---

#### Но първо, бързо припомняне.

```elixir
iex> %{age: x} = %{name: "Гошо", age: 25}
iex> x
25
```
---
#### Още едно припомняне

```elixir
iex> pesho = %Person{name: "Пешо", children: "Нема",
...> lotaction: "НикАде"}
%Person{children: "Нема", location: "НикАде", name: "Пешо"}
```
---
#### Maчинг на Map и структура

```elixir
iex> %{children: x} = pesho
iex> x
???
```
---
#### Maчинг на Map и структура

```elixir
iex> %{children: x} = pesho
iex> x
"Нема"
```
---
#### Съпоставяне на 2 структури

```elixir
iex> %Person{location: x} = pesho
iex> x
???
```
---

#### Съпоставяне на 2 структури

```elixir
iex> %Person{location: x} = pesho
iex> x
"НикАде"
```
---

#### Съпоставяне на структура и Map

```elixir
iex> %Person{name: x} = %{name: "Гошо"}
iex> x
???
```

---

#### Съпоставяне на структура и Map

```elixir
iex> %Person{name: x} = %{name: "Гошо"}
** (MatchError) no match of right hand side value: %{name: "Гошо"}
iex> x
** (CompileError) iex:1: undefined function x/0
```

- Това е дборе защото, така можем да проверяваме дали нещо е инстанция на даден а структура|
---

Достъп до елементите на структура:
- Mожем да достъпваме елементите на структура с `"."`
```elixir
iex> pesho.name
"Пешо"
```
- Не mожем да достъпваме елементите на структура с `"[]"`
```elixir
iex()> pesho[:name]
** (UndefinedFunctionError) function Person.fetch/2 is undefined
   (Person does not implement the Access behaviour)
```
---

### За какво и как да ползваме структури
![Image-Absolute](assets/types.jpg)

---
- Структурата се дефинира в модул с идеята, че е нещо като тип дефиниран от нас.
- В модула обикновено слагаме функции, които да работят с този тип:|
  - такива които "конструира" инстанция на структрата по дадени аргументи |
  - или фунциим, които приемат инстанция на структурата, като първи акгумент |
- Така имаме на едно място дефиницията на типа и функциите за работа с него.|

---
### Структурите НЕ СА класове
![Image-Absolute](assets/i_see.jpg)

---
- Примери за структури от стандартната бибиотека са:
  - MapSet
  - Time
```elixir
iex> inspect MapSet.new([2, 2, 3]), structs: false
"%{__struct__: MapSet, map: %{2 => true, 3 => true}}"

iex> {:ok, time} = Time.new(12, 34, 56)
iex> inspect time, structs: false
"%{__struct__: Time, calendar: Calendar.ISO, hour: 12,
   microsecond: {0, 0}, minute: 34, second: 56}"
```
---
- Примери за структури от стандартната бибиотека са:
  - Regex
  - Range
```elixir
iex> inspect ~r("Red"), structs: false
"%{__struct__: Regex, opts: \"\",
   re_pattern: {:re_pattern, 0, 0, 0, <<...>>},
   re_version: \"8.41 2017-07-05\", source: \"\\\"Red\\\"\"}"

iex> inspect 3..93, structs: false
"%{__struct__: Range, first: 3, last: 93}"
```

---
## Протоколи
![Image-Absolute](assets/protocols.jpg)

---
Дефиниране на протокол:

- Прави се с макрото `defprotocol`, като указваме кои функции деинира протокола

```elixir
defprotocol JSON do
  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```
---
```elixir
iex> JSON.encode(nil)
** (Protocol.UndefinedError) protocol JSON not implemented for nil
json_protocol.ex:1: JSON.impl_for!/1
json_protocol.ex:3: JSON.encode/1
```
---
- Протокол се имплементира с макрото `defimpl`.
- Нека имплементираме `JSON` за атоми:

```elixir
defimpl JSON, for: Atom do
  def encode(true), do: "true"
  def encode(false), do: "false"
  def encode(nil), do: "null"

  def encode(atom) do
    JSON.encode(Atom.to_string(atom))
  end
end
```

---
```elixir
iex> JSON.encode(true)
"true"
iex> JSON.encode(false)
"false"
iex> JSON.encode(nil)
"null"
iex> JSON.encode(:name)
** (Protocol.UndefinedError)
```

---
```elixir
defimpl JSON, for: BitString do
  def encode(<< >>), do: ~s("")
  def encode(str) do
    if String.valid?(str) do
      ~s("#{str}") # <=> "\"#{str}\""
    else
      JSON.encode(bitstring_to_list(str))
    end
  end

  defp bitstring_to_list(binary) when is_binary(binary) do
    list_of_bytes(binary, [])
  end

  defp bitstring_to_list(bits), do: list_of_bits(bits, [])

  defp list_of_bytes(<<>>, list), do: list |> Enum.reverse
  defp list_of_bytes(<< x, rest::binary >>, list) do
    list_of_bytes(rest, [x | list])
  end

  defp list_of_bits(<<>>, list), do: list |> Enum.reverse
  defp list_of_bits(<< x::1, rest::bits >>, list) do
    list_of_bits(rest, [x | list])
  end
end
```
@[2]
@[3-9]
@[4-5]
@[6-8]
@[11-13,15]
@[11-13]
@[17-20]
@[15]
@[22-25]

---
```elixir
iex> JSON.encode(:name)
"\"name\""
iex> JSON.encode("")
"\"\""
iex> JSON.encode("some")
"\"some\""
iex> JSON.encode(<< 200, 201 >>)
** (Protocol.UndefinedError)
```

---
```elixir
defimpl JSON, for: List do
  def encode(list) do
    "[#{list |> Enum.map(&JSON.encode/1) |> Enum.join(", ")}]"
  end
end
```

---
```elixir
iex> JSON.encode([nil, true, false])
"[null, true, false]"
iex> JSON.encode(<< 200, 201 >>)
** (Protocol.UndefinedError)
```

---
```elixir
defimpl JSON, for: Integer do
  def encode(n), do: n
end
```

---
```elixir
iex> JSON.encode(<< 200, 201 >>)
"[200, 201]"
```

---
```elixir
defimpl JSON, for: Map do
  def encode(map) do
    "{#{map |> Enum.map(&encode_pair/1) |> Enum.join(", ")}}"
  end

  defp encode_pair(pair) do
    {key, value} = pair

    "#{JSON.encode(to_string(key))}: #{JSON.encode(value)}"
  end
end
```

---
```elixir
iex> data = %{
  name: "Pesho",
  age: 43,
  likes: [:drinking, "eating shopska salad", "да гледа мачове"]
}
iex> IO.puts JSON.encode(data)
{...}
```

---
```json
{
  "age": 43,
  "likes": [
    "drinking",
    "eating shopska salad",
    "да гледа мачове"
  ],
  "name": "Pesho"
}
```

---
### Структури и протоколи
![Image-Absolute](assets/adapters.jpeg)

---
```elixir
defmodule Man do
  defstruct [:name, :age, :likes]
end

kosta = %Man{
  name: "Коста",
  age: 54,
  likes: ["Турбо фолк", "Телевизия", "да гледа мачове"]
}
JSON.encode(kosta)
** (Protocol.UndefinedError)
```

---
Вградените типове за които можем да имплементираме протокол са:
* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`

---
* `List`
* `Map`
* `PID`
* `Port`
* `Reference`
* `Tuple`

---
* Има и начин да имплементираме протокол за всички случаи за които не е имплементиран,
използвайки `Any`:

```elixir
defimpl JSON, for: Any do
  def encode(_), do: "null"
end
```

---
```elixir
JSON.encode(kosta)
** (Protocol.UndefinedError)
```

---
```elixir
defmodule Man do
  @derive JSON
  defstruct [:name, :age, :likes]
end

nikodim = %Man{
  name: "Никодим", age: 15, likes: ["Порно", "GTA V"]
}
JSON.encode(nikodim)
"null"
```

---
```elixir
defprotocol JSON do
  @fallback_to_any true

  @doc "Converts the given data to its JSON representation"
  def encode(data)
end
```

```elixir
iex> JSON.encode({:ok, :KO})
"null"
```

---
## Протоколи идващи с езика
```elixir
iex> path = :code.lib_dir(:elixir, :ebin)
iex> Protocol.extract_protocols([path])
[Collectable, Inspect, String.Chars, List.Chars, Enumerable]
```

---
* `Collectable` - това е протоколът, използван от `Enum.into`.
* `Inspect` - използва се за _pretty printing_.
* `String.Chars` - `Kernel.to_string/1` го използва.
* `List.Chars` - `Kernel.to_charlist/1` го използва.
* `Enumerable` - `Enum` методите очакват имплементации.

---
```elixir
iex>Protocol.extract_impls(Enumerable, [path])
[
  Stream, List, Function, Map,
  IO.Stream, Range, MapSet, HashDict, HashSet,
  GenEvent.Stream, File.Stream
]
```

---
```elixir
defimpl Enumerable, for: BitString do
  def count(str), do: {:ok, String.length(str)}
  def member?(str, char), do: {:ok, String.contains?(str, char)}

  def reduce(_, {:halt, acc}, _fun), do: {:halted, acc}
  def reduce(str, {:suspend, acc}, fun) do
    {:suspended, acc, &reduce(str, &1, fun)}
  end

  def reduce("", {:cont, acc}, _fun), do: {:done, acc}
  def reduce(str, {:cont, acc}, fun) do
    {next, rest} = String.next_grapheme(str)
    reduce(rest, fun.(next, acc), fun)
  end
end
```

---
```elixir
"Далия"
|> Enum.filter(fn
     c when c in ~w(а ъ о у е и) -> false
     _ -> true
   end)
|> Enum.join("")
"Для"
```

---
### Консолидация
![Image-Absolute](assets/consolidation.png)

---
## Край
