## Skills involved: Develop a love for coupons

(sorry this is the only non-spoily title)

This could be a difficult challenge for people less acquainted to web development, it's still not an easy challenge if you don't have the tools needed.

<details>
  <summary> Hint 1: </summary>
  
  How can you purchase the flag? What should you probably do?
</details>
<details>
  <summary> Hint 1b: </summary>
  
  If your jwt only occurs after using coupon, you probably cannot use the coupon again.
</details>
<details>
  <summary> Hint 2: </summary>
  
  Once you know what to do, the forum thread suggests some helpful tools. My network condition is not very good so I used [this tool suggested in forum](https://portswigger.net/bappstore/9abaa233088242e8be252cd4ff534988).
</details>
<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  Upon checking the source, I did not discover any obvious vulnerabilities like SQL injection, SSTI or prototype pollution. There were also no vulnerable packages.
  
  So we really have to use coupon**s** in order to purchase the flag. But when I tried it manually, of course it failed - the server did check for it.
  
  Nonetheless, if we try spamming the apply coupon endpoint within a very short period of time, *maybe* some will get through? This seemed to be the only way, but we need the jwt token, which is received after using the coupon, for resubmission. By the time we read the jwt token and resubmit, the deal has been sealed.
  
  Upon code inspection, I noticed that we can get a jwt token by *trying to purchase the flag* as well.
  
  Before continuing with the actual exploit I should first elaborate on the vulnerability:
  
  - In more complicated systems, sometimes changes have to be carried in many steps, may it's that it involves a number of database tables or some steps depend on the results of previous ones.
  - For databases, it is a good practice to use **transactions** so that either all steps succeed or all steps fail. Database tables are **locked** during the transaction to ensure *the entire change is carried out as expected*.
  - The database term for this desirable principle/property is called *atomicity*. Changes that do not adhere to this principle can run into **race conditions**
  
  In this case, I shall illustrate with the following diagram, which demonstrates a coupon being used 4 times:
```
request 1   C----A--U
request 2      C---A---U
request 3        C--A-----U
request 4          C---A----U
request 5             C--X (failed)
Legends: C: Check coupon history; A: Add credit; U: Update coupon history
```
  
  So a simple workflow would be:
  1. Invoke purchase C8 endpoint, you'll be given a session
  1. Spam the apply coupon endpoint
  1. Invoke purchase C8 endpoint with the session.
  
  I had written a Python script with multiprocessing pools that worked quite well locally but not remotely (only 1 request went through). Thankfully the forum suggested Turbo Intruder and I was able to pull it off.
  
``` py
def queueRequests(target, wordlists):
  engine = RequestEngine(endpoint=target.endpoint,
                         concurrentConnections=100,
                         requestsPerConnection=100,
                         pipeline=False
                         )

  for i in range(200):
      engine.queue(target.req)


def handleResponse(req, interesting):
  if '200 OK' in req.response:
      table.add(req)
```
  
  ### Footnote:
  - I still had >60 bucks left after buying C8.
  - Adding a user coupon count or combining "add credit" with "update coupon history" into 1 sql command does **not** solve the core issue.
  - I once had a non-commercial project with this vulnerability (that's not intentional). Not using transactions can be something that's missed easily.
