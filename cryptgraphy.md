# pico CTF 2017 Writeups

ちょろりちょろり

## CRYPTGRAPHY

### Hash101 (50pt)

> Prove your knowledge of hashes and claim a flag as your prize! > > > Connect to the service at shell2017.picoctf.com:19170
> UPDATED 16:12 EST 1 Apr.
>
>  HINTS  
> All concepts required to complete this challenge, including simple > modular math, are quickly found by googling :)

まずはncでサーバに接続
`nc shell2017.picoctf.com 19170`
するとこんなプロンプトが。

> Welcome to Hashes 101!
>
> There are 4 Levels. Complete all and receive a prize!

4問解かないといけないらしい。
- バイナリ部分は毎回変わる
- 時間制限があるので急がないと (ヒントにも書いてある通り)

> -------- LEVEL 1: Text = just 1's and 0's --------  
> All text can be represented by numbers. To see how different letters translate to numbers, go to http://www.asciitable.com/
>
> TO UNLOCK NEXT LEVEL, give me the ASCII representation of 0111000001101100011000010110100101100100

まずは表示された2進数をアスキーに変換しろということなのでbcで１６進表示に変換、それをxxd -r -pでアスキーに変換。

    $echo "obase=16; ibase=2; 0111000001101100011000010110100101100100" | bc | xxd -r -p
    plaid

    >plaid
    Correct! Completed level 1

> ------ LEVEL 2: Numbers can be base ANYTHING -----  
> Numbers can be represented many ways. A popular way to represent computer data is in base 16 or 'hex' since it lines up with bytes very well (2 hex characters = 8 binary bits). Other formats include base64, binary, and just regular base10 (decimal)! In a way, that ascii chart represents a system where all text can be seen as "base128" (not including the Extended ASCII codes)
>
> TO UNLOCK NEXT LEVEL, give me the text you just decoded, plaid, as its hex equivalent, and then the decimal equivalent of that hex number ("foo" -> 666f6f -> 6713199)

今度は１６進表示と10進表示を入力せよとのこと。  
さっきと同じ要領で。

    $ echo "obase=16; ibase=2; 01110000011011000110000101101001000" | bc
    706C616964
    $ echo "obase=10; ibase=2; 0111000001101100011000010110100101100100" | bc
    482854660452

答えを入力するとき、a〜fを大文字にするとエラーになるので腹立つ。  
10進数もコピペしたらエラーになったのはなんでだろう。

> hex>706c616964  
Good job! 706c616964 to ASCII -> plaid is plaid
Now decimal  
dec>482854660452  
Good job! 482854660452 to Hex -> 706c616964 to ASCII -> plaid is plaid  
Correct! Completed level 2

> ----------- LEVEL 3: Hashing Function ------------  
A Hashing Function intakes any data of any size and irreversibly transforms it to a fixed length number. For example, a simple Hashing Function could be to add up the sum of all the values of all the bytes in the data and get the remainder after dividing by 16 (modulus 16)
>
> TO UNLOCK NEXT LEVEL, give me a string that will result in a 0 after being transformed with the mentioned example hashing function

これは簡単。
キャラクタコードを16(=0x10)で割った余りが"0"になるキャラクタを答えれば良いので、キャラクタコード表から0x10、0x20、0x30 ……のどれかを拾ってくる。
接続するたびに"that will result in a 0"の0の部分が変わるので、例えば"in a 5"なら0x25などを答える。

    \>@  
    Correct! Completed level 3

> --------------- LEVEL 4: Real Hash ---------------  
> A real Hashing Function is used for many things. This can include checking to ensure a file has not been changed (its hash value would change if any part of it is changed). An important use of hashes is for storing passwords because a Hashing Function cannot be reversed to find the initial data. Therefore if someone steals the hashes, they must try many different inputs to see if they can "crack" it to find what password yields the same hash. Normally, this is too much work (if the password is long enough). But many times, people's passwords are easy to guess... Brute forcing this hash yourself is not a good idea, but there is a strong possibility that, if the password is weak, this hash has been cracked by someone before. Try looking for websites that have stored already cracked hashes.
>
TO CLAIM YOUR PRIZE, give me the string password that will result in this MD5 hash (MD5, like most hashes, are represented as hex digits):
1f5ff8608da1e56a59d1dff059286192

一番頭使わない。
表示されたMD5の値をそのままgoogle先生に聞けば、MD5のレインボーテーブルが見つかる。  
本当にいいのか？ 感はあるが、HINTでもgoogle力が問われる、って言ってたからいいよね。

    >pr0m7
    Correct! Completed level 4
    You completed all 4 levels! Here is your prize: 1b16e0e724dc0a8b96d127a6af8ed9a7


### computeAES (50pt)
> You found this clue laying around. Can you decrypt it?
>
>  HINTS
> Try online tools or python

clueの中身
> Encrypted with AES in ECB mode. All values base64 encoded  
ciphertext = rvn6zLZS4arY+yWNwZ5YlbLAv/gjwM7gZJnqyQjhRZVCC5jxaBvfkRapPBoyxu4e  
key = /7uAbKC7hfINLcSZE+Y9AA==

ヒントに素直に従って、pythonを使う。  
暗号化ライブラリpycryptをインストールしましょう。  
参考: https://kamatari.github.io/2016/04/23/what-is-pycrypto/

入れたら、使い方を調べて実行するだけ。  
あらかじめbase64をデコードしておこうと思ったらそうはいかなかったので、インラインで。

    >>> from Crypto.Cipher import AES
    >>> from base64 import b64decode
    >>> cipher = AES.AESCipher(b64decode("6v3TyEgjUcQRnWuIhjdTBA=="), AES.MODE_ECB)
    >>> print cipher.decrypt(b64decode("rW4q3swEuIOEy8RTIp/DCMdNPtdYopSRXKSLYnX9NQe8z+LMsZ6Mx/x8pwGwofdZ"))
    flag{do_not_let_machines_win_983e8a2d}_________


### BINARY EXPLOITATION

Just No (40pt)
A program at /problems/7e8b456c98db60be9a33339ab4509fca has access to a flag but refuses to share it. Can you convince it otherwise?

 HINTS
Check out the difference between relative and absolute paths and see if you can figure out how to use them to solve this challenge. Could you possibly spoof another auth file it looks at instead...?
