-*-outline-*-
* see also
Voir également file:/home/ychaouche/DOCUMENTS/DOCUMENTATION/bashcondense.odt
Voir également file:/home/ychaouche/NOTES/MAPS/PUBLIC/bash.vym
Voir également ~/howm/2018/04/2018-04-22-140543.txt
the power of aliases : https://www.chiark.greenend.org.uk/~sgtatham/aliases.html
* online ressources
** https://www.shellcheck.net/
Trouver des bugs dans des scripts shell
** https://explainshell.com/
Explains a command line



* Quoting inside aliases
1. enclose aliases in double quotes
alias mailconnect=""

2. grep/sed : use simple quotes
alias mailconnect="something something | grep 'pattern'"

3. awk : escape doublequotes and $ with a backslash
alias mailconnect="something something | awk '{printf \"something something\", \$1, \$2}'"
* programming
** write code
*** variables
**** increment a variable
((i++))
you don't even need to initialize the variable to zero for this to work.
ychaouche#ychaouche-PC 11:28:36 ~/DATA/BACKUPS/WIKIPAGES-505 $ ((i++))
ychaouche#ychaouche-PC 11:28:42 ~/DATA/BACKUPS/WIKIPAGES-505 $ echo $i
1
ychaouche#ychaouche-PC 11:28:44 ~/DATA/BACKUPS/WIKIPAGES-505 $
**** global variables
use them at will
for example, in functions, you can use $RETURN or $RET

**** local variables
just use local
local foo=32
**** BASH variables
***** $FUNCNAME
useful in usage messages : 

usage : $FUNCNAME <args>

use this instead of $0
***** $*
all arguments into a single string
***** $@
all arguments separatly
***** ~+ ~-
$PWD and $OLDPWD

**** default values
${1:-30} # use $1 or 30
${var:=20} # assign 20 to var if it's empty or unset
**** conditional values 
${1:+30} # expands to 30 if $1 is set and non null
**** indirections
***** straightforward access
var=A
A=2
${!var} # will evalutate to 2
***** used in loops
$ name="Colin MacDonald"
$ city="Arlington"
$ state="VA"
$ echo $name
Colin MacDonald
$ vars="name city state"
$ for x in $vars ; do echo "$x=${!x}" ; done
name=Colin MacDonald
city=Arlington
state=VA


**** reading variables from config files
just source the file and you can now access its variables
**** IFS
IFS only works with $* and $@
so only for command line arguments
or function arguments
**** exit if variable is null or not set
${var:?}
only exits non interactive shells

17:49:32 ~/IMAGES/MEMES -1- $ ${hah:?}
bash: hah: parameter null or not set
17:49:41 ~/IMAGES/MEMES -1- $

**** show variable values
declare -p var1, var2, var3
this will print the variable name and value
*** collections
**** Normal arrays
***** store a command in an array
grep_command=(zgrep -h --line-buffered --color "$grep_opts" "$pattern" "$filename");
echo "grep command : ${grep_command[@]}";
"${grep_command[@]}";
***** store the results of a command in an array
mapfile -t ARRAY < <(command)
for example:
mapfile -t pages < <(pup 'strong text{}' < $htmlfile)

don't do
command | mapfile -t array

because the array is created in the subprocess in the pipe,
then disappears.

***** creating an array
A=(1,2,3,4)
B=(/path/to/dir/*) # each file in its own cell
***** adding elements
HOSTS=()
HOSTS+=(192.168.5.50)
HOSTS+=(10.250.20.30)
HOSTS+=(192.168.110.220)
HOSTS+=(192.168.110.123)
***** number of elements
${#HOSTS[@]}
***** looking for an element
for i in ${array[@]}
do
  if [[ $i == value ]]
  then
     return 0
  fi
done
return 1
***** passing as argument to functions
searcharray 3 "${array[@]}"

function searcharray {
  searchfor=$1
  shift
  array=("$@")
  ....
}

voir .bashrc_common > searcharray

***** looping through an array
for element in ${array[@]}
do
   something with $element
done
***** reading from file into array
use mapfile [array]
puts each line of stdin in array(default=MAPFILE)
-t[rim] will trim newlines.
-u FD will read from FD instead of stdin
example:
mapfile -t results < <(dig +short $revip.zen.spamhaus.org @10.10.10.7)

each line from the output of dig,
eventually containing a space,
will be assigned to results' cell.

don't do
command | mapfile -t array

because the array is created in the subprocess in the pipe,
then disappears.



***** slices
15:45:31 ~ -1- $ declare -p array
declare -a array='([0]="WM_NAME(STRING)" [1]="=" [2]="\"HikarinoFurusato_IchikoAoba.mp3" [3]="-" [4]="VLC" [5]="media" [6]="player\"")'
15:45:54 ~ -1- $

15:46:39 ~ -1- $ echo ${array[@]:2:6}
"HikarinoFurusato_IchikoAoba.mp3 - VLC media player"
15:50:20 ~ -1- $ 

***** printing
${array[@]} -> splits to $1 $2 $3...
or 
${array[*]} -> transforms to a single string ($1)

or to inspect/debug from the command line
declare -p 
15:45:31 ~ -1- $ declare -p array
declare -a array='([0]="WM_NAME(STRING)" [1]="=" [2]="\"HikarinoFurusato_IchikoAoba.mp3" [3]="-" [4]="VLC" [5]="media" [6]="player\"")'
15:45:54 ~ -1- $

**** Associative arrays
***** looping over the array
****** looping over keys
for key in ${!array[@]}; do ...; done;
****** looping over values
for value in ${array[@]}; do ...; done;
***** adding values
declare -A[ssociative] array
array[color]=yellow
echo ${array[color]}
***** deleting values
unset ${array[key]}
***** testing if a key exists
${array[key]+_} will be replaced with "_" if the key exists
***** passing as argument to functions
declare -A myassocarray
myassocarray[key]=value
...

function foo() {
  local -n array=$1
  echo array[key] # value
}


array is a local variable that is a reference to myassocarray.

*** tests
**** tests on files
***** exists
[ -e file ] or [ -a file]
***** is readable
[ -r file ] tests if the file is readable
***** is not a symlink
[[ -f $file ]] && [[ ! -L $file ]]
$? should return 0
***** is newer than another file
if [[ file1 -nt file2 ]]

***** test if file is empty
if [ -s file ]
true when file not empty
false when file is empty
if [ -s file ]
then
  echo file is not empty
else
  echo file is empty
**** tests on bash
***** is interactive (useful for ssh)
tests that work :
 - checking for PS1

 if [ "$PS1" ] ; then
  # shell is interactive
 fi

 - checking for the -i option

This is found i my .bashrc file

# Test for an interactive shell.  There is no need to set anything
# past this point for scp and rcp, and it's important to refrain from
# outputting anything in those cases.
# if [[ $- != *i* ]] ; then
# 	# Shell is non-interactive.  Be done now!
# 	return
# fi

 
***** test for a bash option
bash options are stored in $-
but you can test for a particular option with [ -o ]
for example:

 if [ -o i] ; then
  # shell is interactive
 fi


**** if not :: negative tests 
if ! ..
**** test the existence of
***** a file
[ -e file ] or [ -a file]
[ -r file ] tests if the file is readable
***** a variable
[[ -v var ]] && echo \$var exists
***** an alias
if type alias; then echo found; else echo not found; fi

example : 

ychaouche#ychaouche-PC 10:58:41 ~ $ if type desktop.clipboard.set; then echo fou
desktop.clipboard.set is aliased to `desktop.clipboard.put'
found
ychaouche#ychaouche-PC 10:58:45 ~ $ if type desktop.clipboard.sets; then echo fo
bash: type: desktop.clipboard.sets: not found
not found
ychaouche#ychaouche-PC 10:58:49 ~ $

**** test for the number of arguments
if [[ "$#" < 1 ]]
then
    echo "usage : $0 fichier.csv"
    echo "le fichier doit contenir une ligne par compte."
    echo "chaque ligne contient nom prénom, séparés par une virgule (,)."
    exit 1
fi

**** test for an empty/non empty string
[[ -z $string ]] tests if the string is empty
[[ -n $string ]] tests if it is not empty
**** test for 0
***** if it's a command
Don't test for 0 if you want to test if the last command succeeded, just do

if command;
then
  do this;
else
  do that;
fi;



***** if it's a variable
use the arithmetic expression
remember that then is executed if condition returns 0 (which is true)
and else is executed for any value that's not 0.

if ((val))  # no need for $ in arithmetic expressions,
then	    # it is implicit
   not zero # b/c you can't have strings in there
else
   zero
fi
**** if var=$(command)
if var=$(command)
then
   <do things with var>
else
  <command failed>
fi
*** strings
**** removing
***** remove at the beginning
****** remove first word
remove everything before first whitespace
${string# *}
****** remove n first chars
echo ${string:x}

10:28:09 ~ -1- $ str="Hello world!"
11:10:07 ~ -1- $ echo ${str:3}
lo world!
11:10:13 ~ -1- $
****** remove pattern if it is at the beginning
${var#/pat}
ou bien
${var#pat}
***** remove at the end
****** remove a file extension
remove file extensions : ${filename%.*}
****** remove last word
remove everything after last space
${string% *}

****** remove "x" at the end of a string
${string%%x*}

example : 
ID="7540C3A81C9B!"
nID="${ID%%\!*}"
echo "$ID -> $nID" # 7540C3A81C9B! -> 7540C3A81C9B
****** remove n last chars
echo ${string:0:-x}
****** remove pattern if it is at the end
${var%/pat}
***** remove once
****** remove first occurence of pattern
${var/pat}
***** remove anywhere
****** remove all spaces from a string
******* with ${var// }
${var// } # marche

# from newusers.from.csv
email="${prenomminuscules// }.${nomminuscules// }@algerian-radio.dz"


${var//[[:blank:]]}
******* with echo
echo $string va enlever tous les espaces

ychaouche#ychaouche-PC 15:17:17 ~ $ echo "$prenom" b
je suis charlie       b
ychaouche#ychaouche-PC 15:17:20 ~ $ prenomwos=$(echo $prenom) # wos : w/o space
ychaouche#ychaouche-PC 15:17:36 ~ $ echo "$prenomwos" b
je suis charlie b
ychaouche#ychaouche-PC 15:17:42 ~ $

${string%?????} repeat ? x times
****** remove all occurences of pattern
${var//pat}        
***** left trimming
longest string : ${string##*x}
shortest string : ${string#*x}
***** right trimming
longest string : ${string%%x*}
shortest string : ${string%x*}
remove file extensions : ${filename%.*}
**** case transformation
${mastring^}      change le premier character en majuscules
${mastring^^}     change toute la chaine en majuscules
${mastring,}      change le premier character en minuscules
${mastring,,}     change toute la chaine en minuscules
${mastring~}      inverse maj/min du premier char
${mastring~~}     inverse maj/min de toute la chaine

${mastring^l}     change le premier l en capitales
${mastring,l}     change le premier l en minsucules
${mastring^^l}    change tous les l en capitales
${mastring,,l}    change tous les l en minsucules
${mastring,,[l,h]}change tous les l et les h en minsucules
**** matching
***** regex matching
regex matching with =~
[[ "string" =~ regex ]]

What's on the left is a string.
Quotes are removed
What's on the right is a regex
If the regex is quoted it will be treated as a string instead.
***** glob matching
glob matching with =
if [[ $- = *i* ]]

the last * is mandatory
the glob expression has to cover all of the string
***** matching spaces
[[ $string =~ (\ ) ]]
***** example 1 : detecting spaces in first names
# detects if there is a space in prenoms
# parenthesis are optional
if [[ "$prenoms" =~ (\ ) ]]
***** example 2 : detecting parens in file paths

root@messagerie-principale[10.10.10.19] ~ # [[ "(/usr/local/scripts/mail/mail.user.quota)" =~ "(.*)" ]] && echo yes  || echo no
no
root@messagerie-principale[10.10.10.19] ~ # [[ "(/usr/local/scripts/mail/mail.user.quota)" =~ (.*) ]] && echo yes  || echo no
yes
root@messagerie-principale[10.10.10.19] ~ # [[ (/usr/local/scripts/mail/mail.user.quota) =~ (.*) ]] && echo yes  || echo no
-bash: syntax error in conditional expression
-bash: syntax error near `=~'
root@messagerie-principale[10.10.10.19] ~ #

**** replacing
${var/pat/string}  replace one occurence
${var//pat/string} replace all occurences


example :

ychaouche#ychaouche-PC 15:55:27 ~/WTMP $ url="https://distrowatch.com/dwres-mobile.php?resource=ratings&distro=sparky"
ychaouche#ychaouche-PC 16:06:00 ~/WTMP $ echo ${url}
https://distrowatch.com/dwres-mobile.php?resource=ratings&distro=sparky
ychaouche#ychaouche-PC 16:06:06 ~/WTMP $ echo ${url/distrowatch.com/google.com}
https://google.com/dwres-mobile.php?resource=ratings&distro=sparky
ychaouche#ychaouche-PC 16:06:15 ~/WTMP $

**** exploding :: splitting :: split
***** split on spaces
S="je suis là mon pote"
read -ra prenoms <<<$S
for prenom in "${prenoms[@]}"
do
    echo $prenom
done
***** split on arbitrary character
IFS=: read -ra parts <<<"$1"
filepath=${parts[0]}
lineno=${parts[1]}
emacsclient -n +$lineno $filepath
**** extracting
***** using indices
echo ${string:start:length}
if start is negative
echo ${string: -start:length}
or 
echo ${string:(-start):length}

because :- is used for the default value.
***** extract last word
remove everything before last whitespace
${string## *}
***** extract first word
remove everything after first space
${string%% *}

***** extract a regex
the matched regex will be stored in $BASH_REMATCH,
which is an array,
so:

if [[ $string =~ REGEX ]]
then
    echo "${BASH_REMATCH[0]}"
fi


example:
root@messagerie-prep[10.10.10.19] /var/vmail/algerian-radio.dz/saida.boudjenana # head  /tmp/saida.boudjenana.chomp | while read line; do if [[ $line =~ S=.* ]]
> then
> echo "Matched: ${BASH_REMATCH[0]}"
> fi
> done
Matched: S=6964,W=7151
Matched: S=5764,W=5909
Matched: S=48221,W=48921
Matched: S=5888,W=6035
Matched: S=30154,W=30619
Matched: S=5517,W=5661
Matched: S=7141,W=7324
Matched: S=94041,W=95341
Matched: S=5982,W=6126
Matched: S=47470,W=48398
root@messagerie-prep[10.10.10.19] /var/vmail/algerian-radio.dz/saida.boudjenana #



**** counting
${#string}
**** generating
***** file extensions
file.{mpg,avi,mp4} generates file.mpg file.avi file.mp4

ychaouche#ychaouche-PC 09:33:33 ~ $ ls /home/ychaouche/DOCUMENTS/INTERNE/COMMISSIONS/MARCHES/FIREWALL/MARCHE/notes_rapporteur.{pdf,odt}
-rw-r--r-- 1 ychaouche ychaouche 42K May 17 17:07 /home/ychaouche/DOCUMENTS/INTERNE/COMMISSIONS/MARCHES/FIREWALL/MARCHE/notes_rapporteur.odt
-rw-r--r-- 1 ychaouche ychaouche 74K May 17 13:22 /home/ychaouche/DOCUMENTS/INTERNE/COMMISSIONS/MARCHES/FIREWALL/MARCHE/notes_rapporteur.pdf
ychaouche#ychaouche-PC 09:36:08 ~ $


18:15:16 ~/TMP -1- $ echo {one,two,three}.mp3
one.mp3 two.mp3 three.mp3
18:15:32 ~/TMP -1- $ echo file.{jpg,png,gif}
file.jpg file.png file.gif
18:15:36 ~/TMP -1- $



***** combinatorics
{a,b,c}{1,2} generates a1 a2 b1 b2 c1 c2
****** number sequence
{1..3} generates 1 2 3
{01..05..2} generates 01 03 05
***** character sequence
{a..c} generates a b c


*** parse arguments from the command line
**** manual parsing
***** using while loop
ce code est pris de ~/SYNCHRO/bin/net.translate
while (( $# ))
do
    case "$1" in
	-f|--from)
	    from="$2"
	    shift 2
	    ;;
	-t|--to)
	    to="$2"
	    shift 2	    
	    ;;
	*)
	    translate="$1"
	    shift
	    ;;
    esac
done
***** using for loop
don't use this.

**** getopts vs getopt
getopts :
 - doesn't parse long options (--my-var)
 - works well 
 - recommended by #bash

getopt : 
 - parses long options
 - doesn't work well w/ arguments or options involving spaces[u]
 - not recommended by #bash


My choice : getopt 

**** getopt
***** short options
getopt -o EF
***** long options
getopt -l txt,log,all
***** options with arguments
required argument :
optional argument ::

***** arguments containing spaces will be split
if you parse something like this : 
--LOG --headings "nextcloud comments"
last argument "nextcloud comments" will be split in two

use -u to avoid this and something like this to concatenate all remaining args into one big unquoted arg

[...]

TEMP=$(getopt -u [...] -- $@)
case $1 in 
	    --)
		shift
		last_arg="$*"
		;;
esac
***** invocation
TEMP=$(getopt -o EF -l all,log,txt,newer-than:,headings,recent,all -- $@)
*** read
**** read inside another read
use -u3 and close with 3< file, like so :

while IFS=, read -u3 -r nom prenom
do
  ...
  read -r input
  ...
done 3< file
**** read / read -rd ''
-r (raw) empêche l'évaluation des caractères échapés
-d'' lis jusqu'au NULL, au lieu de \n. A utiliser avec un find -print0

#bash :
https://wiki.bash-hackers.org/commands/builtin/read#read_without_-r
Essentially all you need to know about -r is to ALWAYS use it. 
The exact behavior you get without -r is completely useless even for weird purposes.
**** using readline facilities
read -e
**** display prompt
read -p <prompt>
**** read n characters
read -n

**** read silently (don't echo the characters)
read -s
useful for reading secrets
**** reading a file into an array
see ** write code *** collections **** normal arrays ***** reading from file into array
**** IFS=,/
while IFS=,/ read -r code color d m y; date="$y$m$d";
[11:21] <Raithmir> 123456,BLACK,14/08/24
[11:21] <Raithmir> 123456,BROWN,19/08/24
[11:21] <Raithmir> 123456,GREEN,21/08/24

et donc là on se retrouve code et color séparés par ,
puis les dates séparés par /

*** Globs
see * globbing
*** Menus
**** bmenu
https://github.com/HadrienAka/TheBashMenu (bmenu)
~/DOWNLOADS/TOOLS/
**** select
select opt in "option1" "option2" "option3"
do
  case $opt int
     "option1")
         do something
	 break
	 ;;
     "option2")
         do something else
	 break
	 ;;
     "option3")
	 break
	 ;;
   esac
done


*** default value
use  :- like this

{$var:-default value}
root@messagerie[10.10.10.19] ~ # type lastusers
lastusers is a function
lastusers ()
{
    tail -${1:-10} /root/DATA/newusers.lst
}
root@messagerie[10.10.10.19] ~ #
*** verify_args
# in  : 2 "usage : $0 <arg1> <arg2>" $@
# out : return 1 if less than 2 arguments
function verify_args {
    l_n=$1
    l_msg=$2
    shift 2
    if [ "$#" -lt $l_n ]
    then
	echo -e "$l_msg"
	return 1
    fi
    return 0
}



*** loops
**** loop as long as condition is true
while (($# > 0))
do
 <...>
 shift 2 || break
done
**** loop forever
while : # empty expression, loops forever
do
 <...>
done
*** booleans
there are no booleans in bash.
0 is considered true
all other values are false
you can use return inside a function and test the return value with $?
like this :

function foo {
 return 1
}
foo
echo $? # 1


*** functions
**** function or ()?
you can grep functions more easily with function 
but it's not POSIX
**** return values
the return value of a function is that of its last command
**** nested functions
define functions with () instead of {}

f() (
  g() {
    echo G
  }
  g
)

# Ouputs `G`
f
# Command not found.
g

g is only availble inside f, because () is executed in a subshell and everything inside it is destroyed after the function call.
**** undefining a function
unset -f <funcname>
**** dynamically calling a function
function foo {
  ... do things ...
}

function_name="foo"
${function_name} 

*** writing to stderr
echo something >> /dev/stderr
or
>&2 echo something
*** printf
**** write to a variable
write result to a variable with -v
**** output multiple lines, one for each arg
ychaouche#ychaouche-PC 15:48:25 /usr/share/man $ printf "ping %s\n" 10 50 980 978 21
ping 10
ping 50
ping 980
ping 978
ping 21
ychaouche#ychaouche-PC 15:48:32 /usr/share/man $
*** reading from a process
**** Why using pipes is a bad idea
variables are lost
**** Process substitution
while read something; do something_with $something; done < <(other process)
**** named pipes
mkfifo grep_command
grep something somewhere > grep_command
while read line; do something_with $line; done
*** using aliases inside shell scripts
You need shopt -s extend_aliases, which defaults to off (only on for interactive shells)
*** command line argument completion
**** getting user input
cur=${COMP_WORDS[COMP_CWORD]}
**** setting possible completions
if multiple completions for $cur, use an array construct (use parens) like below 
COMPREPLY=(`global -c $cur`)
complete -F <myfunction> <mycommand>

where <myfunction> is the function to call when <mycommand> needs a completion to any of its arguments.
**** example
source : https://www.gnu.org/software/global/globaldoc_toc.html
You can use the -c command with the complete command in the shell.
In Bash:

    $ funcs()
    > {
    >         local cur
    >         cur=${COMP_WORDS[COMP_CWORD]}
    >         COMPREPLY=(`global -c $cur`)
    > }
    $ complete -F funcs global
    $ global kmem_TABTAB
    kmem_alloc           kmem_alloc_wait      kmem_init
    kmem_alloc_nofault   kmem_free            kmem_malloc
    kmem_alloc_pageable  kmem_free_wakeup     kmem_suballoc
    $ global kmem_sTAB
    $ global kmem_suballoc
    ../vm/vm_kern.c
** read code
*** src
https://wiki.bash-hackers.org/scripting/obsolete
*** "${dir:?}"
This checks if the variable dir is set and not empty.
If dir is not set or is empty,
the script will terminate and display an error message indicating that dir is not set.
** debug
*** no such file or directory on a defined alias
try shopt -s expand_alias at the beginning of your script
* variables
** last process id
$!. Warning, only works if process is explicitly launched in the backround with & or similar (<( ) or >( ))

** PROMPT_COMMAND
will execute a command before each prompt
for example : 

export HISTTIMEFORMAT="%Y/%m/%d %H:%M:%S "
export HISTFILESIZE=-1 #don't truncate
export PROMPT_COMMAND="history -an" # [a]ppend commands to histfile immedatly, 
                                    # and read [n]ew entries (could be written from other termin
** $-
contains the options passed to the bash process itself[u]
$ echo $-
himBH
$ echo $-

[i]nteractive
[h]ash
[m]onitor : job control
[B]race expand 
[H]istory expansion

some other options include : 
no [C]lobber : don't overwrite files with >
no [u]nset : error if you try to access a non exisiting variable


** $SHELLOPTS :: shell options
enabled shell options.
same options as those accepted by set -o
there are also shorthands for common options,
for example: 
set -e: exit on error
set -u: exit on use of unset variable

the verbose forms are for those options are:
set -o errexit
set -o nounset
** $BASHOPTS
enabled shell options.
same options as those accepted by shopt -s

* bash / sh
lots of scripts still use sh by policy, especially system scripts, so that it would be easier to switch to another shell in the futre.
* echo
-e : interprète les séquences  \a[udio] \b[ackspace] etc.
voir la liste avec help echo
* quoting
** gl
quoting a variable preserves its whitespaces and special characters like "*"
if you don't quote the variable, whitespaces are removed and special characters are evaled (*)
quoting the assignment is optional
quoting the variable substitution is always mandatory.

if you need to escape a single quote inside a single quoted string,
you need to do it outside the single quoted string,
because inside it the backslash character has no special function,
it is litteral

so instead of:
echo 'I\'m the one'
do
'I'\''m the one'
where you split the string in two and put a single quote in the middle
'I' + \' + 'm the one'




** tcpdump example
_tcpdump_default_filter="host $(net.ip.private) and (tcp[13]==2 or icmp or udp)"

this works
but on command line I need to quote the (), because parens are part of bash syntax
so on the command line we have

_asroot tcpdump -i eth4 -q -l -n host 192.168.211.112 and '(tcp[13]==2 or icmp)'

the quotes are necessary, but on the script we have

_tcpdump_default_filter="host $(net.ip.private) and (tcp[13]==2 or icmp or udp)"

the single quotes were not necessary, because the parens were not part of the syntax, they were litteral

Introducing single quotes inside the double quotes will result in passing the actual single quotes to tcpdump

so tcpdump would litterally recieve tcpdump '(tcp[13]==2 or icmp)'

when it only needed tcpdump (tcp[13]==2 or icmp)
** sha512 example
password_as_hash="$(sha512 "$password_as_text")"
ça marche pour la raison suivante :
les quotes à l'interieur du $() sont lues par un autre shell.
il n'y a donc pas de quotes à l'interieur de quotes.
** grep.historically example
au lieu de mettre toute la commande dans une string,
on met les différentes parties dans un tableau,
ce qui permet de mettre des guillemets autour de valeurs contenant des espaces
et éviter ainsi le word splitting.

grep_command=(zgrep -h --line-buffered --color  "$grep_opts" "$pattern" "$filename")

ensuite pour executer la commande,
on utilise
"${grep_command[@]}"
avec les guiilemets,
car sans les guillemets le word splitting va encore s'opérer à ce niveau.
* my .bashrc_custom
** extract_audio
extracts audio from video
extract_audio <file>
** batchmp3
transform audio files to mp3
batchmp3 <files>
** getduration
get duration of a media
getduration <file>
* explode first argument into multiple arguments
IFS=<sep>
set -- $1 # must not quote!
now $1,$2... are parts of $1 that were seprated by <sep>

example : 

reverseip () {
    local IFS
    IFS=.
    set -- $1
    echo $4.$3.$2.$1
}

* ";"
is a bash control operator to separate commands, just like a newline. 
* ":"
** 1. separator 
PATH=<path>:<path>:<path>
chown user:group 
** 2. modifier
value = ${1:-default}
** 3. noop
*** used in while loops
while : 
*** used in comment blocks
: << 'COMMENT-BLOCK'
your code block here
COMMENT-BLOCK

* .
. file <args>
executes commands from file in current shell
if args are supplied, they'll be bound to positionnal parameters in the file
the exit status is that of the last command.
* !
négation d'un test. 
Example :
! [-r "$file" ] && echo "unreadable file" && exit 1

source : https://www.linuxquestions.org/questions/blog/michael-uplawski-1023960/transform-html-to-lq-markup-or-down-37848/
* < file
write file to stdin
* < fifo
wait until a writer connects to it.
for example : 

< "$FILE" || continue

found in /usr/bin/xzmore

* <(command)
this is called a process substitution.
it will be replaced by /dev/fd/<fd>
which is an ad-hoc fifo

command will write to /dev/fd/<fd>
you can read the output of that command by reading from /dev/fd/<fd> like this:
read < <(command)

consider this : 
find | while read filename; do ((count++)); done; 
echo there are $count files # count is empty.

this is because the while loop executes in a pipe,
and pipes are executed in subshells.
count is only defined in the subshell where the while loop is. 
The count outside the while loop is not even defined.

to avoid this, we use process substitution like this : 

while read filename; do ((count ++)); done < <(find)
echo there are $count files # count is not empty

<(find) will be replaced by /dev/fd/<fd> 
and < is for input redirection.
* >(command)
redirect output to a process
for example
tee >(script1) >(script2)
<paste>
^D

This will redirect the pasted input into script1 and script2 at the same time.
* <<EOF .. EOF
<<EOF is a heredoc
it means read from this until the first line containing only the word EOF.
EOF

* <<<$(command)
perform the substitution in a temporary file.
then feed that as a herestring to the command reading from it.

Useful with commands that take filenames as arguments but you want to give them strings
It's like transforming a string into a temporary document and feeding it to the reading command.
so instead of 
command | wc

we can do
wc <<<$(command)
* <<<"$var" or <<<"string of text"
example : 
instead of
echo "var" | wc

we can do
wc <<<"var"
* when to use <() and when to use <<<$()
use <() on commands expecting to read from files
use <<<$() on commands expecting to read from stdin
does this even make sense?
isn't stdin just like any other file?
* _
used as a placeholder, for example in sh -c 
sh -c "command $1 $2 $3" _ foo bar baz
inside the shell script we have : $0=_, $1=foo, $2=bar, $3=baz.


* ~
~ is POSIX
~ is expanded to $HOME
~login is expanded to /home/login if login exists
~- is expanded to $OLDPWD
* valid characters in function and variable names
** vars
alphanum + _. Can't start with a number.
** functions
anything except keywords I guess.
* writing/reading from a socket over the network
redirect to either
/dev/tcp/host/port
/dev/udp/host/port
* PS1 prompt
\t time in 24h format.
\e ASCII escape character (033)
\[ terminal control sequence \]

\033 -> espace character to introduce a terminal control sequence
[1m  -> bold
[0m  -> remove all effects.

Examples : 

PS1='\[\033[01;31m\]\u#\h\[\033[01;36m\]  \t \w \$\[\033[00m\] '

voir man terminfo
* vocabulary
** parameter expansion
$ 
$() command subtitution
${} variable substitution
** metacharacters
These are characters that separate words.
This in turn means you don't need to add spaces around them.
These are : | & ; ( ) < > <space> <tab>
Note that $ { }  [ ] ! <newline> are not metacharacters. 
* code snippets
** Detecting DE
from /usr/bin/xdg-email
#--------------------------------------
# Checks for known desktop environments
# set variable DE to the desktop environments name, lowercase

detectDE()
{
    if [ x"$MAILER" != x"" ]; then DE=envvar;
    elif [ x"$KDE_FULL_SESSION" = x"true" ]; then DE=kde;
    elif [ x"$GNOME_DESKTOP_SESSION_ID" != x"" ]; then DE=gnome;
    elif `dbus-send --print-reply --dest=org.freedesktop.DBus /org/freedesktop/DBus org.freedesktop.DBus.GetNameOwner string:org.gnome.SessionManager > /dev/null 2>&1` ; then DE=gnome;
    elif xprop -root _DT_SAVE_MODE 2> /dev/null | grep ' = \"xfce4\"$' >/dev/null 2>&1; then DE=xfce;
    elif [ x"$DESKTOP_SESSION" = x"LXDE" ]; then DE=lxde;
    elif [ x"$XDG_CURRENT_DESKTOP" = x"LXDE" ]; then DE=lxde;
    elif [ -r ~/.muttrc ] && which mutt > /dev/null 2>&1; then
        DE=envvar
        MAILER="x-terminal-emulator -e mutt"
    else DE=""
    fi
    DEBUG 2 "DE is \"$DE\""
}


** generated code
a pattern is to use something like this :

if test yes = yes

this is taken from pcre-config

      if test yes = yes ; then
        if test ${prefix}/include != /usr/include ; then
          includes=-I${prefix}/include
        fi
        echo $includes
      else
        echo "${usage}" 1>&2
      fi
      ;;
    --libs-posix)
      if test yes = yes ; then
        echo $libS$libR -lpcreposix -lpcre
      else
        echo "${usage}" 1>&2
      fi
      ;;
    --libs)
      if test yes = yes ; then
        echo $libS$libR -lpcre
      else
        echo "${usage}" 1>&2
      fi
      ;;
    --libs16)
      if test no = yes ; then
        echo $libS$libR -lpcre16
      else
        echo "${usage}" 1>&2
      fi
      ;;
    --libs-cpp)
      if test yes = yes ; then
        echo $libS$libR -lpcrecpp -lpcre
      else
        echo "${usage}" 1>&2
      fi
      ;;

* job control
** job specs
%%/%+ : last job
%1 : job N°1

examples : 
kill %%
kills last process put in the background.

root@messagerie-principale[10.10.10.19] ~ # tail -f /var/log/apache2/roundcube.access /var/log/apache2/mail.radioalgerie.dz.access | grep -E '\b(301|404)\b' --line-buffered  &
[1] 50461
root@messagerie-principale[10.10.10.19] ~ #

50461 is the pid of the last job



* traps
** trap foo INT
trap foo INT
will execute foo each time a SIGINT is issued.
that's it!
** trap foo DEBUG
if you
trap foo DEBUG
then foo will be called before each simple command,
for,
case,
select,
arithmetic for command,
or function call

Example:
if you put this in /home/nagios/.bashrc
trap 'logger -p auth.info -t nagios "Running $BASH_COMMAND"' DEBUG

then each time the nagios user runs a command it will be logged in auth.info
** trap foo RETURN
if you trap foo RETURN
then foo will be called after each command,
or after a file has been sourced



* intput/output redirections
** explained example
{
[1]	tail -f /var/log/apache2/roundcube.error
	     	/var/log/fail2ban.log
		/var/log/apache2/mail.radioalgerie.dz.error
		/var/log/dovecot.log
		/var/log/mail.warn
		/var/log/auth.log
		/var/log/daemon.log
		/var/log/syslog
		/var/log/mysql/error.log
		/var/log/apache2/error.log
[2]		| grep -Ei '(fail|reject|error|banned|quarantine|virus|malware|detected|critic|fatal)' --line-buffered  >&3
[3]		| tail -f /var/log/apache2/roundcube.access /var/log/apache2/mail.radioalgerie.dz.access
[4]		| grep -E '\b(301|404)\b' --line-buffered ;
} 3>&1 | tee -a /tmp/errors

[3] is executed without reading from [2]
and the output of [2] is redirected to fd 3.
then the output of both [2] and [4] are redirected to stdout via that 3>&1 redirection,
because the output of [2] was in fd 3.
So fd 3 is a kind of buffer to save the output of [2],
and when we need that output we just redirect it to stdout with 3>&1.

the [1]|[2] pipline is executed in parallel with the [3]|[4] pipeline through another pipline.
So we have two pipelines running in parallel,
A | B,
but where B isn't reading from the output A.

