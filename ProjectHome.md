# Overview #

IOCTLbf is just a small tool (Proof of Concept) that can be used to search vulnerabilities in Windows kernel drivers by performing two
tasks:
  * **Scanning for valid IOCTL codes supported by drivers**,
  * **Generation-based IOCTL fuzzing**

```
	 _                   _  _       ___ 
	(_)              _  | || |     / __)
	 _  ___   ____ _| |_| || |__ _| |__ 
	| |/ _ \ / ___|_   _) ||  _ (_   __)
	| | |_| ( (___  | |_| || |_) )| |   
	|_|\___/ \____)  \__)\_)____/ |_|     Proof of Concept
	
```


**An advantage of this tool is that it does not rely on captured
IOCTLs. Therefore, it is able to detect valid IOCTL codes supported
by drivers and that are not often, or even never, used by
applications from user land**. For example, it may be the case for:
  * IOCTLs called in very specific conditions (not easy to discover and/or to reproduce).
  * IOCTLs used for debugging purpose that are sometimes let in drivers.

Once scanning is done and valid IOCTLs have been found for a given driver, the user can choose one IOCTL in the list to begin the fuzzing process.
Note that this tool only performs generation-based fuzzing. Compared to mutation-based fuzzing (which consists in taking valid IOCTL buffers and adding anomalies), the code coverage is of
course less important.

Note: for mutation-based IOCTL fuzzing, you should really check out the great tool _IOCTL fuzzer_ (http://code.google.com/p/ioctlfuzzer/).


# Usage #

  1. First of all, it is necessary to locate the target driver. A tool like _Driver View_ (http://www.nirsoft.net/utils/driverview.html) can be used in order to easily spot non-Microsoft drivers (third-party drivers).
  1. Then, it is necessary to check the device(s) associated with the target driver. A good tool to do this is _Device Tree_ for example (http://www.osronline.com/article.cfm?article=97).
  1. Check the security attributes (DACL) of the device(s). It should be available for limited users in order to make it interesting from an attacker point of view. Indeed, vulnerabilities in drivers may lead to Local Privilege Escalation on the system, or just Denial of Service when it is not exploitable.
  1. Retrieve the symbolic link used by applications to communicate with one device of the target driver. All symbolic links can be listed with the Sysinternal's tool _Winobj_ in the "GLOBAL??" section (http://technet.microsoft.com/en-us/sysinternals/bb896657).
  1. Finally, it is necessary to know at least one valid IOCTL code supported by the target driver. For example, it can be easily done by monitoring IRPs with a tool like _OSR's Irp Tracker Utility_ (http://www.osronline.com/article.cfm?article=199). Make sure to apply a filter on "DEVICE\_CONTROL" only and to select only the target driver. Of course, it is also possible to retrieve valid IOCTL codes directly by reverse engineering the driver.
  1. Once a valid IOCTL code is retrieved, IOCTLbf can be used. **The scanning process returns supported IOCTL codes and accepted buffer sizes for each one**. One of the following IOCTL codes scanning modes can be chosen:
    * Function code + Transfer type bruteforce
    * IOCTL codes range
    * Single IOCTL code
  1. The next step simply consists in choosing one IOCTL to fuzz. The fuzzing process actually follows the following steps:
    * (if method != METHOD\_BUFFERED) Invalid addresses of input/output buffers
    * Check for trivial kernel overflows
    * Fuzzing with predetermined DWORDs (invalid addresses, addresses pointing to long ascii/unicode strings, address pointing to a table of invalid addresses)
    * Fuzzing with fully random data

More information available on http://poppopret.blogspot.com and in the README.txt file.


# Example #

![http://3.bp.blogspot.com/-_nviFZvHxss/T3WlI7zcPRI/AAAAAAAAAFI/Yu0joNiqVhk/s1600/screenshot_ioctlbf_1.png](http://3.bp.blogspot.com/-_nviFZvHxss/T3WlI7zcPRI/AAAAAAAAAFI/Yu0joNiqVhk/s1600/screenshot_ioctlbf_1.png)

![http://2.bp.blogspot.com/-N0IkLB2IU7o/T3WlmkPyXUI/AAAAAAAAAFQ/Sl3sWwfEH7M/s1600/screenshot_ioctlbf_2.png](http://2.bp.blogspot.com/-N0IkLB2IU7o/T3WlmkPyXUI/AAAAAAAAAFQ/Sl3sWwfEH7M/s1600/screenshot_ioctlbf_2.png)