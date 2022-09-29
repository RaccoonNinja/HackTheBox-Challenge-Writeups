## Skills involved: Chaining vulnerabilities

I had the core ideas but the implementation was frustrating because of a number of reasons. After working on the problem for weeks I decided to take a look at how someone else had implemented the idea. I'll keep the hints but this writeup will be more on the mistakes I made instead of the actual payloads etc.

<details>
  <summary> Hint 1: </summary>
  
  There are 2 possible attack vectors that can be found out by reading the source code. What are they and how can you use them?
</details>
<details>
  <summary> Hint 2: </summary>
  
  Are there any vulnerable packages? What will the vulnerability be given the version? You may come up with many vulnerabilities but only 1 will be useful.
</details>
<details>
  <summary> Hint 3: </summary>
  
  What is the vulnerability called? You really need to make sure you're searching examples and code with the right terms as there are other vulnerabilities with similar names.
</details>
<details>
  <summary> Hint 4: </summary>
  
  The exploit requires very high accuracy.
  - How can you reliably provide payloads and nullify any parts not needed?
  - Do you want to do it manually?
  - How can you debug the internet traffic locally to find out any mistakes?
</details>
<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  
  To start off, there are 2 blatant vulnerabilities:
  
  - SQL injection at `/register`, but it can only be accessed locally at server-side (`req.socket.remoteAddress` can't be spoofed).
  - Server-Side Request Forgery at `api/weather`, which we can use to access `/register`, but it's `GET` instead of the `POST` required
  
  Upon research, I found out about request ~~smuggling~~ splitting in [NodeJS before 8.14](https://nvd.nist.gov/vuln/detail/CVE-2018-12116). I mixed up request splitting with request smuggling and that affected my search tremendously.
  
  Researching the final vulnerability took some time and I couldn't really find any useful code. In hindsight there's [this good read for CVE-2018-12116](https://www.rfk.id.au/blog/entry/security-bugs-ssrf-via-request-splitting/), which uses the term 'request-splitting' instead of 'request-smuggling'
  
  Being not very familiar with the HTTP protocol, I just tried in Burp and that was my second mistake - doing it manually is very prone to error.
  
  I was frequently greeted by 2 errors:
  - `Parse Error` (unable to handle trailing `\r\nHost:***\r\nConnection: close`)
  - TypeError: Cannot read property 'replace' of undefined (`req.socket.remoteAddress` was undefined)
  
  There are other unpredictable factors which made the experience rather frustrating but in hindsight I should have inspected the packets with Wireshark. The opaqueness was unnecessary.
  
  Finally for the SQLite part, I wasn't really familiar with SQLite and here were the things I tried. 
  - Tried multiple statements - didn't work.
  - Tried `ON CONFLICT DO UPDATE SET...` - gave parsing error despite [the official spec allowing it](https://www.sqlite.org/syntax/upsert-clause.html)
  - Tried `ON CONFLICT(username) DO UPDATE SET password='a' WHERE username='admin'` but I was not inserting admin (I tried to force collision on another username).
  
  That was the place where I cracked, after trying for weeks. [CaveTownie](https://cavetownie.github.io/posts/htb_weatherapp/) has written a very detailed write-up. Here are some keypoints:
  - They use Python for converting `\n\r ` as well as sending the request, which is much more robust than changing payload in Burp repeater.
  - They are also more clear about the payload due to better research
  - They used `\r\n\r\nGET /?r=` to clear all other keys in the original API call, while handling the trailing `\r\nHost:***\r\nConnection: close` correctly.
  
  ### Lessons learnt:
  - Planning and drafting the payload is much better than trial-and-error.
  - Effort should be made to inpsect the internet traffic instead of keeping it a black box.
  - Wrong search terms mean wrong search result. **The one who discovered and initially documented CVE-2018-12116 did not call it by the name CVE-2018-12116**.
  - Request smuggling and request splitting are 2 different vulnerabilities. Smuggling abuses different parsing by frontend server and backend server while splitting abuses http clients' inability to block harmful inputs.
  - Sometimes documentation isn't enough for understanding software behaviors. For SQLite:
    - some documented features did not work
    - the documentation weren't very clear about *WHERE clause in ON CONFLICT is applicable only to conflicting items*
  
