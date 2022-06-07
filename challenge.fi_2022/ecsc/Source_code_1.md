# Source Code 1
```
The developer has made an authenticator app. Are you able to find the correct password?
```
Attachments: `authenticator.sh`  

We're given a bash script that from a quick glance looks like rules to build the flag yourself!

```bash
declare -a ARRAY

read -rp 'password: ' PASSWD # read the password
[[ $PASSWD == "" ]] && { printf "Requires a password!\n"; exit 1; } # make sure it's not empty

for ((i=0;i<${#PASSWD};i++));do ARRAY=("${ARRAY[@]}" "${PASSWD:i:1}"); done # split the password into letters

ARRAYlen=${#ARRAY[@]} # calculate password length

if [[ $ARRAYlen > 5 ]] && [[ $ARRAYlen < 7 ]];then
  :
else
  exit 1 # if the length isn't 6, it's invalid
fi

[[ ${ARRAY[0]} == [X] ]] && { :; } || exit 1 # the first letter of the password is `X`

case ${ARRAY[1]} in # the second letter
  [b-z]) exit 1 # isn't between b-z
  ;;
  [0-9]) exit 1 # isn't a number
  ;;
  aA) : # but instead is the letter `a` or `A`
  ;;
esac

declare -a ARRAY1 ARRAY2
declare -A ARRAY3
ARRAY1=(1 2 0 5 3 4 6)
ARRAY2=("b" "d" "a" "f" "c" "t" "A")
ARRAY3=(["b"]=0 ["d"]=2 ["c"]=1 ["t"]=3 ["A"]=5 ["f"]=6 ["a"]=4)
t1=${ARRAY3[${ARRAY[2]}]}
t2=${ARRAY1[${t1}]}
t3=${ARRAY2[${t2}]}

# ARRAY[2] -> t1 -> t2 -> t3
# b -> 0 -> 1 -> d
# d -> 2 -> 0 -> b
# a -> 4 -> 3 -> f
# f -> 6 -> 6 -> A
# c -> 1 -> 2 -> a
# t -> 3 -> 5 -> t
# A -> 5 -> 4 -> c

for ((i=0;i<${#ARRAY[@]};i++));do # loop through all characters in password
  if [[ $t3 == ${ARRAY[$i]} ]];then # it must contain one of the characters `bdafctA` depending on t3, see below
    break
  elif [[ $i == 5 ]];then # if we reach the 6th character, it's invalid
    exit 1
  else
    :
  fi
done

# Which means that we have requirement that the following character pairs exist depending on the 3rd character:
# [bd], [af], [fA], [ca], [t], [Ac]

function func_one() {
  if [[ $1 == ${ARRAY2[2]} ]]; then # the first argument must be letter `a`
    return 6
  else
    exit 1
  fi
}

func_one ${ARRAY[3]} # 4th character must be letter `a`

[[ ${ARRAY[4]} = "${ARRAY2[$?]}" ]] && { :; } || exit 1 # 5th character must be letter `A`

function func_two() {
  for i in ${ARRAY[@]}; do
    if [[ $i == ${ARRAY[-1]} ]];then # there's one or two letters `M`, but at least the last letter is `M`
      if [[ $i = "M" ]];then
        break
      else
        exit 1
      fi
    fi
  done
}

func_two

[[ ${ARRAY[1]} != ${ARRAY[1],,} ]] && { exit 1; } # 2nd character must be lowercase, and due to one of the initial rules we know it must be either `a` or `A`, so we know the 2nd character is `a`

echo "CORRECT!"
echo "GENZ{${PASSWD}}"
```

Based on this, we know the password has the following letters:
* 1st letter: X
* 2nd letter: a
* 3rd letter: f or t or c
* 4th latter: a
* 5th letter: A
* 6th letter: M

Thus we know the flag is `GENZ{XafaAM}`, `GENZ{XataAM}`, or `GENZ{XacaAM}`.