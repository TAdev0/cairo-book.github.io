# Associated Items

_Associated Items_ are the items declared in [traits] or defined in
[implementations]. They are called like this because they are defined on an associate
type &mdash; the type in the implementation. Specifically, there are [associated
functions] (including methods, that we already covered in Chapter 5), [associated types], [associated constants], and [associated implementations].

[traits]: ./ch08-02-traits-in-cairo.md
[implementations]: ./ch08-02-traits-in-cairo.md#implementing-a-trait-on-a-type
[associated types]: ./ch11-10-associated-items-in-traits.md#associated-types
[associated functions]: ./ch05-03-method-syntax.md#associated-functions
[associated constants]: ./ch11-10-associated-items-in-traits.md#associated-constants
[associated implementations]: ./ch11-10-associated-items-in-traits.md#associated-implementations

Associated items are useful when the associated item logically is related to the associating item. For example, the `is_some` method on `Option` is intrinsically related to Options, so should be associated.

Every associated item kind comes in two varieties: definitions that contain the actual implementation and declarations that declare signatures for definitions.

## Associated Types

Associated types are _type aliases_ allowing you to define abstract type placeholders within traits. Instead of specifying concrete types in the trait definition, associated types let trait implementors choose the actual types to use. This provides a flexible way to define traits with "placeholder" types that get filled in later.

Let's consider the following `CustomAdd` trait:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:associated_types}}
```

The type `Result` is a placeholder, and the method’s definition that follows shows that it will return values of type `Self::Result`. Implementors of the `CustomAdd` trait will specify the concrete type for `Result`, and the next method will return a value of that concrete type.

Let's suppose now that a function `foo<T, U>` needs the ability to add `T` and `U`. If we had defined a `AddGeneric` trait with an additional generic parameter that is used to describe the result, then this trait and one potential implementation using `u32` type for all involved generic types would have looked like this:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:generics_usage}}
```

with `foo` being implemented as follows:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:foo}}
```

However, when using associated types, we can get the result type from the impl of `CustomAdd`, and we don’t need to pollute `foo` with an additional generic argument. In the following snippet, we define a `CustomAddImplU32` impl of `CustomAdd<T, U>` trait with `Result` type being a `u32` :

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:associated_types_impl}}
```

with `bar` function corresponding to the `foo` function but using an associated type:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:bar}}
```

Finally, we can run `foo`, and `bar` in our `main` and see that they both produce the same result:

```cairo
{{#rustdoc_include ../listings/ch11-advanced-features/listing_10_associated_types/src/lib.cairo:main}}
```

The point is that `bar` don't need to use a third generic type for the addition result type, this information is actually associated with the impl of the `CustomAdd` trait.

## Associated Constants

Associated constants are constants associated with a type. They are declared using the `const` keyword and are defined in a trait or implementation.
In our next example, we are building a game with two character types &mdash; `Wizard` and `Warrior`, and each character type has a constant `strength` attribute. We can model this scenario as follows:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_11_associated_consts/src/lib.cairo:associated_consts}}
```

Since `strength` is fixed per character type, associated consts allow us to bind this constant number to the character trait rather than adding it to the struct or just hardcoding the value in the implementation. It provides an overall more elegant solution.

A potential battle between a `Warrior` and a `Wizard` could look like this:

```cairo
{{#rustdoc_include ../listings/ch11-advanced-features/listing_11_associated_consts/src/lib.cairo:battle}}
```

## Associated Implementations

Associated implementations allow you to declare that a trait implementation must exist for an associated type. This feature is particularly useful when you want to enforce relationships between types and implementations at the trait level.

To understand the utility of associated impls, let's examine the `Iterator` and `IntoIterator` traits from the Cairo core library:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_12_associated_impls/src/lib.cairo:associated_impls}}
```

In this example, the `IntoIterator` trait has an associated type `IntoIter` and an associated impl `Iterator: Iterator<Self::IntoIter>`. Let's break down why this is useful:

1. The `IntoIterator` trait is designed to convert a collection into an iterator.
2. The `IntoIter` associated type represents the specific iterator type that will be created.
3. The associated implementation `Iterator: Iterator<Self::IntoIter>` declares that this `IntoIter` type must implement the `Iterator` trait.

This design ensures that any type implementing `IntoIterator` will produce an iterator that adheres to the `Iterator` trait. The associated implementation creates a binding at the trait level, guaranteeing that:

- The `into_iter` method will always return a type that implements `Iterator`.
- This relationship is enforced for all implementations of `IntoIterator`, not just on a case-by-case basis.

Let's focus on the following example to better grasp the concept of associated impls:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_12_associated_impls/src/lib.cairo:example}}
```

We define a generic struct `TupleThree<T>`, as well as a generic impl `IndexTupleThree<T, +Copy<T>>` corresponding to the `IndexView` trait form the corelib and that implements a `index` method that matches an index to retrieve the corresponding field value of our `TupleThree<T>` struct.

After that, we define a `TupleThreeTrait` trait that contains a `at_index` method and uses an associated impl `impl IndexImpl: core::ops::IndexView<TupleThree<T>` corresponding to our previously implemented `IndexTupleThree<T, +Copy<T>> ` impl. Finally, we implement our `TupleThreeTrait` trait and will use it in the following `main` function:

```cairo, noplayground
{{#rustdoc_include ../listings/ch11-advanced-features/listing_12_associated_impls/src/lib.cairo:main}}
```