# 🔐 Path-Traversal-Directory-Traversal

This section documents my learning and practice of **Path Traversal** vulnerabilities from PortSwigger Web Security Academy.  
I’m using these labs to strengthen my **web security**, **file system knowledge**, and **problem-solving skills**.

---

## What is path traversal?

Path traversal is also known as **directory traversal**.  
These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application. This might include:

- Application code and data  
- Credentials for back-end systems  
- Sensitive operating system files  

On **Windows**, both `../` and `..\` are valid directory traversal sequences.  
The following is an example of an equivalent attack against a Windows-based server:

"https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini"


---

### Lab: File path traversal, simple case  
**Solution**: Use `../` for traversal  

**Example**: 
Directly reference a file without using any traversal sequences :  `filename=/etc/passwd` 


#### Lab: File path traversal, traversal sequences blocked with absolute path bypass  

**Solution**: Use above notes to solve  

---

## Nested Traversal Sequences

**What it is**: 
Using modified traversal strings like `....//` or `....\/` which become normal traversal sequences after the inner sequences are stripped out.

**How it works**:  
When a filter removes `../` naively but not recursively or fully, the nested sequences collapse to valid `../` patterns after stripping. 

Example: 
`....// → ../`


**When used**:  
To bypass defensive filters that only detect and remove basic traversal patterns but do not handle these nested or obfuscated variants.

**Benefit**:  
Effective against weak or incomplete sanitization, allowing attackers to still perform directory traversal despite protections.

#### Lab: File path traversal, traversal sequences stripped non-recursively  
**Solution**: Use nested traversal sequences to solve this  

---

## Explanation: URL Encoding and Double Encoding in Path Traversal

When a website tries to block directory traversal sequences like `../` by filtering or stripping them directly, attackers can bypass this defense by encoding these characters so the filter does not recognize them, but the server still understands them after decoding.

### Basic URL Encoding

The basic traversal sequence is `../`.  
In URL encoding, each character is represented by `%` followed by its ASCII hex code:

| Character | Encoded form |
|-----------|-------------|
| .         | %2e         |
| /         | %2f         |

So:  
`../ → %2e%2e%2f`

**Example**:

Normal traversal:
https://example.com/load?file=../../etc/passwd

Encoded traversal:
https://example.com/load?file=%2e%2e%2f%2e%2e%2fetc/passwd


The filter might only look for `../` literally and miss the encoded version, but the server decodes `%2e%2e%2f` back to `../` and reads the file.

---

### Double URL Encoding

Some websites decode inputs twice due to various processing layers.  
Double encoding means encoding the `%` itself to `%25`, so `%2e` becomes `%252e`.  

Thus:  
`../ → %252e%252e%252f`

**Example**:

Double encoded traversal:
https://example.com/load?file=%252e%252e%252f%252e%252e%252fetc/passwd


When the request passes through two decoding phases, the `%252e%252e%252f` is first decoded to `%2e%2e%2f`, then to `../`, allowing traversal.

---

### Other Non-Standard Encodings

Attackers may also use other encodings to bypass filters, such as:

- `..%c0%af` (overlong UTF-8 encoding for `/`)  
- `..%ef%bc%8f` (UTF-8 full-width slash)  

Filters not designed to detect these encodings can be bypassed, and the server may still decode these forms to valid traversal sequences.

---

#### Lab: File path traversal, traversal sequences stripped with superfluous URL-decode  
**Solution**: Use double-encoded traversal to solve this  

---

#### Lab: File path traversal, validation of start of path  
**Solution**:  
An application may require the user-supplied filename to start with the expected base folder, such as `/var/www/images`.  

Example solution:  
`filename=/var/www/images/../../../etc/passwd`


---

#### Lab: File path traversal, validation of file extension with null byte bypass  
**Solution**:  
An application may require the user-supplied filename to end with an expected file extension, such as `.png`.

Use a null byte to terminate the file path before the required extension:  
`filename=../../../etc/passwd%00.png`


---

## How to prevent a path traversal attack

The most effective way to prevent path traversal vulnerabilities is to **avoid passing user-supplied input to filesystem APIs altogether**.  
Many application functions can be rewritten to deliver the same behavior in a safer way.

If you can't avoid passing user-supplied input to filesystem APIs, use **two layers of defense**:

1. **Validate the user input before processing it.**  
   Ideally, compare the user input with a whitelist of permitted values.

2. If a whitelist is not possible, verify that the input contains only permitted content.
