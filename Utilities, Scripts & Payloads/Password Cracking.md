

## Table of Contents:

- [ ] [[#Default Credentials]]
- [ ] [[#Online Databases - Hash Cracking]]
- [ ] [[#Making Custom Wordlists]]
- [ ] [[#Making Wordlist Mutations]]
- [ ] [[#Wordlist Optimisation]]

---

## Default Credentials:

Before attempting login bruteforcing or any password-based attacks, you should always check for **password re-use and default credentials usage**.

1. <https://github.com/danielmiessler/SecLists/blob/master/Passwords/Default-Credentials/default-passwords.csv>
2. <https://github.com/Dormidera/WordList-Compendium>
3. <https://datarecovery.com/rd/default-passwords/>
4. <https://bizuns.com/default-passwords-list>
5. <https://www.cirt.net/passwords>

---

## Online Databases - Hash Cracking:

1. <https://crackstation.net/>
2. <https://www.cmd5.org/>
3. <https://md5decrypt.net/>
4. <https://www.md5online.org/md5-decrypt.html>

---

## Making Custom Wordlists

### [Cupp](https://github.com/Mebus/cupp.git)

```sh
cupp -i
### Interactively create a custom Password Wordlist using cupp

python3 cupp.py -w wordlist.txt
### Use existing wordlist with profiling
```

### Username Anarchy

```sh
./username-anarchy Bill Gates > wordlist.txt
### Generate usernames list starting from name and surname
```

### Crunch

```bash
# Basic syntax
crunch <min-len> <max-len> [charset] -o <output>

# Generate 4-6 character passwords
crunch 4 6 -o wordlist.txt

# With specific characters
crunch 8 8 abcdefghijklmnopqrstuvwxyz0123456789 -o wordlist.txt

# With pattern
crunch 6 6 -t @@%%^^ -o wordlist.txt
# @ = lowercase, , = uppercase, % = number, ^ = special

# Custom charset
crunch 4 4 -f /usr/share/crunch/charset.lst mixalpha-numeric -o wordlist.txt

# Pipe to hashcat
crunch 8 8 0123456789 | hashcat -m 0 hash.txt
```


---

## Making Wordlist Mutations

### [CeWL](https://github.com/digininja/CeWL)

``` sh 
cewl https://example.idk -d 4 -m 6 --lowercase -w wordlist.txt
### Based on website keywords, -d: depth spider, -m: miminum length

cewl https://target.com -e -w wordlist.txt
### Include Emails with -e 
```

### Hashcat

| **Function** | **Description**                                  |
| ------------ | ------------------------------------------------ |
| `:`          | Do nothing                                       |
| `l`          | Lowercase all letters                            |
| `u`          | Uppercase all letters                            |
| `c`          | Capitalize the first letter and lowercase others |
| `sXY`        | Replace all instances of X with Y                |
| `$!`         | Add the exclamation character at the end         |
Each rule is written on a new line and determines how a given word should be transformed.

```sh
hashcat --force password.list -r custom.rule --stdout | sort -u > new.list
```

---

## Wordlist Optimisation

```sh
sed -ri '/^.{,7}$/d' wordlist.txt
### Remove passwords shorter than 8 characters from wordlist

sed -ri '/[0-9]+/!d' wordlist.txt
### Remove passwords without numbers from wordlist

sort -u wordlist.txt -o wordlist.txt
### Sort and remove duplicates

sed -i '/^$/d' wordlist.txt
# Remove empty lines

awk 'length >= 4' wordlist.txt > filtered.txt
# Remove lines shorter than 4 chars

tr '[:upper:]' '[:lower:]' < wordlist.txt > lowercase.txt
# Convert to lowercase

wc -l wordlist.txt
# Count lines
```


---
