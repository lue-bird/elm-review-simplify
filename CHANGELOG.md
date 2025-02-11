# Changelog

## [Unreleased]

The rule now simplifies:
- `Array.fromList []` to `Array.empty`
- `Array.map f Array.empty` to `Array.empty`
- `Array.map identity array` to `array`
- the same operations for `Array.filter` as for other types like `List.filter` and `Set.filter`
- `Array.isEmpty Array.empty` to `True`
- `Array.isEmpty (Array.fromList [ x ])` to `False`
- `List.singleton >> String.fromList` to `String.fromChar`

## [2.1.1] - 2023-09-18

- A very large number of error messages were reworded to be more consistent, precise and descriptive.
- Checks that applied on `[ a ]` now also report for `List.singleton a` (ex: `List.concatMap f (List.singleton 1)` gets simplified to `f 1`)

The simplification `(\x y -> x + y) n m` introduced in [2.1.0] was removed ([#147](https://github.com/jfmengels/elm-review-simplify/pull/147)).

The rule now simplifies:
- `0 // n` to `0`
- `n // 0` to `0`
- `n // 1` to `n`
- `Tuple.first ( a, b )` to `a`
- `Tuple.second ( a, b )` to `b`
- `Tuple.pair a b` to `( a, b )`
- `List.repeat 1 x` to `List.singleton x`
- `List.reverse [ x ]` to `[ x ]`
- `List.intersperse s [ x ]` to `[ x ]`
- `List.concatMap List.singleton x` to `x`
- `String.reverse (String.fromChar a)` to `String.fromChar a`
- `Dict.intersect Dict.empty dict` to `Dict.empty`
- `Dict.diff Dict.empty dict` to `Dict.empty`
- `Dict.diff dict Dict.empty` to `dict`
- `Dict.union dict Dict.empty` to `dict`
- `Random.andThen f (Random.constant x)` to `f x`
- `Random.andThen Random.constant generator` to `generator`
- `Random.andThen (\a -> Random.constant b) generator` to `Random.map (\a -> b) generator`
- `Random.andThen (always thenGenerator) generator` to `thenGenerator`
- `Result.mapError f (if x then Err a else Err b)` to `f (if x then a else b)`
- `Random.map identity generator` to `generator`
- `Random.map (always a) generator` to `Random.constant a`
- `Random.map f (Random.constant x)` to `Random.constant (f x)`
- `Random.list 0 generator` to `Random.constant []`
- `Random.list -1/-2/-3/... generator` to `Random.constant []`
- `Random.list 1 generator` to `Random.map List.singleton generator`
- `Random.list n (Random.constant el)` to `Random.constant (List.repeat n el)`
- `Random.uniform a []` to `Random.constant a`
- `Random.weighted ( weight, a ) []` to `Random.constant a`
- `Random.weighted tuple []` to `Random.constant (Tuple.first tuple)`
- `List.member (List.singleton b) b` to `b == b` when [`expectNaN`] is enabled (and to `True` otherwise)

Bug fixes:
- Fixed an issue where `Dict.size (Dict.fromList [...])` would be fixed to an incorrect value
- Fixed an issue where `Result.toMaybe (if c then Err a else Ok b)` would be fixed to `Nothing`
- Fixed an issue where `Maybe.andThen (always (Just a)) maybe` would be fixed to `maybe`

## [2.1.0] - 2023-08-15

New opt-in configuration option [`expectNaN`] which will disable some simplifications when the user indicates their
project is likely to use `NaN` values. This disables the following simplifications:
- `x == x` to `True`
- `List.member x [ x ]` to `True`
- `n * 0` to `0`
- `not (a < b)` to `a >= b` (similarly for `>`, `<=`, `>=`, `==` and `/=`)

The rule now simplifies:
- `List.any ((==) x) list` to `List.member x list`
- `List.any (\y -> x == y) list` to `List.member x list`
- `n - n` to `0`
- `-n + n` to `0`
- `0 / n` to `0`
- `n * 0` to `0` is now autofixed

The rule now reports:
- Immediately invoked anonymous functions `(\x y -> x + y) 1 2`. This is very simplifiable but there is no autofix because there are varied ways to simplify it.
  - EDIT: This was removed in [2.1.1].

Bug fixes:
- Fixed an issue where `[ [ 1 ], [ 2 ] ] |> List.concat` would be incorrectly fixed and cause a compiler error

Misc:
- Improved error positioning and fixes for errors related to the usage of operators

## [2.0.33] - 2023-08-13

The rule now simplifies:
- `a |> f >> g` to `a |> f |> g`

## [2.0.32] - 2023-07-11

The rule now simplifies:
- `List.concat [ a, [], b ]` to `List.concat [ a, b ]`

## [2.0.31] - 2023-06-25

Now avoids simplifying `String.replace` when the pattern to find contains `\r`.

## [2.0.30] - 2023-06-25

Fixed an issue where `String.replace` would be fixed incorrectly ([01d3ff](https://github.com/jfmengels/elm-review-simplify/commit/01d3ff0e1ce920b5fcece74fed8d12537b7813c9))

## [2.0.29] - 2023-04-17

The rule now simplifies:
- `Html.Attributes.classList [ x, y, ( z, False ) ]` to `Html.Attributes.classList [ x, y ]`
- `Html.Attributes.classList [ ( onlyOneThing, True ) ]` to `Html.Attributes.class onlyOneThing`
- `Set.fromList [ a ]` to `Set.singleton a`
- `Dict.partition f Dict.empty` to `( Dict.empty, Dict.empty )`
- `Dict.partition (always True) dict` to `( dict, Dict.empty )`
- `Dict.partition (always False) dict` to `( Dict.empty, dict )`
- `Result.toMaybe (Ok x)` to `Just x`
- `Result.toMaybe (Err e)` to `Nothing`
- `Result.mapError identity x` to `x`
- `Result.mapError f (Ok x)` to `Ok x`
- `Result.mapError f (Err x)` to `Err (f x)`
- `List.map Tuple.first (Dict.toList dict)` to `Dict.keys dict`
- `List.map Tuple.second (Dict.toList dict)` to `Dict.values dict`

We now also do a better job at figuring what code is the same:
- `(f >> g) a == (g << f) a` will now be replaced by `True`.
- `(\f -> List.map f)` is considered equivalent in all simplifications to `List.map` (and similarly for a number of other functions).

Bug fixes:
- Fixed an issue where `List.append` would be fixed incorrectly ([#105](https://github.com/jfmengels/elm-review-simplify/issues/105))

## [2.0.28] - 2023-02-25

- Fixed an issue where errors for `List.foldl` operations would be incorrectly fixed ([#86](https://github.com/jfmengels/elm-review-simplify/issues/86))

## [2.0.27] - 2023-02-21

- Fixed an issue where `String.fromList [ f x ]` would incorrectly be changed to `String.fromChar f x` ([#85](https://github.com/jfmengels/elm-review-simplify/issues/85))

## [2.0.26] - 2023-02-06

The simplification `String.slice 0 n str` -> `String.left n str` has been removed because they were not necessarily equivalent. In the case where `n` is negative, then the behavior of the 2 functions differ.

The rule now simplifies:
- `List.member a []` to `False`
- `List.member a [ a, b, c ]` to `True`


## [2.0.25] - 2023-02-02

The rule now simplifies:
- `String.fromList []` to `""`
- `String.fromList [ a ]` to `String.fromChar a`
- `List.append [] list` to `list`
- `List.head []` to `Nothing`
- `List.head (a :: bToZ)` to `a`
- `List.tail []` to `Nothing`
- `List.tail (a :: bToZ)` to `bToZ`
- `List.sum []` to `0`
- `List.sum [ a ]` to `a`
- `List.product []` to `1`
- `List.product [ a ]` to `a`
- `List.minimum []` to `Nothing`
- `List.minimum [ a ]` to `Just a`
- `List.maximum []` to `Nothing`
- `List.maximum [ a ]` to `Just a`
- `List.map2 fn xs []` to `[]` (same for up to `List.map5`)
- `List.map2 fn [] ys` to `[]` (same for up to `List.map5`)
- `List.unzip []` to `( [], [] )`
- `List.foldl f x (Set.toList set)` to `Set.foldl f x set`

All the changes in this release were contributed by [@lue-bird](https://github.com/lue-bird).

## [2.0.24] - 2023-01-20

All the changes in this release were contributed by [@lue-bird](https://github.com/lue-bird).

The rule now simplifies:
- `String.slice n n str` to `""`
- `String.slice 0 n str` to `String.left n str`
- `String.slice n 0 str` to `""`
- `String.slice a z ""` to `""`
- `String.left 0 str` to `""`
- `String.left -1 str` to `""`
- `String.left n ""` to `""`
- `String.right 0 str` to `""`
- `String.right -1 str` to `""`
- `String.right n ""` to `""`
- `String.slice 2 1 str` to `""`
- `String.slice -1 -2 str` to `""`
- `List.sortBy (\_ -> a) list ` to `list`
- `List.sortBy identity list ` to `List.sort list`
- `List.sortWith (\_ _ -> LT) list ` to `List.reverse list`
- `List.sortWith (\_ _ -> EQ) list ` to `list`
- `List.sortWith (\_ _ -> GT) list ` to `list`

The following simplifications for `List.sort` also work for `List.sortBy fn` and `List.sortWith fn`:
- `List.sort []` to `[]`
- `List.sort [ a ]` to `[ a ]`

The following simplifications for List.foldl also work for `List.foldr`:
- `List.foldl fn x []` to `x`
- `List.foldl (\_ soFar -> soFar) x list ` to `x`
- `List.foldl (+) 0 list ` to `List.sum list`
- `List.foldl (+) initial list ` to `initial + List.sum list`
- `List.foldl (*) 1 list ` to `List.product list`
- `List.foldl (*) 0 list ` to `0`
- `List.foldl (*) initial list ` to `initial * List.product list`
- `List.foldl (&&) True list ` to `List.all identity list`
- `List.foldl (&&) False list ` to `False`
- `List.foldl (||) False list ` to `List.any identity list`
- `List.foldl (||) True list ` to `True`


## [2.0.23] - 2022-11-08

Add better support for `jfmengels/elm-review` v2.10.0.

## [2.0.22] - 2022-11-03

Fixed an issue where `0 - f x` would be simplified to `-f x` instead of `-(f x)` [#52]

## [2.0.21] - 2022-09-06

Fixed an issue in where let declarations would not be fused when there was only a single element in the first let declaration.

## [2.0.20] - 2022-09-06

The rule now simplifies:
1. Applying record access simplification over if and case expressions.
```elm
(if condition then
   { record | a = 1 }
 else
   { record | field = 2 }
).field
-->
 if condition then
   { record | a = 1 }.field
 else
   { record | field = 2 }.field
```
when all patterns can later be simplified. In this example the final result will be
```elm
if condition then
  record.field
else
  2
```

This also applies to case expressions when all branches can be simplified. Thanks [@miniBill]! [#40]

2. Simplify case expressions that can be simplified to let variables [#48]

```elm
a =
  case value of
    { x, y } ->
      1
-->
a =
  let
    { x, y } =
      value
  in
  1
```

3. Merging multiple let declarations

```elm
let
    a = 1
in
let
    b = 1
in
a + b
-->
let
    a = 1
    b = 1
in
a + b
```

## [2.0.19] - 2022-08-29

The rule now DOESN'T (it did before) simplify case of expressions where all the branches have the same code when one of
the patterns references a custom type from your project. For example
```elm
case x of
  A -> 1
  B -> 1
  C -> 1
```
does not get simplified to `1` like before. But the simplification still happens if the patterns only reference custom
types that come from dependencies or `elm/core`, like
```elm
case x of
  Just _ -> 1
  Nothing -> 1
--> 1
```

The reasoning is that often you want the compiler to give you a reminder when you introduce a new custom type, which this
simplification made very hard. It also sometimes created some worse code when you pattern matched on a custom type with
only a single constructor.

The configuration setting `Simplify.ignoreCaseOfForTypes` now only takes custom types from dependencies. Any type
provided to this function that is not found in the dependencies will now trigger a global error.
It is likely that you won't need this function anymore. If you do, please open an issue because I'd love to know. 

A number of `elm-review` users didn't use `Simplify` because of the simplification above, so I'm hoping that this change will make
you able to use the rule again.

The rule now simplifies:
- `{ a = 1, b = 2 }.a` to `1`. Thanks [@miniBill]! [#35]
- `{ foo | b = 1 }.a` to `foo.a`. Thanks [@miniBill]! [#37]
- `if a == "a" then if a == "b" then 1 else 2 else 3` to `if a == "a" then 2 else 3`

## [2.0.18] - 2022-08-14

Improves the error message for some simplifications.

## [2.0.17] - 2022-08-14

#### Removal of unreachable `if` branches [#31]

`Simplify` now has the ability to infer values from if conditions, which it will use to simplify boolean expressions and even to remove some `if` branches.
```elm
if a && b then
  if a then -- we know this must be true
    1
  else -- so we can remove this else
    2
else
  3
```

It should also be able to catch things like

```elm
if x == 1 then
  if x == 2 then -- we know this must be false
    ...
```

Note that this will (purposefully) only simplify boolean expressions by what has been inferred from conditions. Therefore,
the following will not be simplified. The reasoning behind the decision is that you would not be able to write code like
below, which can be useful if you want to rely on top-level constants that you may wish to change at a later point in time.

```elm
enableDevMode = False

value =
    if enableDevMode then
```


#### Simplifications

The rule now simplifies:
- `(a < b) == (b > a)` to `True`
- `(a <= b) == (b >= a)` to `True`
- `(a && b) == (b && a)` to `True`
- `(a || b) == (b || a)` to `True`
- `Dict.member x Dict.empty` to `False`

## [2.0.16] - 2022-07-16

The rule now simplifies:
- `.field a == a.b` to `True`
- `a |> fn == fn a` to `True`
- `fn <| a == fn <| a` to `True`


## [2.0.15] - 2022-05-05

The rule now simplifies:
- `List.member x []` to `False`
- `Set.member x Set.empty` to `False`
- `List.intersperse x []` to `[]`


## [2.0.14] - 2022-04-22

The rule now simplifies:
- `List.indexedMap f []` to `[]`
- `List.indexedMap (\_ value -> f value) list` to `List.map (\value -> f value) list`
- `List.indexedMap (always f) list` to `List.map f list`


## [2.0.13] - 2022-04-21

The rule now simplifies:
- `Maybe.andThen (\b -> let y = 1 in Just y) maybe` to `Maybe.map (\b -> let y = 1 in y) maybe`
- `Result.andThen (\b -> let y = 1 in Ok y) result` to `Result.map (\b -> let y = 1 in y) result`


## [2.0.12] - 2022-04-09

The rule now simplifies:
- `List.concatMap (\a -> [ b ]) list` to `List.map (\a -> b) list` 
- `Maybe.andThen (\a -> if condition a then Just b else Just c) maybe` to `Maybe.map (\a -> if condition a then b else c) maybe`
- `List.filterMap (\a -> if condition a then Just b else Just c) maybe` to `List.map (\a -> if condition a then b else c) maybe`


## Missing changelog

Help would be appreciated to fill the blanks!

[Unreleased]: https://github.com/jfmengels/elm-review-simplify/compare/v2.1.1...HEAD
[2.1.1]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.1.1
[2.1.0]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.1.0
[2.0.33]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.33
[2.0.32]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.32
[2.0.31]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.31
[2.0.30]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.30
[2.0.29]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.29
[2.0.28]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.28
[2.0.27]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.27
[2.0.26]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.26
[2.0.25]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.25
[2.0.24]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.24
[2.0.23]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.23
[2.0.22]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.22
[2.0.21]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.21
[2.0.20]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.20
[2.0.19]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.19
[2.0.18]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.18
[2.0.17]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.17
[2.0.16]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.16
[2.0.15]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.15
[2.0.14]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.14
[2.0.13]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.13
[2.0.12]: https://github.com/jfmengels/elm-review-simplify/releases/tag/2.0.12

[#31]: https://github.com/jfmengels/elm-review-simplify/pull/31
[#35]: https://github.com/jfmengels/elm-review-simplify/pull/35
[#37]: https://github.com/jfmengels/elm-review-simplify/pull/37
[#40]: https://github.com/jfmengels/elm-review-simplify/pull/40
[#48]: https://github.com/jfmengels/elm-review-simplify/pull/48
[#52]: https://github.com/jfmengels/elm-review-simplify/pull/52

[@miniBill]: https://github.com/miniBill

[`expectNaN`]: https://package.elm-lang.org/packages/jfmengels/elm-review-simplify/latest/Simplify#expectNaN