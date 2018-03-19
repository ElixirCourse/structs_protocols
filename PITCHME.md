---
## Структури и протоколи
![Image-Absolute](assets/title.jpg)

---
## Структури
![Image-Absolute](assets/structures.jpg)

---
Една структура всъщност много приличат на `Map`, с няколко разлики:
- Ключовете им са атоми. Задължително атоми.|
- Ключовете им са предефинирани.|
---
- Структурите се дефинират в модул.
- Структурата взема името на модула в който е дефинирана.
- Може да даваме стойности по-подразбиране на някои ключове.

```elixir
defmodule Person do
  defstruct [:name, age: 20, :location, children: []]
end
```

---
- Създаването на инстанция на структура, също е подобно на създаването `Map`
- Разлика са:
  - трябва да посочим името на структурта

```elixir
iex> %Person{name: "Пешо", age: 35}
%Person{age: 35, children: [], location: nil, name: "Пешо"}
```
  - не можем да даваме ключове, които не са дефинирани в структурата
```elixir
iex> %Person{full_name: "Пешо Гошов"}
** (KeyError) key :full_name not found in: %Person{...}
    (stdlib) :maps.update(:full_name, "Пешо Гошов", %Person{...})
```
---
```elixir
iex> pesho = %Person{name: "Пешо", age: 35, children: "Нема"}
%Person{age: 35, children: "Нема", location: nil, name: "Пешо"}
iex> is_map(pesho)
true
iex> map_size(pesho)
5
```
@[1-2]
@[3-4]
@[5-6]
@[1-2,5-6]

---
```elixir
iex> inspect pesho, structs: false
"%{
  __struct__: Person,
  age: 35,
  chldren: [],
  location: \"Горен Чвор\", name: \"Пешо\"
}"
```

---
* Хубава новина е, че `Map` модулът работи със структури:

```elixir
iex> Map.put(pesho, :name, "Стойчо")
%Person{
  age: 35, chldren: [], location: "Горен Чвор", name: "Стойчо"
}
```

---
* Операторът за обновяване също работи:

```elixir
iex> %{pesho | name: "Стойчо"}
%Person{
  age: 35, chldren: [], location: "Горен Чвор", name: "Стойчо"
}
```

---
* Можем да съпоставяме структури с речници и други структури:

```elixir
iex> %{name: x} = pesho
%Person{
  age: 35, chldren: [], location: "Горен Чвор", name: "Пешо"
}
iex> x
"Пешо"
```

---
```elixir
iex> %Person{name: x} = pesho
%Person{
  age: 35, chldren: [], location: "Горен Чвор", name: "Пешо"
}
iex> x
"Пешо"

iex> %Person{} = %{}
** (MatchError) no match of right hand side value: %{}
```

---
### За какво и как да ползваме структури
![Image-Absolute](assets/types.jpg)

---
* Структурите се дефинират в модул с идеята че са нещо като тип дефиниран от нас.
* В модула би трябвало да напишем специални функции, които да работят с този тип.
* Така имаме на едно място дефиницията на типа и функциите за работа с него.

---
* Пример е `MapSet`:

```elixir
iex> inspect MapSet.new([2, 2, 3, 4]), structs: false
"%{__struct__: MapSet, map: %{2 => true, 3 => true, 4 => true}}"
```

---
```elixir
iex> MapSet.union(MapSet.new([1, 2, 3]), MapSet.new([2, 3, 4]))
#MapSet<[1, 2, 3, 4]>
```

---
### Структурите НЕ СА класове
![Image-Absolute](assets/i_see.jpg)

---
## Протоколи
![Image-Absolute](assets/protocols.jpg)

---
### Дефиниране на протокол
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
* Протокол се имплементира с макрото `defimpl`.
* Нека имплементираме `JSON` за атоми:

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
    cond do
      String.valid?(str) -> ~s("#{str}")
      true -> JSON.encode(bitstring_to_list(str))
    end
  end
end
```

---
```elixir
defimpl JSON, for: BitString do
  defp bitstring_to_list(binary) when is_binary(binary) do
    list_of_bytes(binary, [])
  end

  defp bitstring_to_list(bits), do: list_of_bits(bits, [])
end
```


---
```elixir
defimpl JSON, for: BitString do
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
