# `cracking` MS Office Password

For real? yes.

## Some Background

In case one forgets the password to an important encrypted excel file! No, not protected. ENCRYPTED. There's is a distinction between "protected" and "encrypted" MS Office files. I am not sure why on earth or what on earth is the precise difference but AFAICT encrypted one just AES and protected is just like write protection.


If you google `how to crack excel password` you would most likely end up with people trying to show how to `unprotect` which doesn't crack any passwords. If you're lucky enough you'd get to an article by [null-byte](https://null-byte.wonderhowto.com/how-to/crack-password-protected-microsoft-office-files-including-word-docs-excel-spreadsheets-0193959/) which is a great resource. Good starting point but you may want to use your GPU to boost up hash rate.


## What did I do? BRUTEFORCE but smarter

Wait wait, brute force? isn't that something that can possibly take a few hundred years to crack passwords? Yep, that's the one I am talking about. In theory, it can take years to crack a strong password. Something like `aTt%wkrMo*k7]W` would take almost a millennia for supercomputers. However, something like `hithere` would take a few minutes on your desktop computer. My assumption here was that the password I was looking for falls into `hithere` category so I can do a hybrid dictionary brute force.

## How?

Use Tools and GPU! (and MPI if nothing works out)

### JtR (John The Ripper) [https://github.com/openwall/john/](https://github.com/openwall/john/)

Given that I was on windows (yes, I do have a windows machine for games, don't judge me, okay?) grab the prebuilt binaries from https://www.openwall.com/john/. Fire up a [Git bash ](https://git-scm.com/download/win). I am assuming you have [python](https://www.python.org/downloads/) installed on your machine and it is in your PATH. DO NOT USE PowerShell or CMD! For reasons not known to men, PowerShell appends some weird bytes when one pipes the output to a file!

Extract password hash from the office file,

```bash
cd john-1.9.0-jumbo-1-win64/run

python office2john.py myfile.xlsx > hash.txt
```

### GPU and OpenCL 

JtR can use OpenCL to generate hashes, however, its configs don't run out of the box,

```bash
$ ls john-1.9.0-jumbo-1-win64/etc/OpenCL/vendors/
amd.icd  nvidia.icd

$ cat john-1.9.0-jumbo-1-win64/etc/OpenCL/vendors/nvidia.icd
c:\Windows\System32\nvopencl.dll
```

Typically on new systems the `nvopencl.dll` is named `nvopencl64.dll` and it is somewhere in `c:\Windows\System32\DriverStore\FileRepository` so just copy it `c:\Windows\System32\nvopencl.dll` or anywhere you fancy and update the `nvidia.icd` file.

### Run the JtR

just make sure you're in `run/kernel/` dir otherwise the OpenCL kernel fails to find header files.

```bash
$ cd john-1.9.0-jumbo-1-win64/run/kernel/
$ ../john.exe --format=office-opencl ../hash.txt
```

One may want to change `run/john.conf`. I didn't do it because I got the password by the time I finish researching how to write optimized config.

### MPI?

If a single GPU wouldn't have worked, My next step was to use all of my compute resources to run JtR. This is somewhat described here https://countuponsecurity.com/2015/05/07/step-by-step-clustering-john-the-ripper-on-kali/. However, one wouldn't need NFS for a non-wordlist attack. OpenMPI does have windows support and it doesn't seem difficult to build JtR with MPI on Windows.

# Conclusion?

For realistic and quick solutions, break things fast and fix one problem at a time.
