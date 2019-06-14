# Awk Command:-
Awk is a scripting language used for manipulating data and generating reports.
The awk command programming language requires no compiling, and allows the user to use variables, numeric functions, string functions, and logical operators.

WE DO WITH AWK
1.Scans a file line by line

Splits each input line into fields
Compares input line/fields to pattern
Performs action(s) on matched lines
Sed Command:-

sed is a Stream Editor used for modifying the files in linux.
Whenever you want to make changes to the file automatically, sed comes in handy to do this.

Shebang:-

It is called a shebang or a “bang” line. It is nothing but the absolute path to the Bash interpreter.
It consists of a number sign and an exclamation point character (#!), followed by the full path to the interpreter such as /bin/bash.
All scripts under Linux execute using the interpreter specified on a first line

Make a script and print your name 10 times(use fuction to print your name).

#!/bin/bash
echo enter your name
read name
amar()
{
echo your name is $name
}
i=1
while [ $i -le 10 ]
do
amar
let i++
done

print 5 times “INDIA” if value is greater than or equal to 10

#!/bin/bash
if [ $1 -lt 10 ]
then
i=1
while [ $i -le 10 ]
do
expr $1 * $i
let i++
done
else
j=1
while [ $j -le 5 ]
do
echo INDIA
let j++
done
fi

Make a script and pass 5 command line arguments

#!/bin/bash
echo first argument is $1
echo fifth argument is $5
if [ $2 -eq 10 ]
then
echo “INDIA”
else
echo “india”
fi
sum=expr $1 + $2 + $3 + $4 + $5
echo Sum of all argument is $sum
