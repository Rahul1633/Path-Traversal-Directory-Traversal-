# Path-Traversal-Directory-Traversal
This section documents my learning and practice of Path Traversal vulnerabilities from PortSwigger Web Security Academy.


What is path traversal?

Path traversal is also known as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application. This might include
Application code and data.
Credentials for back-end systems.
Sensitive operating system files.
    
On Windows, both ../ and ..\ are valid directory traversal sequences. The following is an example of an equivalent attack against a Windows-based server:
'https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini'


Lab: File path traversal, simple case 
 solution : use ../ for traversal 

If an application strips or blocks directory traversal sequences from the user-supplied filename, it might be possible to bypass the defense using a variety of techniques.

example :  filename=/etc/passwd to directly reference a file without using any traversal sequences.

 Lab: File path traversal, traversal sequences blocked with absolute path bypass
 solution: use  above notes to solve 
 
Nested Traversal Sequences

What it is: Using modified traversal strings like ....// or ....\/ which become normal traversal sequences after the inner sequences are stripped out.

How it works: When a filter removes ../ naively but not recursively or fully, the nested sequences collapse to valid ../ patterns after stripping. For example, ....// becomes ../ once filtered.

When used: To bypass defensive filters that only detect and remove basic traversal patterns but do not handle these nested or obfuscated variants.

Benefit: Effective against weak or incomplete sanitization, allowing attackers to still perform directory traversal despite protections.



Lab: File path traversal, traversal sequences stripped non-recursively
 solution : Use nested traversal sequences to solve this


Explanation: URL Encoding and Double Encoding in Path Traversal
When a website tries to block directory traversal sequences like ../ by filtering or stripping them directly, attackers can bypass this defense by encoding these characters so the filter does not recognize them, but the server still understands them after decoding.

Basic URL Encoding
The basic traversal sequence is ../ (dot-dot-slash).

In URL encoding, each character is represented by a % followed by its ASCII hex code:

. becomes %2e

/ becomes %2f

So ../ becomes %2e%2e%2f.

Example:

Normal traversal:

text
https://example.com/load?file=../../etc/passwd
Encoded traversal:

text
https://example.com/load?file=%2e%2e%2f%2e%2e%2fetc/passwd
The filter might only look for ../ literally and miss the encoded version, but the server decodes %2e%2e%2f back to ../ and reads the file.

Double URL Encoding
Some websites decode inputs twice due to various processing layers.

Double encoding means encoding the % itself to %25, so %2e becomes %252e.

Thus, ../ double URL encoded turns to %252e%252e%252f.

Example:

Double encoded traversal:

text
https://example.com/load?file=%252e%252e%252f%252e%252e%252fetc/passwd
When the request passes through two decoding phases, the %252e%252e%252f is first decoded to %2e%2e%2f, then second decoded to ../, allowing traversal.

Other Non-Standard Encodings
Attackers may also use other encodings to bypass filters, such as:

..%c0%af (overlong UTF-8 encoding for /)

..%ef%bc%8f (UTF-8 full-width slash)

Filters not designed to detect these encodings can be bypassed, and the server may still decode these forms to valid traversal sequences.
