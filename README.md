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

Get back to your terminal via `ctrl+c` or execute `exit`.

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
⏂ > pipe(pluck('keyword'), slice(-3))($x['dataset'])
```

## Non-interactive mode

Pipe data in:

```bash
> echo '[{"a":1,"b":3},{"a":2,"b":4}]' | philo 'pluck(a)'
[{"a":1},{"a":2}]
```

Or send via input redirection:

```bash
> curl https://newton.now.sh/factor/x^2-1 > /tmp/math
> cat /tmp/math                                                           
{"operation":"factor","expression":"x^2-1","result":"(x - 1) (x + 1)"}
> philo 'pipe(values, spread(fn($a, $b, $c) => [$c, $b, $a]))' < /tmp/math
["(x - 1) (x + 1)","x^2-1","factor"]
```

...more soon!
