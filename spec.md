# cBNF specification

cBNF v1.1  
Madeline Kahn, 2025

## Rationale

cBNF is a Backus–Naur form derivative designed to provide many useful extensions, including all features provided by EBNF, while including most of BNF's basic features in the same form as they appear in standard BNF. For example (as long as the document's string literals contain no `\`'s) you can convert a standard BNF syntax to a cBNF syntax simply by appending a `;` to the end of each rule.

The c stands for _cute_.

## Structure

A cBNF syntax consists of zero or more production **rules**. Optional whitespace is allowed between most tokens.

A rule assigns an **expression** to an **identifier**. The standard assignment operator is `::=`. Each rule must be terminated by `;`.

```cbnf
// Rule example:
<greeting> ::= "Hello, world!" ;
```

### Comments

A comment may start anywhere arbitrary whitespace is permitted. C-style line comments and block comments are both allowed.

```cbnf
// line comment
/* block comment
on two lines */
```

### Literals

A string **literal** (terminal symbol) may be double-quoted (preferred) or single-quoted, and backslash escape sequences are allowed. _The details of escape sequence handling may be implementation-defined._

### Identifiers

An **identifier** (nonterminal symbol) is written as one or more identifier characters enclosed in angle brackets (actually `<` and `>`). Identifier characters consist of all letters, digits, and the hyphen (`-`). "Letters" may include any characters used to represent words in a supported natural language, and "digits" may include any characters used to represent integers in a supported numeral system (as long as neither contain `<` or `>`). Identifiers should be considered case-sensitive. The recommended naming convention for identifiers is snake case (ex. `<street-address>`), except for "pseudo-terminal" identifiers which should use all caps (ex. `<EOL>`).

### Expressions

An **expression** can usually be considered to be an alternation of one or more concatenations of one or more terms each. (The single other type of expression is the exception, described at the end of this section.) Alternatives are separated by `|`, and concatenation needs no operator.

```cbnf
// Alternation example:
<floor-label> ::= "Ground Floor" | "Floor " <positive-integer> ;
```

There are various types of terms that may be used in an expression. The **literal** and **identifier** have already been discussed above, while the **group**, **quantification**, and **description** will be discussed below.

#### Group

A **group** is a term consisting of any expression in parentheses. Groups are used to nest expressions within one another.

```cbnf
// Group example:
<example-domain> ::= "example." ("com" | "org" | "net") ;
```

#### Quantification

A **quantification** is a term consisting of any expression in curly braces, followed by a quantifier. Unlike in some other BNF derivatives, the quantifier is _required_. The available quantifiers follow (`x` and `y` represent non-negative decimal integers):

* `?` – 0 or 1
* `*` – 0 or more
* `+` – 1 or more
* `[x]` – Exactly `x`
* `[x,]` – `x` or more
* `[x,y]` – Between `x` and `y` inclusive

```cbnf
// Quantification examples:
<decimal-number> ::= {"-"}? {<digit>}+ {"." {<digit>}+}? ;
<markdown-header> ::= "#"[1,6] " " <markdown-text> ;
```

#### Description

A **description** term consists of some text between `(?` and `?)`. In a usual cBNF syntax, the text is simply natural language describing what sequences satisfy the term. Since description terms are effectively an escape hatch from cBNF, they should be used with caution. In a cBNF syntax interpreted by software, _description terms may be prohibited or restricted to certain predefined values_.

```cbnf
// Description example:
<whitespace> ::= {(? any whitespace character ?)}+ ;
```

#### Exception

An **exception** is a special type of expression consisting of exactly two terms separated by a minus sign. It represents a set difference, and is satisfied by any sequence that satisfies the first term but does not satisfy the second term. _An exception is an expression, not a term, and must be parenthesized when used in a context expecting a term._

```cbnf
// Exception examples:
<non-zero-digit> ::= <digit> - "0" ;
<non-zero-integer> ::= {"-"}? ({<digit>}+ - {"0"}+) ;
```

### Extension

A cBNF syntax may extend a preceding cBNF syntax in the following manners:

* A rule can define a new identifer.
* A rule can redefine an identifier using the `::=` assignment operator again, overriding its previous definition.
* A rule can append alternatives to an identifier's definition using the `::=|` assignment operator.

```cbnf
// Extension demonstration:

<fruit> ::= "apple" | "orange" ;
<veggie> ::= "carrot" | "lettuce" ;

// -------------------------------------

<fruit> ::= "lemon" ;
<veggie> ::=| "broccoli" ;
<name> ::= "Maddie" ;
```

```cbnf
// The above syntax is equivalent to the following:

<fruit> ::= "lemon" ;
<veggie> ::= "carrot" | "lettuce" | "broccoli";
<name> ::= "Maddie" ;
```

If used to assign to an identifier that has not yet been defined, `::=|` is equivalent to `::=`.
