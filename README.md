# Path-Traversal-Directory-Traversal
This section documents my learning and practice of Path Traversal vulnerabilities from PortSwigger Web Security Academy.


What is path traversal?

Path traversal is also known as directory traversal. These vulnerabilities enable an attacker to read arbitrary files on the server that is running an application. This might include:

    Application code and data.
    Credentials for back-end systems.
    Sensitive operating system files.
    
On Windows, both ../ and ..\ are valid directory traversal sequences. The following is an example of an equivalent attack against a Windows-based server:
https://insecure-website.com/loadImage?filename=..\..\..\windows\win.ini


Lab: File path traversal, simple case 
 solution : use ../ for traversal  
