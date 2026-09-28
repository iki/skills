# Tera v1 Built-in Filters, Tests, and Functions Reference

This document lists all built-in filters, tests, and functions available in the Tera template engine.

---

## Built-in Filters

### lower
Converts a string to lowercase.

### upper
Converts a string to uppercase.

### wordcount
Returns the number of words in a string.

### capitalize
Returns the string with the first character capitalized and all other characters lowercased.

### replace
Replaces all occurrences of a string with another. Takes two named arguments: `from` and `to`.
- Example: `{{ name | replace(from="Robert", to="Bob") }}`

### addslashes
Adds backslashes before quote characters.
- Example: `{{ value | addslashes }}` (If value is `"I'm using Tera"`, output is `"I\'m using Tera"`)

### slugify
Transforms a string into ASCII, lowercases it, trims it, converts spaces to hyphens, and removes non-alphanumeric characters (except hyphens).
- Example: `{{ value | slugify }}` (If value is `"-Hello world! "`, output is `"hello-world"`)

### title
Capitalizes each word in a string.
- Example: `{{ value | title }}`

### trim
Removes leading and trailing whitespace from a string.

### trim_start
Removes leading whitespace.

### trim_end
Removes trailing whitespace.

### trim_start_matches
Removes leading characters matching the given pattern.
- Example: `{{ value | trim_start_matches(pat="//") }}` (If value is `"//a/b/c//"`, output is `"a/b/c//"`)

### trim_end_matches
Removes trailing characters matching the given pattern.
- Example: `{{ value | trim_end_matches(pat="//") }}` (If value is `"//a/b/c//"`, output is `"//a/b/c"`)

### truncate
Truncates a string to a specified `length`. Appends `...` by default, which can be customized via the `end` argument.
- Example: `{{ value | truncate(length=10) }}`
- Example: `{{ value | truncate(length=10, end="") }}`

### linebreaksbr
Replaces line breaks (`\n` or `\r\n`) with HTML `<br>` tags. Note: if auto-escaping is active, you may need to chain the `safe` filter afterwards.
- Example: `{{ value | linebreaksbr | safe }}`

### spaceless
Removes spaces and line breaks between HTML tags.
- Example: `{{ value | spaceless }}`

### indent
Indents a string by injecting a prefix (default 4 spaces) at the start of each line.
- Arguments: `prefix` (string), `first` (bool, default false, whether to indent the first line), `blank` (bool, default false, whether to indent empty lines).

### striptags
Tries to strip HTML tags from the input.
- Example: `{{ value | striptags }}`

### first
Returns the first element of an array. Returns an empty string if the array is empty.

### last
Returns the last element of an array. Returns an empty string if the array is empty.

### nth
Returns the nth element of an array (0-indexed). Takes a required `n` argument.
- Example: `{{ value | nth(n=2) }}`

### join
Joins an array of strings with a separator string.
- Example: `{{ value | join(sep=" // ") }}`

### length
Returns the length of an array, object, or string.

### reverse
Returns a reversed string or array.

### sort
Sorts an array into ascending order. If sorting structs or tuples, use the `attribute` argument to specify the path to sort by.
- Example: `{{ people | sort(attribute="age") }}`
- Example: `{{ people | sort(attribute="name.1") }}`

### unique
Removes duplicate items from an array. Can filter by an inner attribute via the `attribute` argument. Case sensitivity can be toggled using `case_sensitive` (default false).
- Example: `{{ people | unique(attribute="age") }}`

### slice
Slices an array from `start` (inclusive, default 0) to `end` (exclusive, default array length). Supports negative indices (e.g. -1 for the last element).
- Example: `{{ my_arr | slice(start=1, end=5) }}`

### group_by
Groups an array into a map where keys are stringified values of the given `attribute`.
- Example:
  ```jinja2
  {% for name, author_posts in posts | group_by(attribute="author.name") %}
      {{ name }}
      {% for post in author_posts %}
          {{ post.year }}: {{ post.content }}
      {% endfor %}
  {% endfor %}
  ```

### filter
Filters an array, returning only elements where the given `attribute` equals `value`.
- Example: `{{ posts | filter(attribute="draft", value=true) }}`

### map
Extracts a specific attribute from each object in an array.
- Example: `{{ people | map(attribute="age") }}`

### concat
Appends values or another array to an array.
- Example: `{{ posts | concat(with=drafts) }}`

### urlencode
Percent-encodes all characters in a string except forward slash (`/`).
- Example: `{{ value | urlencode }}`

### urlencode_strict
Percent-encodes all non-alphanumeric characters in a string, including forward slashes (`/`).
- Example: `{{ value | urlencode_strict }}`

### abs
Returns the absolute value of a number.

### pluralize
Returns a plural suffix (default `s`) if the value is not ±1, otherwise returns the singular suffix (default empty).
- Example: `You have {{ num }} message{{ num | pluralize }}`
- Example: `{{ num }} categor{{ num | pluralize(singular="y", plural="ies") }}`

### round
Rounds a number. Arguments: `method` (default `"common"`, alternatives `"ceil"`, `"floor"`), `precision` (default `0`).
- Example: `{{ num | round(method="ceil", precision=2) }}`

### filesizeformat
Formats an integer byte count into a human-readable file size (e.g., `'110 MB'`).
- Example: `{{ num | filesizeformat }}`

### date
Parses a timestamp or ISO 8601 string into a formatted date/time string (defaults to `YYYY-MM-DD`). Uses chrono format specifiers.
- Arguments: `format` (string), `timezone` (string, e.g. `"Europe/Berlin"`), `locale` (string, e.g. `"fr_FR"`).
- Example: `{{ ts | date(format="%Y-%m-%d %H:%M") }}`
- Example: `{{ "2019-09-19T13:18:48.731Z" | date(timezone="America/New_York") }}`

### escape
Escapes HTML special characters: `&`, `<`, `>`, `"`, `'`, `/`.

### escape_xml
Escapes XML special characters: `&`, `<`, `>`, `"`, `'`.

### safe
Marks a string as safe to prevent HTML escaping. Must be the last filter in the chain.
- Example: `{{ html_content | safe }}`

### get
Accesses a value from an object when the key is not a valid Tera identifier.
- Example: `{{ sections | get(key="posts/content", default="Fallback") }}`

### split
Splits a string into an array of strings by a separator pattern.
- Example: `{{ path | split(pat="/") }}`

### int
Converts a value to an integer. Supports `default` fallback and `base` (e.g. 2, 8, 16).

### float
Converts a value to a float. Supports `default` fallback.

### json_encode
Encodes any value to a JSON string. Can be formatted using the `pretty=true` argument.
- Example: `{{ value | json_encode(pretty=true) | safe }}`

### as_str
Returns a string representation of the given value.

### default
Returns the default value if the variable is not present in the context.
- Example: `{{ value | default(value=1) }}`
- Note: This only checks for existence in the context. If a variable is defined as an empty string or `0`, it is considered defined, and the default is not used.

---

## Built-in Tests

### defined
Returns true if the variable is defined.

### undefined
Returns true if the variable is undefined.

### odd
Returns true if the variable is an odd number.

### even
Returns true if the variable is an even number.

### string
Returns true if the variable is a string.

### number
Returns true if the variable is a number.

### divisibleby
Returns true if the variable is divisible by the argument.
- Example: `{% if rating is divisibleby(2) %}`

### iterable
Returns true if the variable is an array/tuple or object.

### object
Returns true if the variable is an object.

### starting_with
Returns true if the variable is a string starting with the argument.
- Example: `{% if path is starting_with("x/") %}`

### ending_with
Returns true if the variable is a string ending with the argument.

### containing
Returns true if the variable contains the argument. Works on strings (substring), arrays (member check), and maps (key check).
- Example: `{% if username is containing("admin") %}`

### matching
Returns true if the variable matches the regex pattern.
- Example: `{% if name is matching("^[Qq]ueen") %}`

---

## Built-in Functions

### range
Returns an array of integers.
- Arguments: `end` (required, exclusive), `start` (default 0), `step_by` (default 1).
- Example: `{% for i in range(start=1, end=10) %}`

### now
Returns the current date/time.
- Arguments: `timestamp` (bool, whether to return integer seconds), `utc` (bool, whether to use UTC).
- Example: `{{ now() | date(format="%Y") }}`

### throw
Aborts rendering with an error message.
- Example: `{{ throw(message="Invalid condition met") }}`

### get_random
Returns a random integer in the range `[start, end)`.
- Arguments: `start` (default 0, inclusive), `end` (required, exclusive).

### get_env
Returns the value of an environment variable. Errors if not found unless a `default` is provided.
- Arguments: `name` (required), `default` (optional).
- Example: `{{ get_env(name="DATABASE_URL", default="sqlite://") }}`
