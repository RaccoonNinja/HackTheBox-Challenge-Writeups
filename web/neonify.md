## Skills involved: Ruby

This is another simple challenge. It took some time for me because I didn't know any ruby. The forum was helpful.

<details>
  <summary> Hint 1: </summary>
  
  What are the possible vectors? (If you haven't downloaded the source code, download it)
</details>
<details>
  <summary> Hint 2: </summary>
  
  What stops you from achieving what you want? How would you [learn more](https://ruby-doc.org/core-3.1.2/Regexp.html)?
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  There is only 1 attack vector: the `neon` POST parameter. Without the `/^[0-9a-z ]+$/i` regex whitelisting it's an easy RCE.
  
  I'm more acquianted with JavaScript and to me the regex was perfectly normal. **Nonetheless in ruby**:
  
  ![image](https://user-images.githubusercontent.com/114584910/193067610-80c6ba52-68d0-4d98-aae3-49528618e908.png)

  Which means:
  ``` js
  /^[0-9a-z ]+$/i.test("hey\n!!!")   // false in javascript
  ```
  but:
  ``` ruby
  "hey\n!!!" =~ /^[0-9a-z ]+$/i       # true in ruby
  ```
  
  One can come up with payloads like ```lol\n<%= `cat flag.txt` %>```.
  
  **Footnote:** That is not the only difference between ruby and JavaScript. Contrary to most languages, *both 0 and 1 are truthy in ruby*.
</details>
