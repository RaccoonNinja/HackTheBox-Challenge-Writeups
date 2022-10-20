## Skills involved: Authentication Bypass

The technique is common enough and with some research, not hard to understand.

<details>
  <summary>Hint 1: </summary>
  
  What do you want to do? What's the obstacle here?
</details>
<details>
  <summary>Hint 2: </summary>
  
  A certain video by [PwnFunction](https://www.youtube.com/c/PwnFunction) may help your understanding to first part of the solution.
</details>
<details>
  <summary>Hint 3: </summary>
  
  You don't need to implement the tool yourself once you know what to do.
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  
  At first, I was distracted and forgot this is a *crypto* challenge. I regained focus shortly afterwards. 
  The only crypto-related functionality is the `verify_login`, and passing it will give us the flag directly:
  
  ``` py
  def verify_login(func):
    @functools.wraps(func)
    def wrapped(*args, **kwargs):
        if not session.validate_login(request.cookies.get('login_info', '')):
            return redirect(url_for('web.login', error='You are not a logged in member'))
        return func(*args, **kwargs)
    return wrapped
  ```
  
  **models.py**:
  ``` py
  import hashlib, base64, urlparse, os
secret = os.urandom(16)
class session:
    @staticmethod
    def create(username, logged_in='True'):
        if username == 'guest':
            logged_in = 'False'
        hashing_input = 'username={}&isLoggedIn={}'.format(username, logged_in)
        crypto_segment = signature.create(hashing_input)
        
        return '{}.{}'.format(signature.encode(hashing_input), crypto_segment)
    
    @staticmethod
    def validate_login(payload):
        hashing_input, crypto_segment = payload.split('.')
        if signature.integrity(hashing_input, crypto_segment):
            return {
                k: v[-1] for k, v in urlparse.parse_qs(signature.decode(hashing_input)).items()
            }.get('isLoggedIn', '') == 'True'
        
        return False
class signature:
    @staticmethod
    def encode(data):
        return base64.b64encode(data)
    @staticmethod
    def decode(data):
        return base64.b64decode(data)
    @staticmethod
    def create(payload, secret=secret):
        return signature.encode(hashlib.sha512(secret + payload).hexdigest())
    
    @staticmethod
    def integrity(hashing_input, crypto_segment):
        return signature.create(signature.decode(hashing_input)) == crypto_segment
  ```
  
  After looking up some common vulnerabilities associated with hash functions in general, I learnt about [Length Extension Attacks](https://en.wikipedia.org/wiki/Length_extension_attack). I can say this challenge is a textbook example.
  
  The exploit is quite well-documented and the [explanation at Synopsys](https://www.whitehatsec.com/blog/hash-length-extension-attacks/) is comprehensive, yet it didn't include any actual code. It took me some time to realize that I cannot rewrite the hash state in Python.
  
  I eventually used [Ron Bowes's Hash Extender](https://github.com/iagox86/hash_extender). Having installed the `libssl-dev` package, I can `make` and use the tool to get the new hash for the appended string.
  
  ## Footnotes:
  1. For web developers: as mentioned by other posts, by simply hashing twice (i.e. **HMAC**) the issue is resolved.
  1. Alternatively other algorithms like the truncated variants SHA-512/256 or SHA-384 can be used. Also, all SHA-3 algorithms are quite resistant to length extension attacks.
  
</details>
