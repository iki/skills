# Tera v1 Template Syntax Reference

This document covers Tera template basics, delimiters, comments, data structures, expressions, and data manipulation.

## Introduction

### Tera Basics

A Tera template is a text file where variables and expressions get replaced with values when it is rendered. The syntax is based on Jinja2 and Django templates.

There are 3 kinds of delimiters and those cannot be changed:

- `{{` and `}}` for expressions
- `{%` and `%}` for statements
- `{#` and `#}` for comments

### Raw Blocks

Tera will consider all text inside the `raw` block as a string and won't try to render what's inside. Useful if you have text that contains Tera delimiters.

```jinja2
{% raw %}
  Hello {{ name }}
{% endraw %}
```
renders as `Hello {{ name }}`.

### Whitespace control

Use `{%-` to remove all whitespace before a statement and `-%}` to remove all whitespace after. This also works with expressions (`{{-` and `-}}`) and comments (`{#-` and `-#}`).

For example:
```jinja2
{% set my_var = 2 %}
{{ my_var }}
```
will have the following output (with an empty line):
```html

2
```

If we want to get rid of the empty line, we can write:
```jinja2
{% set my_var = 2 -%}
{{ my_var }}
```

### Comments

To comment out part of the template, wrap it in `{# #}`. Anything in between those tags will not be rendered.

```jinja2
{# A comment #}
```

---

## Data Structures

### Literals

Tera supports the following literals:

- **booleans**: `true` (or `True`) and `false` (or `False`)
- **integers**
- **floats**
- **strings**: text delimited by `""`, `''` or ` `` `
- **arrays**: a comma-separated list of literals and/or idents surrounded by `[` and `]` (trailing comma allowed)

### Variables

Variables are defined by the context given when rendering a template.

Render a variable using `{{ name }}`. Trying to access or render a variable that doesn't exist will result in an error.

A special variable `__tera_context` is available in every template to print the current context.

#### Dot notation
Attributes and properties can be accessed using the dot (`.`) like `{{ product.name }}`. Specific members of an array or tuple are accessed by using the `.i` notation, where `i` is a zero-based index. In dot notation, variables cannot be used after the dot.

#### Square bracket notation
A more powerful alternative is to use square brackets (`[ ]`). Variables can be rendered using `{{ product['name'] }}` or `{{ product["name"] }}`. If the item is not in quotes, it will be treated as a variable.
Assuming you have `product.name = "Fred"` and `my_field = "name"`, calling `{{ product[my_field] }}` resolves to `{{ product.name }}`.

---

## Expressions

Tera allows expressions almost everywhere.

### Math
You can do basic math using numbers. Math operations on non-numeric values will result in an error.
Available operators:
- `+`: addition (`{{ 1 + 1 }}` prints `2`)
- `-`: subtraction (`{{ 2 - 1 }}` prints `1`)
- `/`: division (`{{ 10 / 2 }}` prints `5`)
- `*`: multiplication (`{{ 5 * 2 }}` prints `10`)
- `%`: modulo (`{{ 2 % 2 }}` prints `0`)

Priority of operations (from lowest to highest):
1. `+` and `-`
2. `*`, `/`, and `%`

### Comparisons
- `==`: equal to
- `!=`: not equal to
- `>=`: greater than or equal to
- `<=`: less than or equal to
- `>`: greater than
- `<`: less than

### Logic
- `and`: true if both operands are true
- `or`: true if either operand is true
- `not`: negates an expression

### Concatenation
Concatenate strings, numbers, or identifiers using the `~` operator:
```jinja2
{{ "hello " ~ 'world' ~ `!` }}
{{ an_ident ~ " and a string" }}
```
An identifier resolving to something other than a string or a number will raise an error.

### `in` checking
Check whether a left side is contained in a right side using the `in` operator:
```jinja2
{{ some_var in [1, 2, 3] }}
{{ 'index' in page.path }}
{{ an_ident not in an_obj }}
```

---

## Manipulating Data

### Assignments
Assign values to variables during rendering. Assignments inside loops and macros are scoped to their context; assignments outside are set in the global context. Assignments in a `for` loop are only valid until the end of the current iteration.

```jinja2
{% set my_var = "hello" %}
{% set my_var = 1 + 4 %}
```

To assign a value in the global context from within a `for` loop, use `set_global`:
```jinja2
{% set_global my_var = "hello" %}
```
Outside of a `for` loop, `set_global` is identical to `set`.

### Filters
Modify variables using **filters** separated by a pipe symbol (`|`). Filters can have named arguments in parentheses and can be chained:
```jinja2
{{ name | lower | replace(from="doctor", to="Dr.") }}
```
Filters used in math operations have the lowest priority:
```jinja2
{{ a | length + 1 }} {# Evaluates length first, then adds 1 #}
```

#### Filter sections
Whole sections can be processed by filters using `{% filter name %}` and `{% endfilter %}`:
```jinja2
{% filter upper %}
    Hello
{% endfilter %}
```

### Tests
Tests are used in `if` blocks with the `is` keyword to check conditions:
```jinja2
{% if my_number is odd %}
  Odd
{% endif %}

{% if my_number is not odd %}
  Even
{% endif %}
```

### Functions
Functions are called to perform logic or retrieve values:
- In variable blocks: `{{ url_for(name="home") }}`
- In `for` loop containers: `{% for i in range(end=5) %}`
