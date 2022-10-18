## Skills involved: Automation

Many misc challenges are related to automation, brute-forcing or more extensive coding. The challenge is the how-to-do part.

<details>
  <summary> Hint 0: </summary>
  
  What is the pattern shown when you connect to it? The title may be a good hint.
</details>

<details>
  <summary> Hint 1: </summary>
  
  What are the common tools related to QR code parsing? How can you use them?
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  In this challenge, you have to scan a QR code for an algebraic expression. Upon solving the expression, you'll be given a the flag.
  
  It's relatively easy to do the challenge in Python following these steps:
  1. parsing the output with [nclib](http://nclib.readthedocs.io/) or [pwntools](https://docs.pwntools.com/en/stable/)
  1. creating an image with [Pillow](https://pillow.readthedocs.io/en/stable/)
  1. parsing the QR code with [opencv-python-headless](https://pypi.org/project/opencv-python/)
  1. eval-ing the equation *(I didn't specifically find a safe eval)*
  1. sending the result to the server
  
  Here's my code for the challenge using pwntools:
  
```py
from PIL import Image
from pwn import *
import cv2

r = remote(...)
r.readuntil(b'3 seconds!\n\n\n\t')
response = r.readuntil(b'\n\t\n\n')
r.readuntil(b"Decoded string: ")

for junk in [b" ", b"\t"]:
  response = response.replace(junk, b"")
  
response = response.replace(b"\x1b[7m\x1b[0m", b"1")
response = response.replace(b"\x1b[41m\x1b[0m", b"0")
response = response.decode('utf8').strip()

array = response.split("\n")
height = len(array)
width = len(array[0])
qr_array = [int(c) for c in response.replace("\n", "")]

im = Image.new("1", (width, height))
im.putdata(qr_array)
IM = im.resize((width*8, height*8))
IM.save("qr.png")

qrcode = cv2.imread("qr.png")
detector = cv2.QRCodeDetector()
data = detector.detectAndDecode(qrcode)[0]

ans = eval(data.replace(" =", "").replace("x", "*"))
r.send(bytes(str(ans)+"\n", 'utf-8'))
r.interactive()
```
</details>
