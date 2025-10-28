# Android-CTF-writeup


CTF Challenge Write-Up: Android APK Reverse Engineering
Methodology:
1. Setup & Tooling
System: Windows 11 (physical desktop)
IDE: IntelliJ IDEA Ultimate (code navigation, search)
Decompiler: JADX v1.4.7 GUI (APK inspection, quick Java class analysis)
Smali Tool: Baksmali v2.5.2 (smali extraction when APKTool failed)
Scripting: Python 3.11 (flag reconstruction, intermediate calculations)
2. Initial Exploration
Inspected AndroidManifest.xml: Looked for dangerous permissions and notable package names. Found the app's entry point (com.trellix.trellixctf.a.MainActivity) and package (com.trellix.trellixctf). 

Noted Unusual App Structure: The manifest and file structure seemed nonstandard, suggesting deliberate obfuscation.


3. Smali Extraction & Class Mapping
APKTool Failure: Tried APKTool, but it couldn’t extract smali. Learned the value of fallback tools. Used Baksmali to successfully extract readable smali classes.
Navigated to Key Directories: Moved into smali_classes4/com/trellix/trellixctf/b/, following clues from package and class names.

4. Flag Discovery and Direct Encodings
Found Encoded Strings in Smali: In d.smali, a hardcoded Base64 string led, via online tools, to the first flag:

Flag1:YouFoundthis!Findthesecondone?

Second Encoded String: In e.smali, another Base64 string decoded to an intermediate string, which was itself another Base64-encoded flag:

Flag2:Yougothis!Findthelastone?


5. Obfuscated Array Analysis — Critical Path for Flag 3
Noted Suspicious Arrays: Found two arrays (Screenshots below) (from methods q() in a.smali and r() in b.smali). Converted hex values to characters and noted they were likely used as keys or cipher text.
Example: q() → u=28ir@?8C2EDJ@F8@E, r() → 2==E9C66P
Rabbit Hole: Tried to find ciphertext using typical AES decryption logic in smali (from prior experience, expected a key and IV, but code used ECB mode which meant no IV needed).
False Leads: Couldn’t find obvious ciphertext, so pivoted back to tracing array usage, which I originally thought would give me the key and I could go back to looking for the cipher text after I had obtained the key.

6. Transformation Logic and Script
Tracked Usage to f.smali: Found both arrays concatenated and passed through a custom character transformation in method i().
Decompiled and understood the Transformation logic and then made a script to replicate the transformations in python










The transformation logic used in this challenge is a custom ASCII shift cipher applied to printable characters. For each character between '!' (ASCII 33) and '~' (ASCII 126), the process subtracts 33 to create a zero-based index, then shifts it forward by 47 and wraps within the 94 printable characters using modulo arithmetic. Finally, it adds 33 back to restore the result to the printable range. This makes each character cycle forward 47 steps in the printable ASCII table, wrapping around from the end back to the start if needed. For example, the letter 'A' (ASCII 65) becomes 'p' after transformation, because 
((65−33)+47)mod  94+33=112
((65−33)+47)mod94+33=112 which is 'p'. Non-printable or out-of-range characters are left unchanged

Final Discovery: After transformation, the flag revealed itself:
Flag:Congratsyougotallthree!


7. Task Analysis 
This APK used a blend of simple encoding (Base64 strings) and multi-step obfuscated static data (arrays with printable ASCII transformation) to hide all three flags.
The challenge required careful static analysis, mapping references, and patience to test out ambiguous paths, especially when initial leads didn’t produce results.
By following the code, not assuming anything was “obvious,” and documenting dead ends (like searching for an IV or obvious ciphertext), I learned the importance of thoroughness in CTF reversals.




8. Notes & Learning Experiences
Spent time mapping out IV usage for cipher routines, only to learn that ECB mode required none. This taught me to always confirm the cipher mode before spending precious time reconstructing missing variables.
Initial search for ciphertext in smali classes yielded nothing; only after focusing on how data was assembled did the solution reveal itself.
Always map out data transformations and test your assumptions with small code snippets or sanity checks. This is crucial for crypto problems or obfuscated static data.
When referencing files or logic, avoid making assumptions to prevent tunnel vision and unnecessary deep dives.
9. Environment
Operating System: Windows 11 Pro, physical PC
Development Environment: IntelliJ IDEA Ultimate, default config
Decompiler: JADX v1.5.3 GUI
Smali Extraction: Baksmali v3.0.9 
Scripting: Visual Studio Code Python 3.12.7 
Tools: APKTool (dead end), online Base64 decoder, VSCode 
