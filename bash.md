
# debugging

set -x	# activate debugging from here

set +x	# stop debugging from here

set -o  # display shell options

set -u # error when calling unset variable

set -f	(set -o noglob)	Disable file name generation using metacharacters (globbing).

set -v	(set -o verbose) Prints shell input lines as they are read.

set -x	(set -o xtrace)	Print command traces before executing command.

#!/bin/bash -xv

# commands


export = push variable into glabal vars

# Special symbols

$? last exit code

$_ param from pipeline

S1, $2 access param by index

"$$" shell proc id

$! proc id of of recently executed command

$@ Expands to the positional parameters, starting from one. When the expansion occurs within double quotes, each parameter expands to a separate word.  Same as S*

$* Expands to the positional parameters, starting from one. When the expansion occurs within double quotes, it expands to a single word with the value of each parameter separated by the first character of the IFS special variable.  Do not use!

$- A hyphen expands to the current option flags as specified upon invocation, by the set built-in command, or those set by the shell itself (such as the -i).

$0 name of the script

$# number of arguments

~ returns home path

~+ returns value of pwd

~+#  returns directory stack value where # is replaced with numeric value. + is assumed if no + or - is provided


# IF Loops

```bash
#!/bin/bash

echo "This scripts checks the existence of the messages file."
echo "Checking..."
if [ -f /var/log/messages ]
  then
    echo "/var/log/messages exists."
fi
echo
echo "...done."

anny ~> ./msgcheck.sh
This scripts checks the existence of the messages file.
Checking...
/var/log/messages exists.

...done.
```

```bash
anny > num=`wc -l work.txt`

anny > echo $num
201

anny > if [ "$num" -gt "150" ]
More input> then echo ; echo "you've worked hard enough for today."
More input> echo ; fi
```

```bash
#!/bin/bash

# Calculate the week number using the date command:

WEEKOFFSET=$[ $(date +"%V") % 2 ]

# Test if we have a remainder.  If not, this is an even week so send a message.
# Else, do nothing.

if [ $WEEKOFFSET -eq "0" ]; then
  echo "Sunday evening, put out the garbage cans." | mail -s "Garbage cans out" your@your_domain.org
fi
```

```bash
if [ "$(whoami)" != 'root' ]; then
        echo "You have no permission to run $0 as non-root user."
        exit 1;
fi
```

shortt version of above

```bash
[ "$(whoami)" != 'root' ] && ( echo you are using a non-privileged account; exit 1 )
```

Use && to eval if true, or || if false

```bash
anny > gender="female"

anny > if [[ "$gender" == f* ]]
More input> then echo "Pleasure to meet you, Madame."; fi
Pleasure to meet you, Madame.
```

You can also use the test command:

test "$(whoami)" != 'root' && (echo you are using a non-privileged account; exit 1)

### No exit?
If you invoke the exit in a subshell, it will not pass variables to the parent. Use { and } instead of ( and ) if you do not want Bash to fork a subshell.


# If, THEN, ELSE Loops

```bash
freddy scripts> gender="male"

freddy scripts> if [[ "$gender" == "f*" ]]
More input> then echo "Pleasure to meet you, Madame."
More input> else echo "How come the lady hasn't got a drink yet?"
More input> fi
How come the lady hasn't got a drink yet?

freddy scripts>
```

# [] vs. [[]]

Contrary to [, [[ prevents word splitting of variable values. So, if VAR="var with spaces", you do not need to double quote $VAR in a test - eventhough using quotes remains a good habit. Also, [[ prevents pathname expansion, so literal strings with wildcards do not try to expand to filenames. Using [[, == and != interpret strings to the right as shell glob patterns to be matched against the value to the left, for instance: [[ "value" == val* ]].


```bash
anny ~> cat weight.sh
#!/bin/bash

# This script prints a message about your weight if you give it your
# weight in kilos and height in centimeters.

weight="$1"
height="$2"
idealweight=$[$height - 110]

if [ $weight -le $idealweight ] ; then
  echo "You should eat a bit more fat."
else
  echo "You should eat a bit more fruit."
fi

anny ~> bash -x weight.sh 55 169
+ weight=55
+ height=169
+ idealweight=59
+ '[' 55 -le 59 ']'
+ echo 'You should eat a bit more fat.'
You should eat a bit more fat.
```

```bash


anny ~> cat weight.sh
#!/bin/bash

# This script prints a message about your weight if you give it your
# weight in kilos and height in centimeters.

if [ ! $# == 2 ]; then
  echo "Usage: $0 weight_in_kilos length_in_centimeters"
  exit
fi

weight="$1"
height="$2"
idealweight=$[$height - 110]

if [ $weight -le $idealweight ] ; then
  echo "You should eat a bit more fat."
else
  echo "You should eat a bit more fruit."
fi

anny ~> weight.sh 70 150
You should eat a bit more fruit.

anny ~> weight.sh 70 150 33
Usage: ./weight.sh weight_in_kilos length_in_centimeters
```

```bash
#!/bin/bash

# This script gives information about a file.

FILENAME="$1"

echo "Properties for $FILENAME:"

if [ -f $FILENAME ]; then
  echo "Size is $(ls -lh $FILENAME | awk '{ print $5 }')"
  echo "Type is $(file $FILENAME | cut -d":" -f2 -)"
  echo "Inode number is $(ls -i $FILENAME | cut -d" " -f1 -)"
  echo "$(df -h $FILENAME | grep -v Mounted | awk '{ print "On",$1", \
which is mounted as the",$6,"partition."}')"
else
  echo "File does not exist."
fi
```

IF, THEN, ELIF, ELSE Loop

nested IF

```bash
anny ~/testdir> cat testleap.sh
#!/bin/bash
# This script will test if we're in a leap year or not.

year=`date +%Y`

if [ $[$year % 400] -eq "0" ]; then
  echo "This is a leap year.  February has 29 days."
elif [ $[$year % 4] -eq 0 ]; then
        if [ $[$year % 100] -ne 0 ]; then
          echo "This is a leap year, February has 29 days."
        else
          echo "This is not a leap year.  February has 28 days."
        fi
else
  echo "This is not a leap year.  February has 28 days."
fi

anny ~/testdir> date
Tue Jan 14 20:37:55 CET 2003

anny ~/testdir> testleap.sh
This is not a leap year.
```

```bash
anny ~/testdir> cat penguin.sh
#!/bin/bash
                                                                                                 
# This script lets you present different menus to Tux.  He will only be happy
# when given a fish.  We've also added a dolphin and (presumably) a camel.
                                                                                                 
if [ "$menu" == "fish" ]; then
  if [ "$animal" == "penguin" ]; then
    echo "Hmmmmmm fish... Tux happy!"
  elif [ "$animal" == "dolphin" ]; then
    echo "Pweetpeettreetppeterdepweet!"
  else
    echo "*prrrrrrrt*"
  fi
else
  if [ "$animal" == "penguin" ]; then
    echo "Tux don't like that.  Tux wants fish!"
    exit 1
  elif [ "$animal" == "dolphin" ]; then
    echo "Pweepwishpeeterdepweet!"
    exit 2
  else
    echo "Will you read this sign?!"
    exit 3
  fi
fi
```