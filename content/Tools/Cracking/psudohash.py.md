
```sh
./psudohash.py -w microsoft --common-paddings-after -y 2020-2025

# When it comes to people, i think we all have (more or less) set passwords using a mutation of one or more words that mean something to us e.g., our name or wife/kid/pet/band names, sticking the year we were born at the end or maybe a super secure padding like "!@#". Well, guess what?
./psudohash.py -w 'david,maria,lassie,metallica' --common-paddings-after --common-paddings-before -y 1990,2022
```
### Usage Examples
```shell
# No multi‐word (singletons only)
./psudohash.py -w foo,bar,baz -cpa
# → foo, bar, baz

# In‐order joins (-i, up to 2 words by default)
./psudohash.py -w foo,bar,baz -i
# → foo, bar, baz, foobar, foobaz, barbaz

# All‐order combinations (-c, up to 2 words by default)
./psudohash.py -w foo,bar,baz -c
# → foo, bar, baz, foobar, foobaz, barfoo, barbaz, bazfoo, bazbar

# Change separator between joined words
./psudohash.py -w foo,bar,baz -i --sep "_"
# → foo, bar, baz, foo_bar, foo_baz, bar_baz

# Length Filtering (`--minlen`/`--maxlen`)
 ./psudohash.py -w apple,banana -i --minlen 10
 # Warning: exact size cannot be determined because of length filters.
 # Example final outputs might include “applebanana” (11 chars), “bananaapple” (11 chars).
```
 you can Combine up to 3 words (instead of default 2).

### Change this in code
The script implements the following character substitution schema. You can add/modify character substitution mappings by editing the `transformations` list in `psudohash.py` and following the data structure presented below (default):
```
transformations = [
	{'a' : ['@', '4']},
	{'b' : '8'},
	{'e' : '3'},
	{'g' : ['9', '6']},
	{'i' : ['1', '!']},
	{'o' : '0'},
	{'s' : ['$', '5']},
	{'t' : '7'}
]
```

When setting passwords, I believe it's pretty standard to add a sequence of characters before and/or after the main passphrase to make it "stronger". For example, one may set a password "dragon" and add a value like "!!!" or "!@#" at the end, resulting in "dragon!!!", "dragon!@#", etc. Psudohash reads such values from `common_padding_values.txt` and uses them to mutate the provided keywords by appending them before (`-cpb`) or after (`-cpa`) each generated keyword variation. You can modify it as you see fit.

When appending a year value to a mutated keyword, psudohash will do so by utilizing various seperators. by default, it will use the following seperators which you can modify by editing the `year_seperators` list, For example, if the given keyword is "amazon" and option `-y 2023` was used, the output will include "amazon2023", "amazon_2023", "amazon-2023", "amazon@2023", "amazon23", "amazon_23", "amazon-23", "amazon@23".
```
year_seperators = ['', '_', '-', '@']
```

## Usage Tips
1. Combining options `--years` and `--append-numbering` with a `--numbering-limit` ≥ last two digits of any year input, will most likely produce duplicate words because of the mutation patterns implemented by the tool.
2. If you add custom padding values and/or modify the predefined common padding values in the source code, in combination with multiple optional parameters, there is a small chance of duplicate words occurring. psudohash includes word filtering controls but for speed's sake, those are limited.
3. When using `--minlen` or `--maxlen`, the script cannot pre-calculate the exact word count; you’ll see a “exact size cannot be determined” warning and the size without this filter will be calculated, the final size will be smaller.