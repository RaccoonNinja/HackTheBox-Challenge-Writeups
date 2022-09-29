This is supposed to be simple but I was distracted by red herrings.

<details>
  <summary> Hint 1: </summary>
  
  What are the possible vectors? (If you haven't downloaded the source code, download it)
</details>
<details>
  <summary> Hint 2: </summary>
  
  There are at least 2 ways you can leak the flag if you have RCE, which one is easier? What stops you from achieving what you want? How would you *bypass* it?
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  
  There is only 1 page with 1 vector: the `format` query parameter. It is passed through `addslashes` and then supplied to this statement:
```php
eval('$time = date("' . $this->format . '", strtotime("' . $this->prediction . '"));');
```

`addslashes` will add slashes to `'`, `"`, `\` and `NULL`. On the surface it may seem we cannot escape the double quote context, however we have a very good resource [here](https://0xalwayslucky.gitbook.io/cybersecstack/web-application-security/php).

The caveat is that functions within `${}` are executed by php to get a variable name, **including all global variables like `$_GET`**.

I simplified a bit to get `?format=${eval($_GET[1])}&1=system('cp /flag* /www/static/flag.txt')`.
</details>
