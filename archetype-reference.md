# Reference

An archetype contract is composed of declarations, entry points and optionally specification for formal verification.

## Declarations

* `constant` and `variables`  declare global variables. A constant value cannot be modified in the contract's entry points.

```ocaml
constant rate : rational = 0.2
```

```ocaml
variable price : tez = 10tz
```

* `states`  declares contract's possible states. It is then possible to use _transitions_ to change the contract states \(see below\)

```ocaml
states =
| Created initial
| Confirmed
| Canceled
| Success
| Fail
```

 `initial`  is used to declare the initial state value of the contract. 

* `entry` declares an entry point of the contract. An _entry_ has the following sections:
  * `specification` \(optional\) to provide the _post conditions_ the entry is supposed to have
  * `accept transfer` \(optional\) to specify that transfer of tez is accepted 
  * `called by` \(optional\) to declare which role may call this entry
  * `require` \(optional\) to list the necessary conditions for the entry to be executed
  * `failif` \(optional\) to list the conditions which prevent from execution 
  * `effect` is the code to execute by the entry

```css
entry an_entry_1 (arg1 : string, arg2 : int) {
  specification {
    // see 'specification' section below
  }
  called by a_role
  require {
    r1 : arg2 > 0 
  }
  failif {
    f1 : arg2 > 10
  }
  effect {
    // ...
  }
}
```

An entry with just an effect section may omit the effect section syntax:

```css
variable val : nat = 0

entry add(v : nat) {
  val += v;
}
```

* `transition` declares an entry point of the contract that changes the state of the contract. A transition has the same sections as an entry \(except `effect`, see above\) plus:
  * `from` to specify the states the transition starts from
  * `to` to specify the states after the transition 
  * `when` \(optional\) to specify the transition condition
  * `with effect` to specify the effect of the transition

```c
states =
| Created initial
| Confirmed
| Canceled
| Success
| Fail

transition confirm () {
  from Created
  to Confirmed
  when { transferred > 10tz }
  with effect {
    transfer 1tz to @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
  }
}
```

It is possible to specify several `to ... when .... with effect ...` sections in a transition. It is possible to specify a list of original states for the transition to start from:

```c
transition to_fail () {
  from Created or Confirmed
  to Fail
  when { ... }
}
```

## Builtin types

* `bool` : boolean values
* `int` : integer values 
* `nat` : positive integers
* `rational` : floating value that can be expressed as the quotient or fraction of two integers 
* `address` : account address
* `role` : an address that can be used in entry's called by section
* `date` : date values
* `duration` : duration values \(in second, minute, hour, day, week\)
* `string` : string of characters
* `tez` : Tezos currency
* `bytes` : bytes sequence
* `key` : key value
* `signature` : signature value
* `key_hash` : key hash value
* `chain_id` : chain id value

## Composite types

* `*` is the type of tuples.

```c
constant pair : int * string = (1, "astr")
```

* `option` is the type of optional value.

```csharp
constant value : option<int> = none
```

* `list` declares a list of any type \(builtin or composed\)

```csharp
variable vals : list<string> = []
```

* `map` declares a map from a non-composite builtin type to any type.

```csharp
variable assoc : map<int, string> = []
```

* `set` declares a set of non-composite builtin type values.

```csharp
variable poll : set<address> = []
```

* `record` declares a record structure.

```css
record r {
   s : string;
   i : int;
}
```

* `asset`  declares a collection of assets and the data an asset is composed of. For example, the following declares a collection of real estates described by an address, a location, and an owner:

```css
asset real_estate identified by addr {
  addr      : string;
  xlocation : string;
  ylocation : string;
  owner     : address;
}
```

* `enum`  declares an enumeration value. It is read with a `match ... with` command \(see below\). 

```ocaml
enum color =
| White 
| Red
| Green
| Blue
```

It is then possible to declare a variable of type _color_:

```css
variable c : color = Green
```

* `contract` is the type of contract entry point signature.

```css
variable b : option<contract<nat>> = entrypoint(anaddress, "getbalance")
```

It is then possible to declare a contract value of type _called\_contract\_sig_ and set its address:

```c
constant c : called_contract_sig = @KT1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm
```

* `aggregate` declares an asset field as an aggregate of another asset.

```css
asset an_asset {
  id     : string;
  a_val  : int;
  col    : aggregate<another_asset>;
}
```

* `partition` declares an asset field as a collection of another asset. The difference with `aggregate` is that a partition ensures at compilation that every _partitioned_ asset \(i.e. element of the partition\) belongs to one and only _partitioning_ asset.

```css
asset partitioning_asset {
  id   : string;
  part : partition<partionned_asset>;
}
```

As a consequence of the partition, a _partitioned_ asset cannot be straightforwardly added or removed to its global collection with `add` and `remove` \(see [Partitions, Reference](archetype-language/subsets-partition.md) section\). 

## Effect

An entry's effect is composed of instructions separated by a semi-colon. 

### Declaration

* `var` declares a local variable with its value. While possible, it is not necessary to declare the type a local variable. Local variables' identifier must be unique \(no name capture in Archetype\).

```javascript
var i = 0;
```

### Assignment

* `:=` enables to assign a new value to a global or local variable. 

```javascript
i := 1;
```

* `+=` , `-=` , `*=` and `/=` enable to respectively increment and decrement an integer variable \(local or global\).

```javascript
i *= 2;  // i := i * 2
d += 3w; // d := i + 3w, date d is now previous d plus 3 weeks
```

### Effects on asset collection

* `add` : adds an asset to an asset collection \(fails if the asset key is already present in the collection\)
* `update` : updates an asset \(fails if the id is not present in the asset collection\)
* `addupdate` is similar to update except that it adds the asset only if its identifier is not present in the collection
* `remove` removes an asset from its collection \(does not fail\).
* `removeif` enables removing assets under a condition
* `clear` clears an asset collection

```c
an_asset.add({ id = "RJRUEQDECSTG", asset_col = [] }); // see 'collection' example
an_asset.update("RJRUEQDECSTG", { a_value += 3 });
an_asset.addupdate("RJRUEQDECSTG", { a_value += 3 });
an_asset.remove("RJRUEQDECSTG");
an_asset.removeif(a_val > 10); // "a_val" is an asset field
an_asset.clear();
```

See [Assets](archetype-language/data-model.md) section for details.

### Control

* `;` sequence

```javascript
res := 1;
res := 2;
res := 3;
```

* `if then` and `if then else` are the conditional branchings instructions. 

```ocaml
if i > 0 then (
  // (* effect when i > 0 *) 
) else (
  // (* effect when i <= 0 *)
);
```

* `match ... with ... end` 

```javascript
archetype effect_control_matchwith

enum t =
  | C1
  | C2
  | C3
  | C4
  | C5

variable res : int = 0

entry exec () {
  specification {
    s0: res = 1;
  }
  effect {
    var x : t = C3;
    match x with
    | C1 | C2 -> res := 0
    | C3 -> res := 1
    | _ -> res := 2
    end
  }
}

```

* `for in do done` iterates over a collection.

```ocaml
 var res = 0;
 for a in an_asset do
   res += a.a_val;
 done;
```

* `iter to do done` iterates over a sequence of integer values \(starting from 1\).

```javascript
var res = 0
iter i to 3 do      // iterates from 0 to 3 included
  res += i;
done;               // res is 6
```

* `transfer` transfers an amount of tez to an address or a contract.

```c
transfer 2tz to @tz1RNB9PXsnp7KMkiMrWNMRzPjuefSWojBAm;
```

* `fail` aborts the execution. It prevents from deploying the contract on the blockchain. As a consequence the storage is left unchanged when executed.

```c
fail("a message");
```

* `dorequire` and `dofailif` fail if the argument condition is respectively false and true. `dorequire(t, msg)` is  sugar for `if t then fail(msg)`.

```c
dorequire(val > 0, "KO");
dofailif(val <= 0, "KO");
```

## Expressions

### bool

```javascript
constant x : bool = true
constant y : bool = false
```

* `and` 
* `or` 
* `not` 
* `=`
* `<>`
* `xor`

```javascript
var b1 = (true and false);
var b2 = (true or false);
var b3 = not true;
var b4 = true = false;
var b5 = true <> false;
var b6 = true xor false;
```

### nat

```javascript
constant i : nat = 1
constant j : nat = 42
```

* `+` `-` `*` `div` `%` 
* `<` `>` `=` `<>` `<=` `>=`
* `min` `max`
* `abs`

```javascript
var n1 = 1 + 2; 
var i1 : int = 1 - 2; /* difference of nats return an integer value */
var n2 = 3 * 4;
var n3 = 45 div 2; /* 22 (euclidean division) */
var n4 = 45 % 2; /* 1 (modulo) */ 
var n5 = min(5, 6);
var n6 = abs(-6);
```

### int

```c
constant i : int = 1i
constant j : int = -42
```

* `+` `-` `*` `div` `%` 
* `<` `>` `=` `<>` `<=` `>=`
* `min` `max` 

```javascript
var a : int = 1i;
var b : int = 2i;
var c : int = 3i;
var d : int = 4i;
var i1 = a + b; 
var i2 = b - a;
var i3 = c * d;
var i4 = d div c; 
var n1 = d % c; 
var i5 = max(6i, 8i);
```

### rational

```c
constant f : rational = 1.1
constant g : rational = -1.1
constant r : rational = 2 / 6
constant t : rational = -2 / 6
```

* `+` `-` `*` `/`  
* `<` `>` `=` `<>` `<=` `>=`
* `min` `max` `abs` `floor` `ceil`

```javascript
var b1 = (1 / 2) <> 2;
var b2 =  1 <> (2 / 3);
var b3 = (1 / 2) = (2 / 4);
var b4 = (1 / 2) < (2 / 4);
var r1 = (4 / 5) + (7 / 6);
var r2 = 3 + (1 / 2);
var r3 = (4 / 3) * (3 / 4);
var r4 = (8 / 9) / (3 / 7);
var i1 = floor(5 / 3);  // int 1
var i2 = floor(-5 / 3); // int -2
var i3 = ceil(5 / 3);   // int 2
var i4 = ceil(-5 / 3);  // int -1
```

### string

```c
constant s : string = "hello world"
```

* `concat` `+`
* `slice`
* `length`
* `<` `>` `=` `<>` `<=` `>=`
* `to_string`

```javascript
var s1 = "str1" + "str2"; /* concat */
var s2 = slice("abcdef", 1, 2);
var l  = length("archetype");
var b1 = "a" <> "b";
var b2 = "a" < "b";
var s  = to_string(42)
```

### tez

```css
constant ctz  : tez = 1tz
constant cmtz : tez = 1mtz
constant cutz : tez = 1utz
```

* `+` 
* `<` `>` `=` `<>` `<=` `>=`
* `min` `max`

```javascript
var b1 = 1tz <= 2tz;
var b2 = 1tz < 2tz;
var b3 = 1tz <> 2tz;
var t1 = 1tz + 1tz;
var t2 = 2 * 4tz;
var t3 = 1/2 * 8tz;
```

### address

```c
constant a : address = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg
```

* `=` `<>` 

```javascript
var b1 = @tz1Lc2qBKEWCBeDU8npG6zCeCqpmaegRi6Jg <> @tz1bfVgcJC4ukaQSHUe1EbrUd5SekXeP9CWk;
```

### duration

```c
// duration of 3 weeks 8 days 4 hours 34 minutes 18 seconds
constant d : duration = 3w8d4h34m18s 
```

* `+` `-` 
* `<` `>` `=` `<>` `<=` `>=`
* `min` `max`

```javascript
var b1 = 1h > 2h;
var b2 = 1h >= 2h;
var d1 = 1h + 2s;
var d2 = 1h - 2s; /* absolute value */
```

### date

```c
constant date0 : date = 2019-01-01                
constant date1 : date = 2019-01-01T01:02:03       
constant date2 : date = 2019-01-01T01:02:03Z      
constant date3 : date = 2019-01-01T00:00:00+01:00 
constant date4 : date = 2019-01-01T00:00:00-05:30 
```

* `<` `>` `=` `<>` `<=` `>=`
* `min` `max`

```javascript
var b1 = 2020-01-01 > 2020-12-31;
var b2 = 2020-01-01 >= 2020-12-31;
var d1 = 2020-01-01 + 1d;
var d2 = 2020-01-01 - 1d;
```

### bytes

```c
constant bl : bytes = 0x12f12354356a 
```

* `slice`
* `length`
* `concat`
* `=` `<>`
* `blake2b` `sha256` `sha512` 
* `pack` `unpack`

```javascript
var b1 = concat(0x12, 0xef);
var b2 = slice(0xabcdef01, 1, 2);
var h1 = blake2b(0x050100000009617263686574797065);
var h2 = sha256(0x050100000009617263686574797065);
var h3 = sha512(0x050100000009617263686574797065);
var b3 = pack("archetype");
var s : string option = unpack<string>(0x050100000009617263686574797065);
```

### option

```c
constant op1 : option<int> = none
constant op2 : option<int> = some(0)
```

* `issome`
* `isnone`
* `opt_get`

```css
effect {
    var t1 : bool = isnone(none);
    var t2 : bool = issome(some(1));
    var i  : int = opt_get(some(1i)); /* i = 1 */
}
```

### list

* `head_tail`
* `contains`
* `length`
* `nth`
* `prepend`

```css
effect {
    var l : list<string> = ["1"; "2"; "3"];
    var ht = head_tail(l);
    var t1 = contains(l, "2");
    var t2 = length(l);
    var n  = nth(l);
    var p  = prepend(l,"0");
}
```

### set

* `add`
* `remove`
* `contains`
* `length`

```css
effect {
    var my_set : set<int> = [0; 1 ; 2; 3];
    var new_set2 : set<int> = add(my_set, 4);
    var new_set3 : set<int> = remove(my_set, 0);
    var set_c    : bool     = contains(my_set, 2);
    var c        : nat      = length(my_set);
}
```

### map

* `put`
* `remove`
* `[]`
* `getopt`
* `contains`
* `length`

```css
effect {
    var my_map : map<string, int> = [ ("k0", 3) ;
                                      ("k1", 2) ;
                                      ("k2", 1) ;
                                      ("k3", 0) ];
    var new_map1 : map<string, int> = put(my_map, "k4", 4);
    var new_map2 : map<string, int> = remove(my_map, "k0");
    var new_map3 : option<int>      = my_map["k0"];
    var new_map4 : int              = getopt(my_map, "k0");
    var map_c    : bool             = contains(my_map, "k0");
    var map_l    : nat              = length(my_map);
}
```

### asset 

* `contains`
* `count`
* `nth`
* `head`
* `tail`
* `select`
* `sort`
* `sum`

```css
archetype expr_method_asset_contains

asset my_asset identified by id {
  id    : string;
  value : int;
} initialized by {
  {"id0"; 0i};
  {"id1"; 1i};
  {"id2"; 2i}
}

variable res : bool = false

entry exec () {
  specification {
    s0 : res = true;
  }
  effect {
    var t = my_asset.contains("id0");
    var c = my_asset.count();
    var n = my_asset.nth(1);
    var l1 = my_asset.head(3);
    var l2 = my_asset.tail(2);
    var l3 = my_asset.select(the.id < "id2");
    var l4 = my_asset.sort(value);
    var l5 = my_asset.sort(v1, asc(v2), desc (v3));
    var s = my_asset.sum(value);
  }
}
```

### Constants

* `caller` 

```javascript
var v : address = caller; /* the address calling the contract */
```

* `source` 

```javascript
var v : address = source; /* the address at the origin of the call */
```

* `balance` 

```javascript
var v : tez = balance; /* the contract's balance in tez */
```

* `transferred` 

```javascript
var v : tez = transferred; /* the amount of tez that comes with the call */
```

* `now` 

```javascript
var v : date = now; /* the date at execution */
```

* `state` 

```javascript
archetype sample_state

states =
| First
| Second
| Third


transition mytr () {
  from First
  to Second
  with effect {
    require (state = First)
  }
}
```

* chainid 

```javascript
var v : chain_id = chainid;
```

## Verification

### Specification

Here is a full example with all sections of an entry specification

```css
archetype contract_with_full_specification

asset myasset {
  id : string;
  val: bool;
}

asset col1 {
  id1 : string;
}

asset col2 {
  id2 : string;
}

entry exec () {
 specification {

    definition mydef {
      x : myasset | forall y in col1, x.id = y.id1
    }

    predicate mypredicate (a : int) {
      forall x in col1, forall y in col2, x.id1 = y.id2
    }

    variable myvar : int = 0

    shadow effect {
      myvar := 3
    }

    assert a1 {
      x = y
      invariant for myloop {
        x = 0;
        y = 0
      }
    }

    postcondition s1 {
      x = y
      invariant for myloop {
        x = 0;
        y = 0
      }
    }
  }

  effect {
    var x : int = 0;
    var y : int = 0;
    for : myloop c in col1 do
      require(true)
    done;
    assert a1
  }
}

```

* `definition`

```css
definition mydef {
  x : myasset | forall y in col1, x.id = y.id1
}
```

* `predicate` 

```css
predicate mypredicate (a : int) {
  forall x in col1, forall y in col2, x.id1 = y.id2
}
```

* `variable` 

```css
variable myvar : int = 0
```

* `shadow effect` 

```css
shadow effect {
  myvar := 3
}
```

* `assert` 

```css
assert a1 {
  x = y
  invariant for myloop {
    x = 0;
    y = 0
  }
}
```

* `invariant` 

```css
invariant for myloop {
  x = 0;
  y = 0
}
```

* `postcondition` 

```css
postcondition s1 {
  x = y
  invariant for myloop {
    x = 0;
    y = 0
  }
}
```

One specification section is available at the top of the contract. But there is neither `shadow effect` nor `postcondition`, which is replaced by `contract invariant` for the last one.

* `contract invariant` 

```css
contract invariant c1 {
  true <> false
}
```

### Expression

All expression in effect are available in formula part.

* `forall` universal quantifier

```css
q1: forall x : int, x = x;
q2: forall x in my_asset, x.value = x.value;
```

* `exists` existential quantifier

```css
q3: exists x : int, x = x;
q4: exists x in my_asset, x.value = x.value;
```

* `->` imply

```css
o1: true -> true;
```

* `<->` equivalence

```css
o2: true <-> true;
```

### Asset expression

* `subsetof` 

```css
archetype expr_formula_asset_method_subset

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

entry exec () {

  specification {
    s: my_asset.subsetof(my_asset);
  }

  effect {
    require (true)
  }
}
```

* `isempty` 

```css
archetype expr_formula_asset_method_isempty

asset my_asset identified by id {
  id : string;
  value : int;
} initialized by {
  {"id0"; 0};
  {"id1"; 1};
  {"id2"; 2}
}

entry exec () {

  specification {
    s: my_asset.isempty();
  }

  effect {
    require (true)
  }
}
```

### Extra asset collection

* `before` 

```css
s1: before.my_asset.isempty();
```

* `at` 

```css
s2: at(lbl).my_asset.isempty();
```

* `unmoved` 

```css
s3: unmoved.my_asset.isempty();
```

* `added` 

```css
s4: added.my_asset.isempty();
```

* `removed` 

```css
s5: removed.my_asset.isempty();
```

* `iterated` 

```css
entry exec () {
  specification {
    postcondition p1 {
      true
      invariant for loop {
         iterated.isempty()
      }
    }
  }

  effect {
    var res : int = 0;
    for:loop i in my_asset do
      res += 1;
    done
  }
}
```

* `toiterate` 

```css
entry exec () {
  specification {
    postcondition p1 {
      true
      invariant for loop {
         toiterate.isempty() 
      }
    }
  }

  effect {
    var res : int = 0;
    for:loop i in my_asset do
      res += 1;
    done
  }
}
```

### Security predicate

```css
archetype lang_security

constant admin : role = @tz1aazS5ms5cbGkb6FN1wvWmN7yrMTTcr6wB

asset my_asset identified by id {
  id : string;
  value : int;
}

entry exec () {
  effect {
    ()
  }
}

security {
  s00 : only_by_role (anyentry, admin);
  s01 : only_in_entry (anyentry, exec);
  s02 : only_by_role_in_entry (anyentry, admin, exec);
  s03 : not_by_role (anyentry, admin);
  s04 : not_in_entry (anyentry, exec);
  s05 : not_by_role_in_entry (anyentry, admin, exec);
  s06 : transferred_by (anyentry);
  s07 : transferred_to (anyentry);
  s08 : no_storage_fail (anyentry);
}

```

* `only_by_role` 

```css
s00 : only_by_role (anyentry, admin);
```

* `only_in_entry` 

```css
s01 : only_in_entry (anyentry, exec);
```

* `only_by_role_in_entry` 

```css
s02 : only_by_role_in_entry (anyentry, admin, exec);
```

* `not_by_role` 

```css
s03 : not_by_role (anyentry, admin);
```

* `not_in_entry` 

```css
s04 : not_in_entry (anyentry, exec);
```

* `not_by_role_in_entry` 

```css
s05 : not_by_role_in_entry (anyentry, admin, exec);
```

* `transferred_by` 

```css
s06 : transferred_by (anyentry);
```

* `transferred_to` 

```css
s07 : transferred_to (anyentry);
```

* `no_storage_fail` 

```css
s08 : no_storage_fail (anyentry);
```

