---
name: tera-v1
description: >
  Reference for Tera v1 template engine syntax, built-in filters, tests, functions, control flow,
  inheritance, macros, and the `tera` command-line interface (CLI). Use this skill when working with
  Tera templates (files containing Jinja2-like syntax such as `{{`, `{%`, or `{#`), when rendering
  templates, or when using the `tera` CLI.
---

# Tera Template Engine Skill Documentation

This skill provides comprehensive documentation and guides for writing Tera v1 templates and using the `tera` CLI. It focuses on template syntax, variables, expressions, control flow, and built-in filters/functions.

## When to Use This Skill

Use this skill when you need to:
*   Write or modify Tera templates (`.md`, `.html`, etc.).
*   Understand template syntax, delimiters, and whitespace control.
*   Explore filters, tests, functions, and control structures in templates.
*   Implement template inheritance and macros.
*   Use the `tera` command-line utility to generate output files from templates and data sources (JSON, TOML, YAML).

## ⚡ Quick Reference

Here are some practical examples of common tasks and core concepts in Tera.

### 1. Using Filters in Templates

Filters modify variables. They are chained using the pipe `|` symbol and can accept named arguments.

```jinja2
{# Example: Convert 'name' to lowercase and then replace "Dr" with "Doctor" #}
{{ name | lower | replace(from="Dr", to="Doctor") }}

{# Example: Truncate a long string to 20 characters, adding "..." #}
{{ long_text | truncate(length=20) }}
```

### 2. Conditional Logic (`if`/`elif`/`else`)

Tera templates support standard conditional statements for dynamic content generation.

```jinja2
{% if user.is_admin %}
    <p>Welcome, Administrator!</p>
{% elif user.is_logged_in %}
    <p>Hello, {{ user.name }}!</p>
{% else %}
    <p>Please log in.</p>
{% endif %}
```

### 3. Looping Over Data (`for`)

Iterate over arrays, maps, or even strings. Special loop variables like `loop.index` are available.

```jinja2
{# Loop over a list of products #}
<ul>
{% for product in products %}
  <li>{{ loop.index }}. {{ product.name }} - ${{ product.price }}</li>
{% else %}
  <li>No products available.</li> {# Rendered if 'products' is empty #}
{% endfor %}
</ul>

{# Loop over key-value pairs in an object/map #}
<dl>
{% for key, value in settings %}
  <dt>{{ key }}</dt><dd>{{ value }}</dd>
{% endfor %}
</dl>
```

### 4. Defining and Using Macros

Macros are reusable blocks of template code, similar to functions or components.

```jinja2
{# macros.html: Define a simple form input macro #}
{% macro input_field(id, label, type="text", value="") %}
    <label for="{{ id }}">{{ label }}:</label>
    <input type="{{ type }}" id="{{ id }}" name="{{ id }}" value="{{ value }}" />
{% endmacro input_field %}

{# main_template.html: Import and use the macro #}
{% import "macros.html" as forms %}

<h1>User Profile</h1>
{{ forms::input_field(id="username", label="Username", type="text", value=user.name) }}
{{ forms::input_field(id="email", label="Email", type="email", value=user.email) }}
```

### 5. Template Inheritance (`extends` and `block`)

Define a base layout and extend it in child templates to reuse common structure. The `super()` call includes content from the parent block.

```jinja2
{# base.html: The base layout with defined blocks #}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock title %}</title>
</head>
<body>
    <header><h1>{% block header_content %}Default Header{% endblock header_content %}</h1></header>
    <main>{% block content %}<p>No content provided.</p>{% endblock content %}</main>
    <footer>&copy; 2026</footer>
</body>
</html>

{# about.html: A child template extending base.html #}
{% extends "base.html" %}

{% block title %}About Us - {{ super() }}{% endblock title %}

{% block header_content %}About Our Company{% endblock header_content %}

{% block content %}
    <h2>Our Story</h2>
    <p>This is the content specific to the about page.</p>
{% endblock content %}
```

### 6. Using the Tera CLI Utility

Render templates from context files (JSON, TOML, YAML) or stdin:

```bash
# Render using a JSON context file
tera --template my_template.md context.json

# Render using context data from stdin
echo '{"name": "World"}' | tera --template hello.md --stdin
```

---

## 📖 Project Documentation

The documentation has been split into modular files for quick access:

-   **[CLI Reference](./cli.md)**: Details on running the `tera` command-line utility.
-   **[Template Syntax & Data](./syntax.md)**: Variables, literals, basic math, comparisons, logical operations, assignments, and expression syntax.
-   **[Control Flow & Inheritance](./control-flow.md)**: Conditionals, loops, loop controls (`break`/`continue`), imports, macros, and template inheritance blocks.
-   **[Built-in Filters, Tests & Functions](./builtins.md)**: Exhaustive reference of all built-in filters (e.g., `replace`, `lower`, `date`, `sort`), tests (e.g., `defined`, `odd`), and functions (e.g., `range`, `now`).
