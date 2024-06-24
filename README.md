![Obfell Banner Image](/images/obfell_banner.png)

# Obfell

An invisible Python Powershell obfuscator using randomized character arrays, character casting, and integer sub-division into arithmethic operations. Transform scripts into a complex, unreadable code that 
is undetected by _most_ anti-viruses. 

## Table of Contents
- [Usage](#usage)
- [How it Works](#how-it-works)
- [Installation](#installation)
- [Performance and Detection/Evasion](#performance-and-detectionevasion)
- [License](#license)

# Usage

The obfuscator works by executing `obfell.py` with the necessary parameters.

For information and usage, see the `help` menu (`python3 obfell.py -h`).

```
usage: obfell.py [-h] -f FILEPATH -a ARRAYS [-n NAME_LENGTH] [-o OPERATIONS]

Obfuscate Powershell scripts using randomized characters arrays with character casting

options:
  -h, --help            show this help message and exit
  -f FILEPATH, --filepath FILEPATH
                        Original Powershell script path
  -a ARRAYS, --arrays ARRAYS
                        Amount of obfuscated arrays to generate
  -n NAME_LENGTH, --name_length NAME_LENGTH
                        Length of variable name for arrays (optional, default is 32)
  -o OPERATIONS, --operations OPERATIONS
                        Amount of operations to sub-divide numbers (optional, default is 1)
```

When executed, it will output an obfuscated Powershell script named `output.ps1`.

A usage example for a script named 'script.ps1', obfuscated with 20 arrays, variable name lengths of 64, and 5 arithmetic operations.
```shell
python3 obfell.py -f script.ps1 -a 20 -n 64 -o 5
```

# How It Works
To obfuscate a Powershell script, Obfell uses various mechanisms to make the code incredibly hard to read and bigger in memory size. 
* First, it creates arrays with every printable character, however, the name of the array and the order of each character are randomized.
* Secondly, it uses casting from integer to character to increase unreadability, for example, instead of an array storing the character `d`, it is stored as `[char](110 - 10)`.
* Thirdly, as seen previously, instead of using the direct ASCII value, it sub-divides the value into arithmetic operations. Using the previous example, instead of `[char](100)`, which casts to `d`, it stores it as `[char](110 - 10)`. The amount of arithmetic operations is by default 1, used in the example. If the arithmetic operations parameters is passed as 0, it will not sub-divide the value. 
* Fourth, it puts every character in the code into an array of characters which is joined to form a string and executed using `iex` (`Invoke-Expression`). For each character of the code, it randomly chooses one of the arrays and uses the chosen array with the index of the character to "reform" the code. This is possible because it uses a dictionary to keep track of all the arrays, their name, and their content.

While the explanation might be confusing, a demonstration might be easier to comprehend.

The original Powershell script is:
```ps1
echo "Hello World"
```
Let's obfuscate it using Obfell.

Only 2 randomized arrays with a variable name of 5 characters will be used for simplicity's sake, these are arguments that can be passed for customization. The arrays generated look like this:

```ps1
# Both contain all printable characters
$LJAHS = ([char](80 + 2),[char](43 + 19),[char](80 - 16),...)
$tVAMg = ([char](30 + 25),[char](- 49 + 90),[char](89 - 29),...)
```

Now, let's rebuild the original script using the characters inside of the arrays. It will end up looking like this:

```ps1
# It can be seen that the array used to "pull" each character is random
iex (($LJAHS[87 - 48],$LJAHS[- 16 + 103],$tVAMg[75 + 10],...) -JOIN "")
#            e               c                  h
```

In short, while the original is very simple and readable
```ps1
echo "Hello World"
```
the obfuscated is an extremely hard to comprehend
```ps1
$LJAHS = ([char](80 + 2),[char](43 + 19),[char](80 - 16),...)
$tVAMg = ([char](30 + 25),[char](- 49 + 90),[char](89 - 29),...)

iex (($LJAHS[87 - 48],$LJAHS[- 16 + 103],$tVAMg[75 + 10],...) -JOIN "")
```
Both execute the same code. 

# Installation
To install, simply download the repo and run. Only standard libraries were used
```
git clone https://github.com/Jael-G/Obfell
cd obfell
python3 obfell.py (args)
```

# Performance and Detection/Evasion
### Performance
There are two aspects of performance that should be noted. These are the performance when generating the obfuscated script and the performance when executing said script.

#### Generation:

How fast the Python script is able to generate an obfuscated code depends greatly on the amount of arrays used. For example, using something like 100 arrays is almost instant, while a huge amount in the thousands might take a while. The reason for this is most likely the immense amount of characters the obfuscated script generates and having to write it all into an output file. 

In the example in [How it Works](#how-it-works), the original script was simply 18 characters long while the obfuscated version had an astonishing 3801 characters. In the example only two arrays were used (normally hundreds if not thousands would be preferred), the variable name length of each array was only 5 characters long (the default is 32), and only one arithmetic operation was used to sub-divide each value. With those extremely low parameters, a 21016.7% increase in content was achieved.

The very same original script obfuscated with 500 arrays, variable names of 100 characters, and each integer sub-divided into 10 arithmetic operations is extended to 3,186,376 characters and, even then, it took approximately only 3 seconds to generate... Imagine how much bigger it can get, both in content and memory size. 

#### Execution
Execution time is the biggest drawback of using Obfell to obfuscate a Powershell script.

It's not a surprise that the more complex and obfuscated the file becomes, the longer the time to execute is. Once again using the previous example, unobfuscated it runs, for all sense and purposes, instantly. When obfuscated with the last parameters (500 arrays, 100 characters variable names, 10 arithmetic operations), it takes approximately a massive 52 seconds. This is a huge increase in execution time. In theory, Obfell would be best for scripts that can be left to take their time to run in the background.

### Detection/Evasion
I believe this is one of the strong points of Obfell. When it comes to evading anti-viruses, it seems to work extremely well. It was tested on various computers (with consent) and it was executed without being removed by Microsoft's `Virus & Threat Protection`. The most important test was _VirusTotal_. These tests were performed as follows:

The script used was a known Powershell reverse shell one-liner obtained from [here](https://gist.github.com/egre55/c058744a4240af6515eb32b2d33fbed3).

#### Unobfuscated results
When the script was saved to Windows computers, it was automatically deleted and the user was alerted. 

When uploaded to _VirusTotal_, it instantly got detected as malware. This is due to the fact that this script is commonly used, so _VirusTotal_ did not have to analyze it. It was flagged by 28 security vendors.

![Original Result Picture](/images/original_result.png)

#### Obfuscated Results
When obfuscated with 500 arrays, variable names with a length of 100 characters, and 10 arithmetic operations, Windows did not detect it and could be easily stored in the computer. 

When uploaded to _VirusTotal_, it was slowly analyzed, as the obfuscated script wasn't in _VirusTotal's_ database. In the end, it showed 0 flags from vendors, although both scripts (original and obfuscated) run exactly the same code. 

![Obfuscated Result Picture](/images/obfuscated_result.png)

# License
Copyright (c) 2024 Jael Gonzalez

The content of this repository is bound by the MIT licenses:

```
MIT License

Copyright (c) 2024 Jael Gonzalez

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
