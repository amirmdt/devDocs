# What is YAML?

*YAML is a computer data serialization language.*

*A YAML document represents a computer program's native data structure in a human readable text form. A node in a YAML document can have three basic data types:*

* ***Scalar*** *Atomic data types like strings, numbers, booleans and null*
* ***Sequence*** *A list of nodes*
* ***Mapping*** *A map of nodes to nodes. Also known as Hashes, Hash Maps, Dictionaries or Objects.
Unlike in many programming languages, a key can be more than just a string.
It can be a sequence or mapping itself.*

*On top of that, YAML allows to serialize all other data types and classes:*

* ***Alias and Anchor*** *For serializing References / Pointers, including circular references.*
* ***Tag*** *With Tags it's possible to define custom types/classes.
For example, in many languages a Regular Expression is a builtin data type or object.
Some languages have only arrays, which are represented by the basic sequence type.
But some have tuples, which needs a custom tag.*

## Tutorial

*The following examples will introduce you with YAML syntax elements step by step.*

### Invoice

*Let's write an invoice. It has a number, a name and an address, order items and more.*

#### Mapping

*The most common top level data type are mappings. A mapping maps values to keys.
Keys and values are separated with a colon and a space` :` .
Each Key/Value pair is on its own line.*

```
invoice number: 314159
name: Santa Claus
address: North Pole
```

*An alternative way to write it:*
```
---
invoice number: 314159
name: Santa Claus
address: North Pole
```

*The `---` is explicity starting a `Document.`*\
*It marks the following content as YAML, but it is optional.*\
*It has some use cases, and it is needed when you have multiple Documents in one file.*

#### Nested Mappings

*Now we replace the `address` string with another mapping. In that case the colon is followed by a linebreak. Mapping values that are not scalars must always start on a new line.*\
*Nested items must always be indented more then the parent node, with at least one space. The typical indentation is two spaces.*\
***Tabs are forbidden as indentation.***
```
invoice number: 314159
name: Santa Claus

address:
  street: Santa Claus Lane
  zip: 12345
  city: North Pole
```

*Don't forget the indentation. If you write it like this:*
```
invoice number: 314159
name: Santa Claus

address:
street: Santa Claus Lane
zip: 12345
city: North Pole
```
*... then it will actually mean this:*
```
invoice number: 314159
name: Santa Claus

address: null
street: Santa Claus Lane
zip: 12345
city: North Pole
```

#### Sequence

*A sequence is a list (or array) of scalars (or other sequences or mappings). A sequence item starts with a hyphen and a space `- `. Here is the list of YAML inventors:*
```
- Oren Ben-Kiki
- Clark Evans
- Ingy döt Net
```
*Now back to our invoice.*\
*We map a list of scalars to the key `order items`.*

*The sequence must start on the next line:*
```
invoice number: 314159
name: Santa Claus
address:
  street: Santa Claus Lane
  zip: 12345
  city: North Pole

order items:
  - Sled
  - Wrapping Paper
```
*Because the `- ` counts as indentation, you can also write it like this:*
```
invoice number: 314159
name: Santa Claus
address:
  street: Santa Claus Lane
  zip: 12345
  city: North Pole

order items:
- Sled
- Wrapping Paper
```

#### Nested Sequences

*You can also nest sequences. The typical example is a List of Dice Rolls.*

*The nested sequence items can follow directly on the same line:*
```
---
- - 2
  - 3
- - 3
  - 6
```
*YAML allows to write that in a more compact way, the `Flow Style` :*
```
---
- [ 2, 3 ]
- [ 3, 6 ]
```

***more example :***
```

```

### Aliases / Anchors

*Let's add a billing address to the invoice.*

*In our case it is the same as the shipping address. We rename `address` to `shipping address` and add `billing address` :*
```
invoice number: 314159
name: Santa Claus

shipping address:
  street: Santa Claus Lane
  zip: 12345
  city: North Pole
billing address:
  street: Santa Claus Lane
  zip: 12345
  city: North Pole

order items:
- Sled
- Wrapping Paper
```

*Now that's a bit wasted space. If it's the same address, you don't need to repeat it. Use an `Alias` .*

*In the native data structure of a programming language, this would be a reference, pointer, or alias.*

*Before an `Alias` can be used, it has to be created with an `Anchor` :*

```
invoice number: 314159
name: Santa Claus

shipping address: &address     #   Anchor
  street: Santa Claus Lane     # ┐
  zip: 12345                   # │ Anchor content
  city: North Pole             # ┘
billing address: *address      #   Alias

order items:
- Sled
- Wrapping Paper
```
*When loaded into a native data structure, the shipping address and billing address point to the same data structure.*
