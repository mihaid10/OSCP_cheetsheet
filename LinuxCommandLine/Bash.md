# Bash

[toc]

### スクリプトサンプル

```bash
#!/bin/bash
# Hello World Bash Script
echo "Hello World!"
```

### 変数

```bash
first_name=Good
last_name=Hacker
echo $first_name $last_name

# シングルクオートは文字列のまま、ダブルクオートは文字列展開
kali@kali:~$ greeting='Hello World'
kali@kali:~$ echo $greeting
Hello World

kali@kali:~$ greeting2="New $greeting"
kali@kali:~$ echo $greeting2
New Hello World

# 変数をコマンドの結果で設定
kali@kali:~$ user=$(whoami)
kali@kali:~$ echo $user
kali

kali@kali:~$ user2=`whoami`
kali@kali:~$ echo $user2
kali
```



### 特殊変数

| RIABLE NAME | DESCRIPTION                                      |
| :---------: | :----------------------------------------------- |
|     $0      | The name of the Bash script                      |
|   $1 - $9   | The first 9 arguments to the Bash script         |
|     $#      | Number of arguments passed to the Bash script    |
|     $@      | All arguments passed to the Bash script          |
|     $?      | The exit status of the most recently run process |
|     $$      | The process ID of the current script             |
|    $USER    | The username of the user running the script      |
|  $HOSTNAME  | The hostname of the machine                      |
|   $RANDOM   | A random number                                  |



### if文

```
if [ <some test> ]
then
  <perform action>
elif [ <some test> ]
then
  <perform different action>
else
  <perform yet another different action>
fi
```

※[ の後ろと]の前には必ず空白が必要である

 

### 比較演算子

|       OPERATOR        | DESCRIPTION: EXPRESSION TRUE IF...             |
| :-------------------: | :--------------------------------------------- |
|      !EXPRESSION      | The EXPRESSION is false.                       |
|       -n STRING       | STRING length is greater than zero             |
|       -z STRING       | The length of STRING is zero (empty)           |
|  STRING1 != STRING2   | STRING1 is not equal to STRING2                |
|   STRING1 = STRING2   | STRING1 is equal to STRING2                    |
| INTEGER1 -eq INTEGER2 | INTEGER1 is equal to INTEGER2                  |
| INTEGER1 -ne INTEGER2 | INTEGER1 is not equal to INTEGER2              |
| INTEGER1 -gt INTEGER2 | INTEGER1 is greater than INTEGER2              |
| INTEGER1 -lt INTEGER2 | INTEGER1 is less than INTEGER2                 |
| INTEGER1 -ge INTEGER2 | INTEGER1 is greater than or equal to INTEGER 2 |
| INTEGER1 -le INTEGER2 | INTEGER1 is less than or equal to INTEGER 2    |
|        -d FILE        | FILE exists and is a directory                 |
|        -e FILE        | FILE exists                                    |
|        -r FILE        | FILE exists and has read permission            |
|        -s FILE        | FILE exists and it is not empty                |
|        -w FILE        | FILE exists and has write permission           |
|        -x FILE        | FILE exists and has execute permission         |



### ブール型論理演算子

AND (&&) 論理演算子は、**直前のコマンドが成功した場合 (または True または 0 を返した場合) にのみ、コマンドを実行**

```bash
kali@kali:~$ user2=kali
kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!"
kali:x:1000:1000:,,,:/home/kali:/bin/bash
kali found!

kali@kali:~$ user2=bob
kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!"
```

コマンドリストで使用する場合、**OR (||) 演算子は AND (&&) の逆で、前のコマンドが失敗した (False または非ゼロを返した) 場合のみ、次のコマンドを実行**

```bash
kali@kali:~$ echo $user2
bob

kali@kali:~$ grep $user2 /etc/passwd && echo "$user2 found!" || echo "$user2 not found!"
bob not found!
```



if文で使用する場合

```bash
#/bin/bash
# and example

if [ $USER == 'kali' ] && [ $HOSTNAME == 'kali' ]
then
  echo "Multiple statements are true!"
else
  echo "Not much to see here..."
fi
```



### For文

* スクリプト

  ```bash
  for var-name in <list>
  do
    <action to perform>
  done
  ```

  

* Bashワンライナー

  ```bash
  kali@kali:~$ for ip in $(seq 1 10); do echo 10.11.1.$ip; done
  kali@kali:~$ for i in {1..10}; do echo 10.11.1.$i;done
  ```

  

### while文

```bash
#!/bin/bash
# while loop example

counter=1

while [ $counter -lt 10 ]
do
  echo "10.11.1.$counter"
  ((counter++))
done
```



### カウンター処理

```bash
counter=1

while [ $counter -lt 10 ]
do
  echo "10.11.1.$counter"
  ((counter++))
done
```



### ユーザ入力の読み込み

```bash
#!/bin/bash
echo "Hello there, would you like to learn how to hack: Y/N?"
read answer
echo "Your answer was $answer"
```

```bash
kali@kali:~$ cat ./input2.sh
#!/bin/bash
# Prompt the user for credentials

read -p 'Username: ' username
read -sp 'Password: ' password

echo "Thanks, your creds are as follows: " $username " and " $password

kali@kali:~$ chmod +x ./input2.sh

kali@kali:~$ ./input2.sh
Username: kali
Password: 
Thanks, your creds are as follows:  kali  and  nothing2see!
```

* -pオプション：プロンプトを指定する
* -sオプション：ユーザー入力を無音にする
