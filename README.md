# Bypassing-Filter-to-Traversal-Attacks
Article that out lines bypassing directory traversal
 Bypassing Filter to Traversal Attacks

If your initial attempts to perform a traversal attack, as described previously, are unsuccessful, this does not mean that the application is not vulnerable. Many application developers are aware of path traversal vulnerabilities and implement various kinds of input validation checks in an attempt to prevent them. However, those defenses are often flawed and can be bypassed by a skilled attacker.

The first type of input filter commonly encountered involves checking

whether the filename parameter contains any path traversal sequences, and if so, either rejects the request or attempts to sanitize the input to remove the sequences. This type of filter is often vulnerable to various attacks that use alternative encodings and other tricks to defeat the filter. These attacks all exploit the type of canonicalization problems faced by input validation mechanisms

Always try path traversal sequences using both forward slashes and

backslashes. Many input filters check for only one of these, when the file system may support both.

Try simple URL-encoded representations of traversal sequences, using

the following encodings. Be sure to encode every single slash and dot

within your input:

dot                            %2e

forward slash           %2f

backslash                  %5c

Try using 16-bit Unicode–encoding:

dot                           %u002e

forward slash          %u2215

backslash                %u2216

Try double URL–encoding:

dot                        %252e

forward slash         %252f

backslash                %255c

Try overlong UTF-8 Unicode–encoding:

dot                        %c0%2e       %e0%40%ae    %c0ae etc.

forward slash        %c0%af       %e0%80%af      %c0%2f etc.

backslash              %c0%5c       %c0%80%5c      etc.

You can use the illegal Unicode payload type within Burp Intruder to generate a huge number of alternate representations of any given character, and submit this at the relevant place within your target parameter. These are representations that strictly violate the rules for Unicode representation but are nevertheless accepted by many implementations of Unicode decoders, particularly on the Windows platform.

If the application is attempting to sanitize user input by removing traversal sequences, and does not apply this filter recursively, then it may be possible to bypass the filter by placing one sequence within another. For example:

....//

....\/

..../\

....\\

The second type of input filter commonly encountered in defenses against path traversal attacks involves verifying whether the user-supplied filename contains a suffix (i.e., file type) or prefix (i.e., starting directory) that the application is expecting.

Some applications check whether the user-supplied file name ends in a

particular file type or set of file types, and reject attempts to access anything else. Sometimes this check can be subverted by placing a URL encoded null byte at the end of your requested filename, followed by a file type that the application accepts.

For example:

../../../../../boot.ini.jpg

The reason this attack sometimes succeeds is that the file type check

is implemented using an API in a managed execution environment

in which strings are permitted to contain null characters (such as

String.endsWith() in Java). However, when the file is actually retrieved, the application ultimately uses an API in an unmanaged environment in which strings are null-terminated and so your file name is effectively truncated to your desired value.

A different attack against file type filtering is to use a URL-encoded newline character. Some methods of file retrieval (usually on Unix-based platforms) may effectively truncate your file name when a newline is encountered:

../../../../../etc/passwd%0a.jpg

Some applications attempt to control the file type being accessed by

appending their own file type suffix to the filename supplied by the user. In this situation, either of the preceding exploits may be effective, for the same reasons.

Some applications check whether the user-supplied file name starts with a particular subdirectory of the start directory, or even a specific file name. This check can of course be trivially bypassed as follows:

wahh-app/images/../../../../../../../etc/passwd

If none of the preceding attacks against input filters are successful individually, it may be that the application is implementing multiple types of filters, and so you need to combine several of these attacks simultaneously (both against traversal sequence filters and file type or directory filters). If possible, the best approach here is to try to break the problem down into separate stages. For example, if the request for

diagram1.jpg

is successful, but the request for

foo/../diagram1.jpg

fails, then try all of the possible traversal sequence bypasses until a variation on the second request is successful. If these successful traversal sequence bypasses don’t enable you to access /etc/passwd, probe whether any file type filtering is implemented and can be bypassed, by requesting

diagram1.jpg.jpg

Working entirely within the start directory defined by the application, try to probe to understand all of the filters being implemented, and see whether each can be bypassed individually with the techniques described.

Of course, if you have white box access to the application, then your task is much easier, because you can systematically work through different types of input and verify conclusively what filename (if any) is actually reaching the file system.



Copied from: https://tipstrickshack.blogspot.com/2013/02/how-to-bypassing-filter-to-traversal_8831.html
