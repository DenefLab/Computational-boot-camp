## Basic UNIX commands (one-liners)

### A. navigating and handling files
* ~/ {your home directory}
* cd {change to you home directory}
* cd / {change to top directory}
* mv filename directoryname/ {move filename to other directory}
* mv filename1 filename2 {renames filename1 to filename2}
* mkdir / rmdir {create new / remove directory}
* ls {display non-hidden files, –a to show hidden files also, -lrt to list details + reverse based order based on time-stamp, so most recent last; -F to discriminate files vs. folders etc.}
* {Directory structure contains flags (l = link, d = dir, p = pipe, b = block, c =?) owner (rwx = read write executable) group world
* pwd {where am I}
* clear {clear screen}
* cp filename directoryname/ {copy filename to other directory}
* rm {delete file or directory}
* \*.\* {all files}
* \*.faa {all files with extension .faa}
* wc {get the number of lines, words and characters of a file}
* chown vdenef:geomicr filename {change ownership user:group}
* chmod ug+x {make file executable for user and group; chmod -R =recursive = make change in all files in subdirs as well; can also be applied for dir: if application tries to write in dir and permission is not +r for user:group => error (e.g. Phred, gzip, …}
* su {change user}
* passwd {change password}
* ln -s {create symbolic link from filename1 to filename2 = shortcut e.g. ln -s /usr/bin/gunzip /bin/gunzip}
* more filename {view file contents}
* less filename {~ more; show contents of a file}
* head -# {show first # lines of file}
* tail -# {show last # lines of file}
* gzip / gunzip {compress, uncompress}
* tar {compress whole dir into 1 file}
* fetch (program) or scp {copy between computers}
* {scp filname user@server:filename}
* wget {to get everything from FTP foler at once}
* note: highlighting text copies it into buffer. alt+click = paste at cursor

### b. running software
* -help or –h {display help for command by adding behind command}
* man xxx {display manual pages for command xxx}
* apropos “xxx” {search for program that does xxx}
* | {pipes the output from previous command to next, e.g. ls |more}
* \> outputfile {used to keep output and save in outputfile}
* ps –u <user> {shows processes active under user x}
* kill # {forces quit of process with number # → number is shown using ps –u}
* alias {list of defined alias for certain application, have to be added in bash. Can add by typing e.g. alias UBA=”cd /Volumes/XRAID/….” on commandline}
* export {fix particular alias and other changes made to …}
* For running a lot of scripts and other programs your paths need to be defined. These are stored in .bashrc and other similarly named files.
* time perl xxx {xxx is program, will calculate the time it takes to process}

### c. some other handy shell commands to search and modify files
* grep –i “>” filename {show all lines starting with “>”; Add | more to pause per page}
* grep –c “>” filename {count all lines starting with “>”}
* cat inputfiles outputfile {concatenate files}
* cut –f # xxx {from file xxx, take column # and output. default is tab-delimited, can change by –d “X” with X being the delimiter
* sort ` {sort alphanumerically}
* uniq {remove redundant lines in file}
* dos2unix {remove excel formatting MS-DOS formatted files; also mac2unix} -n -c mac file file.new 
* ssh user@ip.address {secure shell connection to server}

### d. Some specific applications
* Use of screen
Purpose: 
Usage: 
screen {to activate}
Ctrl+a d {to detach a window, this will allow you to disconnect while the process keeps running}
to return screen -r xxx {xxx is a number in case you have multiple at once}

* Use of stream editor (sed)
Purpose: 
to remove or change files throughout, e.g. modify gene identifiers

Usage: 
sed –e ‘s/re/place/g’ inputfile > outputfile.sed {substitute (s) re with place throughout file (g)} {multiple commands possible, each time start with –e {to view but not save changes, do not put outputfile.sed}

‘s/…/ {any 3 characters}

‘/^$/d’ or ‘/\n/d’ {beginning of the line (), if nothing then delete line, so remove empty lines}

‘s/.*/ {any character to end of the line}

{start of line}

$ {end of line}

‘s/$/c/g’ {adds “c” at end of each line}
