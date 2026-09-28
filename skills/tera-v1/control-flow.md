# Tera v1 Control Flow & Inheritance Reference

This document covers conditionals, loops, loop controls, including sub-templates, defining macros, and template inheritance.

## Conditionals (`if`, `elif`, `else`)

Conditionals are fully supported and are similar to Python:

```jinja2
{% if price < 10 or always_show %}
   Price is {{ price }}.
{% elif price > 1000 and not rich %}
   That's expensive!
{% else %}
    N/A
{% endif %}
```

Undefined variables are considered falsy. You can test for the presence of a variable in the current context by writing:

```jinja2
{% if my_var %}
    {{ my_var }}
{% else %}
    Sorry, my_var isn't defined.
{% endif %}
```
Every `if` statement must end with an `endif` tag.

---

## Loops (`for`)

Loop over items in an array:
```jinja2
{% for product in products %}
  {{ loop.index }}. {{ product.name }}
{% endfor %}
```

Or on characters of a string:
```jinja2
{% for letter in name %}
  {% if loop.index % 2 == 0 %}
    <span style="color:red">{{ letter }}</span>
  {% else %}
    <span style="color:blue">{{ letter }}</span>
  {% endif %}
{% endfor %}
```

### Loop Variables
Special variables are available inside for loops:
- `loop.index`: current iteration (1-indexed)
- `loop.index0`: current iteration (0-indexed)
- `loop.first`: whether this is the first iteration
- `loop.last`: whether this is the last iteration

Every `for` statement must end with an `endfor` tag.

### Iterating over Maps and Structs
```jinja2
{% for key, value in products %}
  {{ key }}. {{ value.name }}
{% endfor %}
```

### Filters on Containers
You can apply filters to the container being iterated:
```jinja2
{% for product in products | reverse %}
  {{ loop.index }}. {{ product.name }}
{% endfor %}
```

### Iterating on Array Literals
```jinja2
{% for a in [1,2,3] %}
  {{ a }}
{% endfor %}
```

### Default Empty Body (`else`)
You can set a default body to be rendered when the container is empty:
```jinja2
{% for product in products %}
  {{ loop.index }}. {{ product.name }}
{% else %}
  No products.
{% endfor %}
```

---

## Loop Controls (`break` & `continue`)

Within a loop, `break` and `continue` may be used to control iteration.

Stop iterating when `target_id` is reached:
```jinja2
{% for product in products %}
  {% if product.id == target_id %}{% break %}{% endif %}
  {{ loop.index }}. {{ product.name }}
{% endfor %}
```

Skip even-numbered items:
```jinja2
{% for product in products %}
  {% if loop.index is even %}{% continue %}{% endif %}
  {{ loop.index }}. {{ product.name }}
{% endfor %}
```

---

## Includes

Insert another template to be rendered using the current context with the `include` tag:

```jinja2
{% include "included.html" %}
```

The template path must be a static string. (Dynamic paths like `{% include "partials/" ~ name ~ ".html" %}` are invalid).

### Handling Missing Templates
You can ignore a missing template using `ignore missing`:
```jinja2
{% include "header.html" ignore missing %}
```

Or provide a list of fallback templates; the first one that exists will be included:
```jinja2
{% include ["custom/header.html", "header.html"] %}
{% include ["special_sidebar.html", "sidebar.html"] ignore missing %}
```

*Note: You cannot mix inheritance (`extends`) and `include`s in the same files (i.e. you cannot have inheritance inside included files).*

---

## Macros

Macros are reusable template components, similar to functions.

Defining a macro:
```jinja2
{% macro input(label, type="text") %}
    <label>
        {{ label }}
        <input type="{{ type }}" />
    </label>
{% endmacro input %}
```

Importing and calling a macro from a separate file:
```jinja2
{% import "macros.html" as macros %}

{{ macros::input(label="Name", type="text") }}
```
*Note: Macros require keyword arguments. Use the `self` namespace when calling a macro defined in the same file.*

Recursive macros are supported (ensure they have a base case to terminate):
```jinja2
{% macro factorial(n) %}
  {% if n > 1 %}{{ n }} - {{ self::factorial(n=n-1) }}{% else %}1{% endif %}
{% endmacro factorial %}
```

---

## Inheritance

Define a base layout and extend it in child templates using blocks.

### Base Template (`base.html`)
```jinja2
<!DOCTYPE html>
<html lang="en">
<head>
    {% block head %}
    <link rel="stylesheet" href="style.css" />
    <title>{% block title %}{% endblock title %} - My Webpage</title>
    {% endblock head %}
</head>
<body>
    <div id="content">{% block content %}{% endblock content %}</div>
    <div id="footer">
        {% block footer %}
        &copy; Copyright 2008
        {% endblock footer %}
    </div>
</body>
</html>
```

### Child Template
```jinja2
{% extends "base.html" %}

{% block title %}Index{% endblock title %}

{% block head %}
    {{ super() }}
    <style type="text/css">
        .important { color: #336699; }
    </style>
{% endblock head %}

{% block content %}
    <h1>Index</h1>
    <p class="important">
      Welcome to my homepage.
    </p>
{% endblock content %}
```

The `{{ super() }}` call tells Tera to render the parent block's content at that position.
In a child template, any content outside of a block will be ignored.
