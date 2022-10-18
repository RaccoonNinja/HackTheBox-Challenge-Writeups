## Skills involved: Using a suitable tool

This writeup will be as beginner-friendly as possible ~because I'm also a beginner to the tool used~.

<details>
  <summary> Hint 1: </summary>
  
  What framework does this binary use? What are the useful reversing tools?
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  
  <br/>
  
  This 32-bit exe is built using the .NET framework. This can be easily found out by using the strings command.
  
  After research, we learnt that we can use [dnspy](https://github.com/dnSpy/dnSpy) to set breakpoints, add variable watches and edit methods and classes in C# or Visual Basic.
  
  Also all strings are obfuscated, it's not hard to know which part corresponds to which functionality. Here's how I set up the breakpoints.
  
  <img width="721" alt="bp" src="https://user-images.githubusercontent.com/114584910/196462701-6ef91042-25d6-4dc2-a3bb-b374612d708c.png">
  
  When the program breaks, the values of local variables are displayed and *can be edited*:
  
  <img width="362" alt="locals" src="https://user-images.githubusercontent.com/114584910/196462717-5f6f4a89-d8d8-4b9a-8964-2c6ba00867c3.png">
  
  The flag comes right away.

</details>
