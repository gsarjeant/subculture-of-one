---
date: 2025-01-16T00:00:00-05:00
draft: false
params:
  author: Greg Sarjeant
title: An act of preservation
aliases:
  - "/2025/01/17/an-act-of-preservation.html"
weight: 10
tags: [stumps]
---

This is a guide that I wrote for the [gentoo forums](https://forums.gentoo.org/viewtopic-t-122145.html) in 2004. I've been thinking about submitting a piece for this month's [IndieWeb Carnival](https://vhbelvadi.com/indieweb-carnival-friction) and I think my experience with ACPI in the early 2000s is a good topic for the theme: On the Importance of Friction.

It also struck me that this is a good thing to preserve on my own site. It remains one of my proudest writing accomplishments, and although it's long since defunct, I'd feel a sense of loss if the gentoo forums were to disappear and this were to be lost.

So, I'm republishing it here for posterity. If you should stumble across it because you're actually looking for information about ACPI on Linux, I beg you, _stop reading now_. There is nothing here that is of any technological value in 2025. Some of it may hold entertainment value, depending on what you find entertaining.

## HOWTO: Fix Common ACPI Problems (DSDT, ECDT, etc.)

__NOTE (2009-11-09)__: This guide is coming up on 6 years old, so it's probably pretty outdated, especially where kernel options and such are concerned. Some of the links are also stale.

For the past few weeks, I've been wrestling with ACPI on my laptop (a Gateway 200X). The main problems are that it has a buggy DSDT and a missing ECDT. A bit of googling revealed that neither is an uncommon problem. Unfortunately, some other OSes are more forgiving of deviations from the ACPI spec than they should be, and that causes problems for the rest of us. The only real solutions are to provide a fixed DSDT and ECDT to the kernel.

Now when I heard this, at first I was afraid. I was petrified. But as it turned out, it wasn't so hard to fix the problem. The trickiest part was diagnosing it; from there (in my case at least) the fix was very straightforward. Here's how to go about doing it if you are also having trouble with ACPI on your machine. Much of this is taken from this post, but I've weeded out some other stuff that was more specific to my machine.

NOTE

This involves some kernel patching, and possibly overwriting your existing initrd. PLEASE make backups of important things. In particular, I found that backups of my kernel configs, working kernels and bootsplash initrd were extremely helpful.

Table of Contents

	Revisions
	Definitions
	Description of the Problem
	Disclaimer
	Tested Kernels (DSDT override)
	What you'll need to fix your DSDT
	Diagnosing a Buggy DSDT
	Fixing the DSDT
	Incorporating the fixed DSDT into the kernel
	    Static DSDT override
	    initrd DSDT override
	    initrd override with bootsplash
	My DSDT is fixed, but I still have ACPI errors. Now what?
	    Windows-only DSDT functionality
	    Missing ECDT
	    My power and/or sleep buttons don't work
	    My BIOS is blacklisted
	    I have an Asus laptop
	    It's still not working!
	Useful ACPI resources
	Acknowledgements

## 1. Revisions

### 01/08/2004
* Added three new sections:
	1. Revisions
	2. Definitions
	3. Description of the Problem
	
### 01/09/2004
* Tested with some 2.6 kernels
* Added initrd patch section, corrected misinformation about bootsplash and initrd DSDT (it does work)
* Reformatted to conform better to the guidelines
* Added links to HOWTOs with fixes for common DSDT errors
* Added fixes for warnings to the main doc
	
### 01/10/2004
* Added missing ACPI resource links
* Corrected numerous typos
	
### 01/11/2004
* Added Disclaimer section
	
### 01/12/2004
* Added explanation of "Return(Package(0x02){0x00, 0x00})" fix for \_WAK method warning.
	
### 01/13/2004
* Modified Intel iasl links to point to the main download page. Thanks, Moled.

### 01/14/2004
* Clarified Description of the Problem, included Microsoft compiler information. Thanks, Kesereti.
* Added link to the acpi.sourceforge.net DSDT Repository to section 8.
	
### 01/25/2004
* Added a couple paragraphs about acpi\_os\_name boot parameter to section 8.
* Added new section for ECDT, acpi\_os\_name and ff button problems
* Added a couple more links to patches
* Removed To Do section, since it's all done.
* Renamed, because this has evolved into more of a general ACPI HOWTO
	
### 01/26/2004
* Clarified section 10a (Windows-only DSDT functionality)
	
### 02/02/2004
* Reformatted revisions as code block so they don't take up so much space.
* Added blacklisted BIOS section.
* Added acpi4asus section.
* Added brief information about some other acpi boot parameters to Section 10f.
	
## 2. Definitions

ACPI Machine Language (AML)

From the ACPI Spec, page 13:

> Pseudocode for a virtual machine supported by an ACPI-compatible OS and in which ACPI control methods and objects are written. 

This is what the DSDT is compiled to. An ACPI-compatible OS must be able to understand AML. You don't have to be able to.

ACPI Source Language (ASL)

Again from the Spec, Page 13

> The programming language equivalent for AML. ASL is compiled into AML images. 

This is the language that the DSDT is written in. If you need to fix your DSDT, you'll have to write at least a little bit of ASL.

Differentiated System Description Table (DSDT)

From the ACPI Spec, page 14:

> An OEM must supply a DSDT to an ACPI-compatible OS. The DSDT contains the Differentiated Definition Block, which supplies the implementation and configuration information about the base system. The OS always inserts the DSDT information into the ACPI Namespace at system boot time and never removes it. 

Well, of course. Basically, what this boils down to is that the DSDT describes the configuration of your system. It has definitions of all of the devices that ACPI supports, and describes their capabilities. So, for example, it describes the battery, AC Adapter, Embedded Controller, Fans and Thermal Zones. It presents this information in a hierarchical manner, so that ACPI (and therefore the OS) can be aware of dependency relationships among the system hardware. The DSDT is loaded at boot time. It basically tells the ACPI drivers what to watch out for.

## 3. Description of the Problem

The ACPI Specification defines the requirements for the DSDT (and everything else, for that matter) pretty explicitly. Intel's ASL compiler, iasl, used to compile the DSDT to AML from ASL, will throw errors and warnings if the underlying ASL is buggy. Unfortunately, Microsoft's ASL compiler allows many of these errors and warnings to sneak by. As a result, many OEMs write buggy DSDTs, and it turns out that Windows is very forgiving of bugs in the DSDT that get by Microsoft's compiler (not surprisingly).

What this means is that a DSDT that does not conform to the ACPI specification will work under Windows, even though it shouldn't. However, when you try to use it in Linux, where the ACPI developers expect that the DSDT is written to comply with the standard (and the Intel ASL compiler), the buggy sections of the DSDT are unsupported. If you have a buggy DSDT, ACPI may not be aware that certain devices exist. Or, if it is aware, it may not support all of their capabilites. If you have either of these symptoms (missing or incompletely supported functionality in /proc/acpi), then the cause may be a buggy DSDT.

You can tell whether your DSDT was compiled with the Microsoft compiler by the presence of "MSFT" in the DSDT line in dmesg, as below:

```bash
ACPI: RSDP (v000 GBT ) @ 0x000f6d20
ACPI: RSDT (v001 GBT AWRDACPI 0x42302e31 AWRD 0x01010101) @ 0x3fff3000
ACPI: FADT (v001 GBT AWRDACPI 0x42302e31 AWRD 0x01010101) @ 0x3fff3040
ACPI: MADT (v001 GBT AWRDACPI 0x42302e31 AWRD 0x01010101) @ 0x3fff7080
ACPI: DSDT (v001 GBT AWRDACPI 0x00001000 MSFT 0x0100000c) @ 0x00000000
```

Ideally, we could each convince our favorite purveyor of laptops to fix their DSDTs. That's a good idea, but while we're waiting for hell to freeze over, we can implement a workaround to fix our DSDTs ourselves, and provide the fixed DSDT to the linux kernel, making it content, perhaps even happy.

In addition to the DSDT, laptops that conform to the ACPI 2.0 specification may provide an ECDT. This is a small table that provides minimal information about the Embedded Controller to the ACPI drivers before this section is parsed from the DSDT. This is necessary to prevent some chicken and egg type problems with initializing devices (such as the battery and ac adapter in some systems) that rely on the Embedded Controller. Sadly, again, some laptops that should provide an ECDT do not. So, to get around this, we'll have to patch the kernel to read the necessary information from the DSDT.

> Kernel happiness not guaranteed. The author does not take responsibility for surly kernels.

## 4. Disclaimer

The processes outlined here should help you to solve many common problems with ACPI under linux. However, they will not necessarily solve all of your ACPI problems. Unfortunately, there are many causes of ACPI trouble. If after trying these fixes, your ACPI problems persist, then you may have found a bug in the ACPI driver code, which is still under active development. You may also have found a system-related issue of which I am not aware. In either case, your best options are to send an email to the acpi-devel mailing list or to open up a bug at bugzilla. Sometimes, it's best to let the pros handle things.

But enough about what this can't fix. Let's dig into the things that it can. First things first - let's make sure we have a good DSDT.

## 5. Tested Kernels (DSDT override)

The DSDT override procedures outlined in the following sections have been tested successfully with the following kernels:

	2.4.23 vanilla with ACPI patches from acpi.sourceforge.net (static DSDT)
	2.4.24 vanilla (static DSDT, initrd DSDT)
	2.6.0 vanilla (initrd DSDT)
	2.6.0-gentoo (initrd DSDT + bootsplash)
	2.6.1-rc1 vanilla (static DSDT)
	2.6.1 vanilla (initrd DSDT)

## 6. What you'll need to fix your DSDT

Before you get started fixing your DSDT, you will need the following:

	ACPI support compiled into your kernel (static or modules), so you have a /proc/acpi directory.
	The latest ACPI patches from acpi.sourceforge.net (if using vanilla sources).
	The Intel asl compiler, available here.
	and EITHER
	The static DSDT override patch (tested with kernels 2.4.23 and above), available here
	OR
	The initrd DSDT override patch appropriate for your kernel, avaliable here

## 7. Diagnosing a Buggy DSDT

Obviously, fixing your DSDT won't be of any help if it isn't broken. So, the first thing we have to do is find out if it is buggy or not. The easiest way to do this is to recompile the DSDT and see if we get any errors from the compiler. In order to do that, we'll need the iasl compiler and the DSDT to compile. First, let's get the compiler ready:

Get the iasl compiler here.
su to root (not really necessary yet, but you'll need to be root eventually, so why not?)
Extract the iasl compiler to a directory somewhere.
	
```bash	
tar -xvzf iasl-linux-20030228.tar.gz	
```

Enter the newly created directory. You should see a file called iasl. That's the compiler. For simplicity, I'll assume that the rest of the diagnostic steps are done from this directory.
	
```bash	
cd iasl-linux-20030228
ls
>> AslCompiler.pdf     iasl	
```
	
Now we need to get the current DSDT, disassemble it and recompile it to see if we get any errors.
	
Extract your current DSDT to a file as follows:

```bash	
cat /proc/acpi/dsdt > dsdt.dat	
```
	
This will give you a file, dsdt.dat that contains your current compiled dsdt. Now we have to disassemble it so we can recompile it. To do that, we'll use iasl.
	
Disassemble the DSDT

```bash
./iasl -d dsdt.dat	
```

This will create a file called dsdt.dsl, which contains the disassembled DSDT. Have a look at it if you like. You can view its contents with any text editor. Now we need to recompile it, again with iasl.
	
Recompile the DSDT.

```bash	
./iasl -tc dsdt.dsl	
```

This will create two files, dsdt.hex and DSDT.aml.

That's it! If your DSDT is buggy, then you'll see some errors and/or warnings when you recompile. For instance, on my laptop, I got the following messages when recompiling:

```bash
Intel ACPI Component Architecture
ASL Optimizing Compiler / AML Disassembler version 20030228 [Feb 28 2003]
Copyright (C) 2000 - 2003 Intel Corporation
	
Supports ACPI Specification Revision 2.0b
	
dsdt.dsl   163:     Method (\_WAK, 1, NotSerialized)
Warning  2026 -                ^ Reserved method must return a value (\_WAK)

dsdt.dsl  2626:                     Field (ECR, DWordAcc, Lock, Preserve)
Error    1048 -                              ^ Host Operation Region requires ByteAcc access
	
dsdt.dsl  2672:                     Method (\_GLK, 1, NotSerialized)
Warning  2024 -                                ^ Reserved method has too many arguments ( \_GLK requires 0)
	
ASL Input:  dsdt.dsl - 3759 lines, 123154 bytes, 1862 keywords
Compilation complete. 1 Errors, 2 Warnings, 0 Remarks, 390 Optimizations
```	

One error, two warnings. In my case, the error was the source of my trouble. So, if we can fix that up, it should help. At this point, you're done diagnosing the problem. If you got errors from recompiling, then you have a buggy DSDT. The next step is to fix it.

## 8. Fixing the DSDT

The first thing that you should do if you have a buggy DSDT is to head over to the DSDT repository at acpi.sourceforge.net. They have fixed DSDTs for a number of laptops, so your model may already have a fix available. If there is a fix there, then just download it, extract it, recompile the asl using iasl as above, and proceed to Section 9. If there isn't already a fixed DSDT available, then you will have to try to fix your DSDT yourself. Read on for an example from my machine.

Unfortunately, this step will differ for every system. In my case, the fix was simple and self-evident, but I can't guarantee that that will be the case with you. You can find solutions to common DSDT compilation errors here. At the end of the document, I'll have a list of useful resources in case you get stuck. OK - on to the fixing!

In order to fix the DSDT, you'll have to edit the dsdt.dsl file that we created in the diagnosis. Let's use mine as an example. Unless you also have a Gateway 200X, your process will be different (and if you do have a 200X, then you can get the fixed DSDT here). However, this should at least give you an overview of the process.

As you may recall, when I compiled my DSDT, I got one error:

```bash
dsdt.dsl  2626:                     Field (ECR, DWordAcc, Lock, Preserve)
Error    1048 -                              ^ Host Operation Region requires ByteAcc access
```	

This tells me the following:

- The error is on line 2626
- The region in question requires ByteAcc Access

Since I see that this line has a DWordAcc value specified, I assume that that is what is causing the problem. So, I opened dsdt.dsl in a text editor and fixed that line:

Edit dsdt.dsl

```bash	
vi dsdt.dsl	
```

Change any lines that are raising errors. I changed line 2626 from this

```bash	
	Field (ECR, DWordAcc, Lock, Preserve)	
```	
to this

```bash
	Field (ECR, ByteAcc, Lock, Preserve)	
```
	
Save and close the file. Now that we've made the change, we have to recompile to see if we actually fixed the problem.
	
Recompile dsdt.dsl

```bash
./iasl -tc dsdt.dsl	
```

Now, when I recompile, I get the following:

```bash
Intel ACPI Component Architecture
ASL Optimizing Compiler / AML Disassembler version 20030228 [Feb 28 2003]
Copyright (C) 2000 - 2003 Intel Corporation
Supports ACPI Specification Revision 2.0b
	
dsdt.dsl   163:     Method (_WAK, 1, NotSerialized)
Warning  2026 -                ^ Reserved method must return a value (_WAK)
	
dsdt.dsl  2672:                     Method (_GLK, 1, NotSerialized)
Warning  2024 -                                ^ Reserved method has too many arguments ( _GLK requires 0)
	
ASL Input:  dsdt.dsl - 3759 lines, 123153 bytes, 1862 keywords
AML Output: DSDT.aml - 14600 bytes 499 named objects 1363 executable opcodes

Compilation complete. 0 Errors, 2 Warnings, 0 Remarks, 390 Optimizations
```

Alright! The error is gone. The warnings are still there, though. Let's get rid of them now, too.

Repeat steps 1-3 for the warnings

The second warning seems to be the easier one:

```bash
dsdt.dsl  2672:                     Method (_GLK, 1, NotSerialized)
Warning  2024 -                                ^ Reserved method has too many arguments ( _GLK requires 0)
```	
	
So, the _GLK method has too many arguments. Let's get rid of them. We can fix this by changing line 2672 from this:

```bash
Method (_GLK, 1, NotSerialized)	
```

to this:

```bash
Method (_GLK)	
```	
	
Recompiling now only gives me the one warning:

```bash
dsdt.dsl   163:     Method (_WAK, 1, NotSerialized)
Warning  2026 -                ^ Reserved method must return a value (_WAK)
```	
	
From this HOWTO, I see that the solution is to add the following line to the end of the _WAK method:

```bash
Return(Package(0x02){0x00, 0x00})	
```	
	
What does this line mean? A little digging into the ACPI spec (Section 7.3.5) yields some information about the _WAK method.
	
```bash
Arguments:
0   The value of the sleeping state (1 for S1, 2 for S2, and so on).
Result Code (2 DWORD package):
Status   Bit field of defined conditions that occurred during sleep.
   0x00000000   Wake was signaled and was successful
   0x00000001   Wake was signaled but failed due to lack of power.
   0x00000002   Wake was signaled but failed due to thermal condition.
   Other   Reserved
PSS   If non-zero, the effective S-state the power supply really entered.

This value is used to detect when the targeted S-state was not entered because of too much current being drawn from the power supply. For example, this might occur when some active devices current consumption pushes the systems power requirements over the low power supply mark, thus preventing the lower power mode from being entered as desired.	
```	
	
OK, so the _WAK method accepts one argument, which is the number of the sleep state that was requested. It returns its result as a package of 2 DWORDs. The first value is a code that tells whether the wake was successful (0 on success, nonzero on failure) and, if not, why. The second value is also zero on success and on failure returns the value of the sleep state that was actually entered. So basically, it's a success/failure code.
	
In Chapter 16, we can find out how to define a Package.

```bash
PackageTerm   := Package(
                          NumElements  //Nothing |
                                       //ByteConstExpr |
                                       //TermArg=>Integer
                        ) {PackageList} => Package
```	
	
The first argument of the package declaration specifies the number of elements in the package, and the second is the package itself. So, the declaration above simply defines a two element package, where each of the elements is zero. This is necessary because the spec requires that the _WAK method return two values.
	
So, what this really boils down to is a dummy return value that satisfies the spec (thus eliminating the warnings), but doesn't really do anything. It just always returns a "Success" condition.
	
OK, now that that is cleared up, I add that line, recompile, and.... (drum roll please)
	
```bash
Intel ACPI Component Architecture
ASL Optimizing Compiler / AML Disassembler version 20030228 [Feb 28 2003]
Copyright (C) 2000 - 2003 Intel Corporation
Supports ACPI Specification Revision 2.0b
	
ASL Input:  dsdt.dsl - 3760 lines, 123177 bytes, 1863 keywords
AML Output: DSDT.aml - 14606 bytes 499 named objects 1364 executable opcodes
	
Compilation complete. 0 Errors, 0 Warnings, 0 Remarks, 392 Optimizations
```

Excellent! No errors, no warnings. We now have a "fixed" DSDT (remember, the \_WAK method isn't really fixed, we've just shut up the warning on compile). Many of the suggestions in the DSDT HOWTOs that I found are really just workarounds, not proper fixes. If you would like a more thorough analysis of your DSDT, you may want to ask the folks on the acpi-devel mailing list. If you succesfully used this process to fix your DSDT, please consider posting the fix to the DSDT repository, so that others can benefit from your work.

All that remains now is to convince our kernel to use the new DSDT.

As stated above, this will create two files, dsdt.hex and DSDT.aml. You will need to use one or the other of these files in the next step, depending on which method you use to override your DSDT. If you use the static DSDT override method, then you will need dsdt.hex. If you use the initrd method, then you will need DSDT.aml.

## 9. Incorporating the fixed DSDT into the kernel

There are two ways to incorporate your new DSDT into the kernel. The first way is to include it statically at compile time. The second is to pass it to the kenel at boot time as an initrd. The initrd approach is probably preferable, especially if you need to make a lot of changes to your DSDT, because it doesn't require that you recompile your kernel for each new DSDT. The static method does. Each method requires a kernel patch. Let's start with the static method first:

### 9a. Static DSDT override

To statically override your DSDT at kernel compile time, you will have to apply a patch to your kernel to have it read in the new DSDT, and then copy your fixed DSDT .hex file (dsdt.hex) to the kernel source tree for inclusion in the kernel.

So, first things first, let's patch the kernel.

* Obtain DSDT override patch from here
* Save the file to your machine. Let's call it dsdt_override.diff.
* Patch your kernel. I'm assuming that the kernel sources are located in /usr/src/linux-2.4.23, since that's the version I used. If yours are elsewhere, then modify the paths accordingly.

```bash
cd /usr/src/linux-2.4.23
patch -p1 < /path/to/dsdt_override.diff
```	
	
If you don't get any errors, then the patch succeeded. The patch expects to find the fixed DSDT in the linux source tree, under includes/acpi. It expects it to be named dsdt_table.h. So, let's give it what it wants.
	
Copy dsdt.hex to the kernel source tree.

```bash
cp /path/to/dsdt.hex /usr/src/linux-2.4.23/include/acpi/dsdt_table.h	
```	

Recompile your kernel.

That ought to do it. After recompiling, move your new kernel image to /boot (maybe don't overwrite your current kernel - just to be safe), and reboot. If any of your acpi problems were caused by the buggy DSDT, then they should be fixed. /proc/acpi/dsdt should now contain your new fixed DSDT, so you can always cat that back out and recompile it to make sure that you are getting what you expect (i.e. no errors).

Remember, if you make more changes to your DSDT, then you'll have to recopy it to `LINUX_ROOT`/include/acpi/dsdt\_table.h and recompile your kernel.

### 9b. initrd DSDT override

The initrd method requires about the same amount of setup as the static method for the initial DSDT, but any subsequent changes can be incorporated much more easily. Basically, you will have to patch your kernel, copy your fixed DSDT .aml file (DSDT.aml) to /boot, and direct the kernel to incorporate it as an initrd. If you are already using an initrd for something, then there is a bit more work involved. I'll go over that after describing the basic case.

So, again, the first thing we have to do is patch the kernel.

* Get the appropriate initrd patch for your kernel from here
* Apply the patch to your kernel (in my case, I tested this on 2.6.0)

```bash	
cd /usr/src/linux-2.6.0
patch -p1 < /path/to/acpi-dsdt-initrd-patch-v0.4-2.6.0.diff	
```

Make sure that ramdisk and initrd support are compiled statically into the kernel

```bash
Device Drivers --->
    Block Devices --->
        <*> RAM disk support
        [*] Initial RAM disk (initrd) support
```

Make sure that the new "Read DSDT from initrd" option is selected under the ACPI menu

```bash
Power management options (ACPI, APM) --->
    ACPI (Advanced Configuration and Power Interface) Support --->
        [*] Read DSDT from initrd
```

Recompile the kernel
	
Great! Now the kernel is ready to accept a DSDT as an initrd. Next, we need to copy the fixed DSDT.aml file to /boot and edit our grub.conf to point to the DSDT.
	
Copy the fixed DSDT.aml to /boot:

```bash
mount /boot
cp /path/to/DSDT.aml /boot
```

Edit grub.conf to use the new DSDT as an initrd. For example, mine looks like this:

```bash
title=Gentoo Linux (2.6.0 - DSDT initrd)
root (hd0,5)
kernel (hd0,5)/boot/linux-2.6.0-dsdt-initrd root=/dev/hd8
initrd=/boot/DSDT.aml	
```	

The important line is the "initrd=" line. Please remember to use your actual paths for root and kernel - don't just copy this into your grub.conf verbatim.

When you reboot, the new kernel should pick up the DSDT from the /boot partition and load it up as directed by the initrd parameter. You should see a message in dmesg that says "Looking for DSDT in initrd ..."

Now, if you make changes to your dsdt, all you have to do is copy the new DSDT.aml to /boot and reboot to incorporate those changes. No kernel recompile required.

### 9c. initrd override with bootsplash

If you are already using an initrd for something, like bootsplash, you can still use this method. You just have to create the initrd a bit differently. Instead of simply copying DSDT.aml to /boot and using it as the initrd, you have to cat a signature for the DSDT into your existing initrd, and then cat the DSDT into it as well. For example, my bootsplash initrd is currently called initrd-1024x768. So, here's what I did to add the DSDT.

Make a copy of /boot/initrd-1024x768 for use with the DSDT

```bash
cp /boot/initrd-1024x768 /boot/initrd-1024x768-dsdt	
```

Append the DSDT signature to the initrd

```bash	
echo "INITRDDSDT123DSDT123" >> /boot/initrd-1024x768-dsdt	
```

cat the fixed DSDT.aml into the initrd

```bash
cat /path/to/DSDT.aml >> /boot/initrd-1024x768-dsdt	
```

Modify grub.conf to use the new initrd on boot:

```bash
title=Gentoo Linux (2.6.0 gentoo - bootsplash + DSDT initrd)
root (hd0,5)
kernel (hd0,5)/boot/linux-2.6.0-gentoo-dsdt root=/dev/hd8 video=vesa:ywrap,mtrr vga=0x317
initrd=/boot/initrd-1024x768-dsdt	
```

Voila! When you boot up, you should now get your lovely bootsplash screen and your new dsdt, both incorporated from the initrd.

If you make changes to the DSDT, you just have to rebuild your initrd as above (which is a good reason to make a copy of your existing one, rather than appending to it directly).

## 10. My DSDT is fixed, but I still have ACPI errors. Now what?

### 10a. Windows-only DSDT functionality

You may find that you have no errors in your DSDT, but there are still errors in dmesg, or missing ACPI functionality. This may be because your DSDT is testing for the name of your OS. Many DSDTs do this, and enable certain functionality only if you are running a particular OS (usually, of course, Windows XP). To test for this, look for lines in your DSDT that check the value or length of the "\_OS" variable. For example, you may find lines like this:

```bash
If (LEqual (SizeOf (\_OS), 0x14))
```

This is checking for an OS name with a length of 20 (0x14) characters. Some examples are "Microsoft Windows NT" or "Microsoft Windows XP". You could try to rewrite your DSDT to skip these checks, or to provide the missing functionality for other OSes, but this is tedious and error-prone. Fortunately, there is a simpler way. There is a boot parameter that you can pass to the kernel to tell ACPI to pretend that you are running Windows to restore the missing functionality, rather than rewriting your DSDT. The parameter is called acpi\_os\_name. So, in grub.conf, you would just add this parameter to the kernel line, like this:

```bash
acpi_os_name="Microsoft Windows XP"
```

This should restore any functionality that is dependent on your OS identifying itself as Windows XP. Other common \_OS length checks are:

`0x11: "Microsoft Windows"`

`0x27: "Microsoft WindowsME: Millennium Edition"`

__NOTE__

These tests are also sometimes used to disable certain functionality for certain Windows OSes. So, you may want to take a few minutes to try to work out what your DSDT is doing before fudging the OS name. You should probably take note of what works and what doesn't before and after trying this, just in case you end up turning something off.

### 10b. Missing ECDT

If your problem isn't that the DSDT is checking whether or not you are running Windows, then you may have a missing ECDT. The ECDT is used to provide some minimal information about the Embedded Controller to the ACPI drivers before the Embedded Controller region has actually been parsed from the DSDT. This is frequently necessary before initializing the battery and ac adapter, so if you have errors in dmesg from the battery, adapter or EmbeddedControl, the ECDT is the likely culprit. For instance, after fixing my DSDT, I had the following errors in dmesg:

```bash
evregion-0251 [22] ev\_address\_space\_dispa: No handler for Region [ECR\_] (df5ec688) [EmbeddedControl]
 exfldio-0284 [21] ex\_access\_region      : Region EmbeddedControl(3) has no handler
 dswexec-0435 [14] ds\_exec\_end\_op        : [LEqual]: Could not resolve operands, AE\_NOT\_EXIST
dswstate-0273 [16] ds\_result\_pop\_from\_bot: No result objects! State=df5fb428
 dsutils-0525 [16] ds\_create\_operand     : Missing or null operand, AE\_AML\_NO\_RETURN\_VALUE
 psparse-1120: \*\*\* Error: Method execution failed [\_SB\_.ADP1.\_STA] (Node df5f42c8), AE\_AML\_NO\_RETURN\_VALUE
evregion-0251 [22] ev\_address\_space\_dispa: No handler for Region [ECR\_] (df5ec688) [EmbeddedControl]
 exfldio-0284 [21] ex\_access\_region      : Region EmbeddedControl(3) has no handler
 dswexec-0435 [14] ds\_exec\_end\_op        : [LEqual]: Could not resolve operands, AE\_NOT\_EXIST
dswstate-0273 [16] ds\_result\_pop\_from\_bot: No result objects! State=df5fb428
 dsutils-0525 [16] ds\_create\_operand     : Missing or null operand, AE\_AML\_NO\_RETURN\_VALUE
 psparse-1120: \*\*\* Error: Method execution failed [\_SB\_.BAT1.\_STA] (Node df5f4848), AE\_AML\_NO\_RETURN\_VALUE
```	

EmbeddedControl is obviously the embedded controller. ADP1 is the ac adapter and BAT1 is the battery. After booting, my /proc/acpi/ac\_adapter and /proc/acpi/battery directories were empty. All of this was caused by the missing ECDT.

A couple of patches have been proposed to correct this behavior. So far, the most promising one is here. This defers the Embedded Controller's initialization until it has actually been parsed from the DSDT. This patch restored my missing ECDT-related functionality (battery and adapter), and has been reported to do the same for a number of different systems. It may be applied to both the 2.4 and 2.6 kernel sources.

### 10c. My power and/or sleep buttons don't work

The ACPI spec defines two types of power and sleep buttons. They are called fixed-feature (FF) buttons and control method (CM) buttons. The difference between the two is that FF buttons are defined at a lower level than the DSDT, in the FADT. CM buttons are defined in the DSDT. It is possible for your system to have both types of button. You can tell which you have by the presence of an "FF" or "CM" next to the button alert in dmesg. For example, my dmesg gives the following:

```bash
ACPI: Power Button (FF) [PWRF]
ACPI: Lid Switch [LID0]
ACPI: Sleep Button (CM) [SLPB]
```	

So, acpi is detecting an FF power button and a CM sleep button. I actually also have a CM power button defined in my DSDT. However, in accordance with the ACPI spec, if the linux drivers detect an FF button, the CM button is ignored. Unfortunately, the FF power button does not generate an ACPI event in my case. However, the CM sleep button does generate events.

I opened a bug for this behavior. However, it is unlikely that the behavior will change, since it would contradict the spec to give CM buttons precedence over FF buttons. So, I posted a patch to provide a workaround for the problem on the 2.6 kernels.

The patch allows you to provide a new boot parameter, called ignore\_ff\_buttons. It accepts one of three values, PWRF (to ignore only the FF power button), SLPF (to ignore only the FF sleep button), or BOTH (to ignore both FF buttons).

In my case, I just want to ignore the FF power button, since I am already using the CM sleep button. So, I modified my grub.conf as below:

```bash
title=Gentoo Linux (2.6.1 - all ACPI patches)
root (hd0,5)
kernel (hd0,5)/boot/linux-2.6.1 root=/dev/hda8 ignore\_ff\_buttons=PWRF
initrd=/boot/DSDT.aml
```	

Now, when I reboot, dmesg shows me the following buttons:

```bash
ACPI: Lid Switch [LID0]
ACPI: Sleep Button (CM) [SLPB]
ACPI: Power Button (CM) [PWRB]
```

And I get ACPI events from my power button.

__NOTE__

If you decide to use this patch, please be careful. I haven't created a kernel patch before. This one doesn't do much, so it should be pretty safe, but I highly recommend that you back up your kernel tree before applying it. Also, if you do decide to use it, I would be very interested to hear whether or not it works for you. Of course, I'm sure that I'll hear if it doesn't, but if it does, I would also like to know. :)

### 10d. My BIOS is blacklisted

Some BIOSes have been blacklisted by the ACPI developers because they have known problems. These problems are usually related to the DSDT, so you might be able to repair these problems using the above techniques. Unfortunately, if the ACPI code detects that you have a blacklisted BIOS, it will disable ACPI even if you have fixed your DSDT. If you have a blacklisted BIOS, you will see a message like this on boot:

```bash
ACPI: Vendor "PTLTD " System "  DSDT  " Revision 0x6040000 has a known ACPI BIOS problem.
ACPI: Reason: Multiple problems. This is a non-recoverable error
ACPI: BIOS listed in blacklist, disabling ACPI support
```

To get around this, you'll have to comment out the line in the ACPI code that causes your BIOS to short circuit the ACPI startup process. The file that sets the blacklisted BIOSes is /usr/src/linux/drivers/acpi/blacklist.c. To prevent your blacklisted BIOS from causing trouble, open up the file and comment out the appropriate line in static struct acpi\_blacklist\_item acpi\_blacklist[]. For example, in the above case, the line to comment is:

```bash
{"PTLTD ", "  DSDT  ", 0x06040000, ACPI_DSDT, less_than_or_equal, "Multiple problems", 1},	
```	

Once you've commented out the line that corresponds to your BIOS, recompile your kernel and reboot. The ACPI drivers should now initialize, and the messages about the blacklisted BIOS should go away. You may also have to boot with "acpi=force" on the kernel line of your grub.conf to get this to work (thanks to sleek for clarifying).

### 10e. I have an Asus laptop

If you have an Asus laptop, you may want to check out the acpi4asus project. This is a driver that provides ACPI support on linux for most Asus laptops, including special keys and LEDs. The driver is being actively developed, so if you have a recent Asus laptop that isn't yet supported by the driver, support may be coming soon.

### 10f. It's still not working!

If, at this point, you are still having errors, there are one or two more things to try. If ACPI refuses to initialize, you can try booting with "acpi=force" in your grub.conf. This is sometimes necessary in order to force the ACPI drivers to initialize with older BIOSes. If you still have ACPI-related errors on boot and have a single-processor machine, then try disabling multiprocessor support, local apic support and IO-apic support in your kernel. You can also try booting with the "pci=noacpi" or "noapic" options. The apic code still seems to be a bit finicky; many people have reported problems with it.

If none of the above recommendations helps, then I would strongly suggest contacting the ACPI developers. You have likely found a bug in the ACPI code. In particular, if you are having suspend/resume problems, you are far from alone. This code is still being worked out. Give the acpi-devel mailing list a shot. You can also, of course, post your errors here, and I'll try to see what I can do to help, but the acpi gurus are likely to be a better resource.

## 11. Useful ACPI resources

There are a number of good ACPI resources, in case you get stuck. The ones that I found most helpful were:

	bugzilla.kernel.org
	acpi.sourceforge.net
	acpi-devel and acpi-bugzilla mailing lists
	www.acpi.info (the ACPI spec is here, including ASL documentation, which is the language used for the DSDT)
	The Intel ACPI downloads page - get the iasl compiler here.
	HOWTO fix common DSDT errors
	DSDT fixes for the Thinkpad T31
	DSDT repository at acpi.sourceforge.net
	DSDT submission page at acpi.sourceforge.net.
	Documentation for the static DSDT override
	Documentation for the initrd DSDT override
	ECDT initialization patch
	FF button patch
	acpi4asus homepage

## 12. Acknowledgements

I'd like to thank the ACPI developers, especially Luming Yu and Bruno Ducrot. All of them have been very helpful and very responsive. In general, the ACPI mailing lists and bugzilla are wonderful resources for ACPI help.
