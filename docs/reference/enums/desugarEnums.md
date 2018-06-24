---
layout: doc-page
title: "Translation of Enums and ADTs"
---

The compiler expands enums and their cases to code that only uses
Scala's other language features. As such, enums in Scala are
convenient _syntactic sugar_, but they are not essential to understand
Scala's core.

We now explain the expansion of enums in detail. First,
some terminology and notational conventions:

 - We use `E` as a name of an enum, and `C` as a name of a case that appears in `E`.
 - We use `<...>` for syntactic constructs that in some circumstances might be empty. For instance,
   `<value-params>` represents one or more a parameter lists `(...)` or nothing at all.

 - Enum cases fall into three categories:

   - _Class cases_ are those cases that are parameterized, either with a type parameter section `[...]` or with one or more (possibly empty) parameter sections `(...)`.
   - _Simple cases_ are cases of a non-generic enum that have neither parameters nor an extends clause or body. That is, they consist of a name only.
   - _Value cases_ are all cases that do not have a parameter section but that do have a (possibly generated) extends clause and/or a body.

  Simple cases and value cases are collectively called _singleton cases_.

The desugaring rules imply that class cases are mapped to case classes, and singleton cases are mapped to `val` definitions.

There are eight desugaring rules. Rule (1) desugar enum definitions. Rules
(2) and (3) desugar simple cases. Rules (4) to (6) define extends clauses for cases that
are missing them. Rules (7) and (8) define how such cases with extends clauses
map into case classes or vals.

1.  An `enum` definition

         enum E ... { <defs> <cases> }

    expands to a `sealed` `abstract` class that extends the `scala.Enum` trait and
    an associated companion object that contains the defined cases, expanded according
    to rules (2 - 8). The enum trait starts with a compiler-generated import that imports
    the names `<caseIds>` of all cases so that they can be used without prefix in the trait.

        sealed abstract class E ... extends <parents> with scala.Enum {
          import E.{ <caseIds> }
          <defs>
        }
        object E { <cases> }

2. A simple case consisting of a comma-separated list of enum names

       case C_1, ..., C_n

   expands to

       case C_1; ...; case C_n

   Any modifiers or annotations on the original case extend to all expanded
   cases.

3. A simple case

       case C

   of an enum `E` that does not take type parameters expands to

       val C = $new(n, "C")

   Here, `$new` is a private method that creates an instance of of `E` (see
   below).

4. If `E` is an enum with type parameters

        V1 T1 > L1 <: U1 ,   ... ,    Vn Tn >: Ln <: Un      (n > 0)

   where each of the variances `Vi` is either `'+'` or `'-'`, then a simple case

        case C

   expands to

       case C extends E[B1, ..., Bn]

   where `Bi` is `Li` if `Vi = '+'` and `Ui` if `Vi = '-'`. This result is then further
   rewritten with rule (7). Simple cases of enums with non-variant type
   parameters are not permitted.

5. A class case without an extends clause

        case C <type-params> <value-params>

   of an enum `E` that does not take type parameters expands to

        case C <type-params> <value-params> extends E

   This result is then further rewritten with rule (8).

6. If `E` is an enum with type parameters `Ts`, a class case with neither type parameters nor an extends clause

        case C <value-params>

   expands to

        case C[Ts] <value-params> extends E[Ts]

   This result is then further rewritten with rule (8). For class cases that have type parameters themselves, an extends clause needs to be given explicitly.

7. A value case

       case C extends <parents>

   expands to a value definition in `E`'s companion object:

       val C = new <parents> { <body>; def enumTag = n; $values.register(this) }

   where `n` is the ordinal number of the case in the companion object,
   starting from 0.  The statement `$values.register(this)` registers the value
   as one of the `enumValues` of the enumeration (see below). `$values` is a
   compiler-defined private value in the companion object.

8. A class case

       case C <params> extends <parents>

   expands analogous to a final case class in `E`'s companion object:

       final case class C <params> extends <parents>

   However, unlike for a regular case class, the return type of the associated
   `apply` method is a fully parameterized type instance of the enum class `E`
   itself instead of `C`.  Also the enum case defines an `enumTag` method of
   the form

       def enumTag = n

   where `n` is the ordinal number of the case in the companion object,
   starting from 0.

### Equality

An `enum` type contains a `scala.Eq` instance that restricts values of the `enum` type to
be compared only to other values of the same enum type. Furtermore, generic
`enum` types are comparable only if their type arguments are. For instance the
`Option` enum type will get the following definition in its companion object:

    implicit def eqOption[T, U](implicit ev1: Eq[T, U]): Eq[Option[T], Option[U]] = Eq

### Translation of Enumerations

Non-generic enums `E` that define one or more singleton cases
are called _enumerations_. Companion objects of enumerations define
the following additional members.

   - A method `enumValue` of type `scala.collection.immutable.Map[Int, E]`.
     `enumValue(n)` returns the singleton case value with ordinal number `n`.
   - A method `enumValueNamed` of type `scala.collection.immutable.Map[String, E]`.
     `enumValueNamed(s)` returns the singleton case value whose `toString`
     representation is `s`.
   - A method `enumValues` which returns an `Iterable[E]` of all singleton case
     values in `E`, in the order of their definitions.

Companion objects of enumerations that contain at least one simple case define in addition:

   - A private method `$new` which defines a new simple case value with given
     ordinal number and name. This method can be thought as being defined as
     follows.

         def $new(tag: Int, name: String): ET = new E {
           def enumTag = tag
           def toString = name
           $values.register(this)   // register enum value so that `valueOf` and `values` can return it.
         }

### Scopes for Enum Cases

A case in an `enum` is treated similarly to a secondary constructor. It can access neither the enclosing `enum` using `this`, nor its value parameters or instance members using simple
identifiers.

Even though translated enum cases are located in the enum's companion object, referencing
this object or its members via `this` or a simple identifier is also illegal. The compiler typechecks enum cases in the scope of the enclosing companion object but flags any such illegal accesses as errors.

### Other Rules

A normal case class which is not produced from an enum case is not allowed to extend
`scala.Enum`. This ensures that the only cases of an enum are the ones that are
explictly declared in it.