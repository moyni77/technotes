## returns the number of vowels in string

```python
def getCount(inputStr):
    return sum(1 for let in inputStr if let in "aeiouAEIOU")
```

## remote character srfom string

```python
def disemvowel(string):
    return "".join(c for c in string if c not in "aeiouAEIOU")

disemvowel("hello world")
```

## return true if pin is 4/6 characters and is digits only

```python
def validate_pin(pin):
    return len(pin) in (4, 6) and pin.isdigit()
```
