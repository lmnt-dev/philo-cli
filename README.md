# ⍎ ρhilo-cli

Wrangle data from your terminal using extensible &amp; expressive philo syntax

Requires PHP 7.4+

## Install globally

If you're not using `cgr` for global installs, consider adopting it:

```bash
composer global require consolidation/cgr
```

Then install philo globally via `cgr`:

```bash
cgr --stability dev lmnt/philo-cli
```

## Interactive mode

Use the repl to experiment:

```bash
> philo
```

### Predicates & Types

Let's define a few types:

```php
⏂ > $Age = all(is_int, gt(0), lt(150))
⏂ > $Person = ['name' => is_string, 'age' => $Age, 'website' => maybe(is_url)]
⏂ > $Contacts = every($Person)
```

We can make some assertions now:

```php
⏂ > is($Person, ['name' => 'bob', 'age' => 50])
=> true
⏂ > is($Person, ['name' => 'bob', 'age' => 50, 'extra' => 1])
=> true
⏂ > is(strict($Person), ['name' => 'bob', 'age' => 50, 'extra' => 1])
=> false
⏂ > is($Person, ['name' => 'bob', 'age' => -1])
=> false
⏂ > is($Person, ['name' => 'bob', 'age' => 50, 'website' => 'google.com'])
=> false
⏂ > is($Person, ['name' => 'bob', 'age' => 50, 'website' => 'http://google.com'])
=> true
⏂ > is($Contacts, [['name' => 'bob', 'age' => 50]])
=> true
⏂ > is($Contacts, [['name' => 'bob', 'age' => 50], 'barb'])
=> false
```

On built-in types as well:

```php
⏂ > $x = new \ArrayObject()
⏂ > is(\ArrayObject::class, $x)
=> true
⏂ > is(\Countable::class, $x)
=> true
```

Get back to your terminal via `ctrl + c` or by typing `exit`.

### Loading data

Load data into philo from a local file or a URL.

Let's restart the repl with some json from a public endpoint:

```bash
> philo http://data.cdc.gov/data.json
```

Examine the datasource:

```php
⏂ > $x
```

What are the keywords for the last 3 entries?

```php
⏂ > pipe(to_array(pluck('keyword')), slice(-3))($x['dataset'])
```

## Non-interactive mode

Pipe data in:

```bash
> echo '[{"a":1,"b":3},{"a":2,"b":4}]' | philo 'to_array(pluck(a), true)'
[{"a":1},{"a":2}]
```

Or send via input redirection:

```bash
> curl https://newton.now.sh/factor/x^2-1 > /tmp/math
> cat /tmp/math
{"operation":"factor","expression":"x^2-1","result":"(x - 1) (x + 1)"}
> philo 'pipe(to_array(values), spread(fn($a, $b, $c) => [$c, $b, $a]))' < /tmp/math
["(x - 1) (x + 1)","x^2-1","factor"]
```

## Extending philo

Open `~/philo-lib.php` in your editor and define a new type:

```php
<?php
  
namespace philo;

const size = 'philo\size';

function size($x) {
    return is_string($x) ? strlen($x) : (is_countable($x) ? count($x) : null);
}

return [
    'Thing' => ['name' => pipe(size, gte(5))]
];
```

Save and you're done! Try it out:

```bash
> echo '[{"name": "abc"},{"name":"01234"}]' | \
> philo --lib ~/philo-lib.php 'to_array(map(match($Thing, "thing")))'
[null,"thing"]
```
