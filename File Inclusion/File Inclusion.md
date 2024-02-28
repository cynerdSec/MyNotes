
# File Inclusion

## Introduction

**What is File inclusion?**

This room aims to equip you with the essential knowledge to exploit file inclusion vulnerabilities, including Local File Inclusion (LFI), Remote File Inclusion (RFI), and directory traversal. Also, we will discuss the risk of these vulnerabilities if they're found and the required remediation. We provide some practical examples of each vulnerability as well as hands-on challenges.

In some scenarios, web applications are written to request access to files on a given system, including images, static text, and so on via parameters. Parameters are query parameter strings attached to the URL that could be used to retrieve data or perform actions based on user input. The following diagram breaks down the essential parts of a URL.

![URL diagram](Url_Structure.png)

For example, parameters are used with Google searching, where `GET` requests pass user input into the search engine. `https://www.google.com/search?q=TryHackMe` If you are not familiar with the topic, you can view the [How The Web Works](https://tryhackme.com/module/how-the-web-works) module to understand the concept.  
Let's discuss a scenario where a user requests to access files from a webserver. First, the user sends an HTTP request to the webserver that includes a file to display. For example, if a user wants to access and display their CV within the web application, the request may look as follows, `http://webapp.thm/get.php?file=userCV.pdf`, where the **file** is the parameter and the `userCV.pdf`, is the required file to access.

![LFI](LFI_Demo_1.png)

**Why do File inclusion vulnerabilities happen?**

File inclusion vulnerabilities are commonly found and exploited in various programming languages for web applications, such as PHP that are poorly written and implemented. The main issue of these vulnerabilities is the input validation, in which the user inputs are not sanitized or validated, and the user controls them. When the input is not validated, the user can pass any input to the function, causing the vulnerability.

**What is the risk of File inclusion?**

By default, an attacker can leverage file inclusion vulnerabilities to leak data, such as code, credentials or other important files related to the web application or operating system. Moreover, if the attacker can write files to the server by any other means, file inclusion might be used in tandem to gain remote command execution (RCE).

## Path Traversal

Also known as **Directory traversal**, a web security vulnerability allows an attacker to read operating system resources, such as local files on the server running an application. The attacker exploits this vulnerability by manipulating and abusing the web application's URL to locate and access files or directories stored outside the application's root directory.

Path traversal vulnerabilities occur when the user's input is passed to a function such as file_get_contents in PHP. It's important to note that the function is not the main contributor to the vulnerability. Often poor input validation or filtering is the cause of the vulnerability. In PHP, you can use the file_get_contents to read the content of a file. You can find more information about the function [here](https://www.php.net/manual/en/function.file-get-contents.php).

The following graph shows how a web application stores files in `/var/www/app`. The happy path would be the user requesting the contents of userCV.pdf from a defined path `/var/www/app/CVs`.

![LFI](LFI_Demo_2.png)

We can test out the URL parameter by adding payloads to see how the web application behaves. Path traversal attacks, also known as the `dot-dot-slash attack`, take advantage of moving the directory one step up using the double dots `../.` If the attacker finds the entry point, which in this case `get.php?file=`, then the attacker may send something as follows, `http://webapp.thm/get.php?file=../../../../etc/passwd`

Suppose there isn't input validation, and instead of accessing the PDF files at `/var/www/app/CVs` location, the web application retrieves files from other directories, which in this case `/etc/passwd`. Each `..` entry moves one directory until it reaches the root directory `/`. Then it changes the directory to `/etc`, and from there, it read the `passwd` file.

![LFI](LFI_Demo_3.png)

As a result, the web application sends back the file's content to the user.

![LFI](LFI_Demo_4.png)

Similarly, if the web application runs on a Windows server, the attacker needs to provide Windows paths. For example, if the attacker wants to read the `boot.ini` file located in `c:\boot.ini`, then the attacker can try the following depending on the target OS version:

`http://webapp.thm/get.php?file=../../../../boot.ini` or

`http://webapp.thm/get.php?file=../../../../windows/win.ini`

The same concept applies here as with Linux operating systems, where we climb up directories until it reaches the root directory, which is usually `c:\`.

Sometimes, developers will add filters to limit access to only certain files or directories. Below are some common OS files you could use when testing. 



| Location      | Description |
| ----------- | ----------- |
| `/etc/issue`      | contains a message or system identification to be printed before the login prompt. |
| `/etc/profile`   | controls system-wide default variables, such as Export variables, File creation mask (umask), Terminal types, Mail messages to indicate when new mail has arrived |
| `/proc/version`   | specifies the version of the Linux kernel |
| `/etc/passwd`   | has all registered user that has access to a system |
| `/etc/shadow`   | contains information about the system's users' passwords |
| `/root/.bash_history`   | contains the history commands for `root` user |
| `/var/log/dmessage`   | contains global system messages, including the messages that are logged during system startup |
| `/var/mail/root`   | all emails for `root` user |
| `/root/.ssh/id_rsa`   | Private SSH keys for a root or any known valid user on the server |
| `/var/log/apache2/access.log`   | the accessed requests for `Apache`  webserver |
| `C:\boot.ini`   | contains the boot options for computers with BIOS firmware |

## Remediation

As a developer, it's important to be aware of web application vulnerabilities, how to find them, and prevention methods. To prevent the file inclusion vulnerabilities, some common suggestions include:

1. Keep system and services, including web application frameworks, updated with the latest version.
2. Turn off PHP errors to avoid leaking the path of the application and other potentially revealing information.
3. A Web Application Firewall (WAF) is a good option to help mitigate web application attacks.
4. Disable some PHP features that cause file inclusion vulnerabilities if your web app doesn't need them, such as allow_url_fopen on and allow_url_include.
5. Carefully analyze the web application and allow only protocols and PHP wrappers that are in need.
6. Never trust user input, and make sure to implement proper input validation against file inclusion.
7. Implement whitelisting for file names and locations as well as blacklisting.