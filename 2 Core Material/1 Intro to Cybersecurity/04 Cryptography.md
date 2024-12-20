# 补充知识

## requests

### 基本用法

#### 1. 发起 GET 请求

GET 请求通常用于从服务器获取资源，`requests.get()` 是用来发送 GET 请求的方法。

```py
# 发起 GET 请求
response = requests.get('https://jsonplaceholder.typicode.com/posts')

# 查看响应的状态码
print(f"Status Code: {response.status_code}")

# 查看响应内容（文本）
print(f"Response Text: {response.text}")

# 如果响应是 JSON 格式的数据，可以用 .json() 方法
json_data = response.json()
print(f"JSON Data: {json_data}")
```

- `status_code`: 返回 HTTP 响应的状态码（例如 `200` 表示成功）。
- `text`: 返回响应的内容，通常是 HTML 或纯文本。
- `json()`: 如果响应内容是 JSON 格式，可以用 `.json()` 方法解析。

#### 2. 发起 POST 请求

POST 请求通常用于将数据发送到服务器（例如提交表单或上传数据）。使用 `requests.post()` 方法可以发送 POST 请求。

```py
import requests

# 发送 POST 请求
data = {'username': 'john', 'password': '123456'}
response = requests.post('https://httpbin.org/post', data=data)

# 查看响应内容
print(f"Response Text: {response.text}")
```

- `data`: 在 `requests.post()` 方法中，数据通常作为表单提交，可以传递字典类型的数据。
- 响应通常会返回 JSON 格式，使用 `.json()` 方法解析它。

#### 3. 传递 URL 参数

在发起 GET 请求时，可以通过 `params` 参数传递 URL 参数。

```py
import requests

# 使用 URL 参数
params = {'q': 'python', 'page': 2}
response = requests.get('https://www.google.com/search', params=params)

# 查看请求的 URL 和响应内容
print(f"Request URL: {response.url}")
print(f"Response Text: {response.text[:200]}")  # 只打印前200个字符
```

- `params`: URL 参数（键值对）会被自动编码成查询字符串，并附加到 URL 上。

#### 4. 发起 PUT 和 DELETE 请求

`requests` 还支持 PUT 和 DELETE 请求，分别用于更新资源和删除资源。

```py
import requests

# PUT 请求更新数据
response = requests.put('https://httpbin.org/put', data={'name': 'john'})
print(response.text)

# DELETE 请求删除数据
response = requests.delete('https://httpbin.org/delete')
print(response.text)
```

### 发送 Headers 和 Cookies

#### 1. 添加 Headers

HTTP 请求头部（Headers）可以包含请求的元数据（如用户代理、授权信息等）。通过 `headers` 参数可以添加自定义请求头。

```py
import requests

headers = {
    'User-Agent': 'my-app',
    'Authorization': 'Bearer token_here'
}

response = requests.get('https://jsonplaceholder.typicode.com/posts', headers=headers)

print(response.text)
```

#### 2. 发送 Cookies

你可以通过 `cookies` 参数传递 cookies，或者使用 `requests.get()` 或 `requests.post()` 返回的 cookies。

```py
import requests

# 发送请求时带上 Cookies
cookies = {'session_id': '123456'}
response = requests.get('https://httpbin.org/cookies', cookies=cookies)

print(response.text)
```

### 处理响应

#### 1. 检查响应状态

可以通过 `response.status_code` 检查 HTTP 响应的状态码。

```py
import requests

response = requests.get('https://jsonplaceholder.typicode.com/posts')

if response.status_code == 200:
    print("Success!")
else:
    print(f"Error: {response.status_code}")
```

#### 2. 响应内容类型

你可以检查响应的内容类型，以确定返回数据的格式。

```py
import requests

response = requests.get('https://jsonplaceholder.typicode.com/posts')

# 获取响应的 Content-Type
content_type = response.headers['Content-Type']
print(f"Content-Type: {content_type}")
```

#### 3. 响应时间

`requests` 提供了 `elapsed` 属性来获取请求的响应时间。

```py
import requests

response = requests.get('https://jsonplaceholder.typicode.com/posts')
print(f"Request took: {response.elapsed.total_seconds()} seconds")
```

### 错误处理

#### 1. 处理请求异常

在网络请求过程中，可能会出现一些异常，比如超时、网络不可达等。可以通过 `try-except` 来处理这些异常。

```py
import requests

try:
    response = requests.get('https://jsonplaceholder.typicode.com/posts', timeout=5)
    response.raise_for_status()  # 如果状态码不是 200，会抛出异常
except requests.exceptions.RequestException as e:
    print(f"An error occurred: {e}")
```

#### 2. 超时设置

你可以设置请求的超时时间，防止请求长时间阻塞。

```py
import requests

try:
    response = requests.get('https://jsonplaceholder.typicode.com/posts', timeout=3)
    print(response.text)
except requests.exceptions.Timeout:
    print("The request timed out")
```

### 上传文件

通过 `files` 参数，`requests` 可以轻松实现文件上传。

```py
import requests

files = {'file': open('myfile.txt', 'rb')}
response = requests.post('https://httpbin.org/post', files=files)
print(response.text)
```

### 会话（Session）

`requests.Session()` 允许你在多个请求之间保持某些参数（如 Cookies 和 Headers）的一致性。这样做可以提高性能并简化代码。

```py
import requests

# 使用会话保持 Cookie 和 Headers
with requests.Session() as s:
    s.headers.update({'User-Agent': 'my-app'})
    response = s.get('https://jsonplaceholder.typicode.com/posts')
    print(response.text)
```

### 总结

- **GET 请求**：用于获取资源，`requests.get()`。
- **POST 请求**：用于提交数据，`requests.post()`。
- **PUT 请求**：用于更新资源，`requests.put()`。
- **DELETE 请求**：用于删除资源，`requests.delete()`。
- **Headers 和 Cookies**：通过 `headers` 和 `cookies` 参数发送。
- **文件上传**：使用 `files` 参数上传文件。
- **超时和异常**：通过 `timeout` 参数和 `try-except` 块处理网络请求异常。
- **会话管理**：通过 `requests.Session()` 管理多个请求之间的状态。

`requests` 库使得 HTTP 请求非常简单和直观，是 Python 开发者常用的工具之一。



## substr

如何使用？

```SQL
select substr(x, 1, 6) from secrets
-- x是字符串，1是index（从1开始），6是长度
-- 这个的意思是：select x from secrets，然后取x的第1个到第6个字符（长度为6）
```



# 01-XOR

## 问题描述

就是考察异或

## 思路

用python的异或

## 截图

如图

![image-20241125144308258](./04%20Cryptography.assets/image-20241125144308258.png)



# 02-Hex

## 问题描述

源码：

```py
#!/opt/pwn.college/python

import random
import sys

for n in range(10):
    print(f"Challenge number {n}...")

    key = random.randrange(1, 256)
    plain_secret = random.randrange(0, 256)
    cipher_secret = plain_secret ^ key

    print(f"The key: {key:#04x}")
    print(f"Encrypted secret: {cipher_secret:#04x}")
    answer = int(input("Decrypted secret? "), 16)
    print(f"You entered: {answer:#04x}, decimal {answer}.")
    if answer != plain_secret:
        print("INCORRECT!")
        sys.exit(1)

    print("Correct! Moving on.")

print("CORRECT! Your flag:")
print(open("/flag").read())
```

## 思路

其实执行完程序后，题目给我两个hex数据，我不知道要让我干什么。

看了看源码，才知道原来还是疑惑，而且我们的输入必须是十六进制。

本来打算手动做，后来觉得太麻烦，就直接让gpt生成一个自动化的脚本。

这个脚本还是很值得学习学习的。

## EXP

```python
hacker@cryptography~hex:~$ ipython
Python 3.11.9 (main, Apr  2 2024, 08:25:04) [GCC 13.2.0]
Type 'copyright', 'credits' or 'license' for more information
IPython 8.24.0 -- An enhanced Interactive Python. Type '?' for help.

   ...:     # 获取 Key 和 Encrypted Secret
   ...:     child.expect(r'The key: (0x[0-9a-fA-F]+)')
   ...:     key = int(child.match.group(1), 16)
   ...: 
   ...:     child.expect(r'Encrypted secret: (0x[0-9a-fA-
   ...: F]+)')
   ...:     cipher_secret = int(child.match.group(1), 16)
   ...: 
   ...: 
   ...:     # 计算解密结果
   ...:     plain_secret = cipher_secret ^ key
   ...: 
   ...:     # 发送解密结果
   ...:     child.expect(r'Decrypted secret\? ')
   ...:     child.sendline(f'{plain_secret:#04x}')
   ...: 
   ...:     # 检查是否正确
   ...:     child.expect(r'Correct!|INCORRECT!')
   ...:     print(child.after)
   ...: 
   ...:     if "INCORRECT!" in child.after:
   ...:         print("Script failed to decrypt correctly
   ...: !")
   ...:         break
   ...: 
   ...: # 获取最终 flag
   ...: child.expect(r'CORRECT! Your flag:')
   ...: child.expect(pexpect.EOF)
   ...: print(child.before)
   ...: 
Challenge number 0...
Correct!
Challenge number 1...
Correct!
Challenge number 2...
Correct!
Challenge number 3...
Correct!
Challenge number 4...
Correct!
Challenge number 5...
Correct!
Challenge number 6...
Correct!
Challenge number 7...
Correct!
Challenge number 8...
Correct!
Challenge number 9...
Correct!

pwn.college{sROBxEuigAlcEh0OjoA7PSL5GLk.QXwMzN5wCO5IzW}



In [2]: 
```

# 03-ASCII

## 问题描述

源码：

```py
#!/opt/pwn.college/python

import random
import string
import sys

if not sys.stdin.isatty():
    print("You must interact with me directly. No scripting this!")
    sys.exit(1)

for n in range(1, 10):
    print(f"Challenge number {n}...")
    pt_chr, ct_chr = random.sample(
        string.digits + string.ascii_letters + string.punctuation,
        2
    )
    key = ord(pt_chr) ^ ord(ct_chr)

    print(f"- Encrypted Character: {ct_chr}")
    print(f"- XOR Key: {key:#04x}")
    answer = input("- Decrypted Character? ").strip()
    if answer != pt_chr:
        print("Incorrect!")
        sys.exit(1)

    print("Correct! Moving on.")

print("You have mastered XORing ASCII! Your flag:")
print(open("/flag").read())
```

## 思路

和上一个一样，依旧是自动化脚本。

虽然我想到了pwntools。

这些脚本都可以好好学一学，以后是一个很好的参考工具。

## EXP

```py
import pexpect
import string

# 启动目标程序
child = pexpect.spawn('/challenge/run', encoding='utf-8')

# 准备 ASCII 字符集
ascii_set = string.digits + string.ascii_letters + string.punctuation

try:
    for _ in range(9):
        # 等待并读取挑战提示
        child.expect(r'Challenge number (\d+)...')
        print(child.after)  # 打印 Challenge 信息

        # 提取 Encrypted Character 和 XOR Key
        child.expect(r'- Encrypted Character: (.)')
        encrypted_char = child.match.group(1)

        child.expect(r'- XOR Key: (0x[0-9a-fA-F]+)')
        xor_key = int(child.match.group(1), 16)

        # 解密得到原始字符
        decrypted_char = None
        for char in ascii_set:
            if ord(char) ^ xor_key == ord(encrypted_char):
                decrypted_char = char
                break

        if decrypted_char is None:
            print("Failed to decrypt the character!")
            break

        # 提交解密答案
        child.expect(r'- Decrypted Character\? ')
        child.sendline(decrypted_char)
        print(f"Decrypted: {decrypted_char}")

        # 检查结果
        child.expect(r'Correct!|Incorrect!')
        print(child.after)
        if "Incorrect!" in child.after:
            print("Decryption failed!")
            break

    # 获取最终 Flag
    child.expect(r'You have mastered XORing ASCII! Your flag:')
    child.expect(pexpect.EOF)
    print("Flag:")
    print(child.before)

except pexpect.exceptions.EOF:
    print("Program terminated unexpectedly.")
except pexpect.exceptions.TIMEOUT:
    print("Timeout occurred. Check the script or the target program.")

```

## 截图

尽管题目不让我们使用脚本，但是还是照用不误。

![image-20241125150303055](./04%20Cryptography.assets/image-20241125150303055.png)

# 04-ASCII Strings

## 问题描述

源码

```py
#!/opt/pwn.college/python

import random
import string
import sys

from Crypto.Util.strxor import strxor

valid_keys = "!#$%&()"
valid_chars = ''.join(
    c for c in string.ascii_letters
    if all(chr(ord(k)^ord(c)) in string.ascii_letters for k in valid_keys)
)

print(valid_keys, valid_chars)

for n in range(1, 10):
    print(f"Challenge number {n}...")

    key_str = ''.join(random.sample(valid_keys*10, 10))
    pt_str = ''.join(random.sample(valid_chars*10, 10))
    ct_str = strxor(pt_str.encode(), key_str.encode()).decode()

    print(f"- Encrypted String: {ct_str}")
    print(f"- XOR Key String: {key_str}")
    answer = input("- Decrypted String? ").strip()
    if answer != pt_str:
        print("Incorrect!")
        sys.exit(1)

    print("Correct! Moving on.")

print("You have mastered XORing ASCII! Your flag:")
print(open("/flag").read())
```

## 思路

依旧是gpt

## EXP

### subprocess版本

```PY
#!/opt/pwn.college/python

import subprocess
from Crypto.Util.strxor import strxor

# 模拟与挑战程序的交互
def solve_challenge():
    # 启动挑战脚本
    proc = subprocess.Popen(
        ["/challenge/run"],  # 替换为实际的脚本路径
        stdin=subprocess.PIPE,
        stdout=subprocess.PIPE,
        stderr=subprocess.PIPE,
        text=True
    )

    while True:
        # 读取一行输出
        line = proc.stdout.readline().strip()
        if not line:
            continue
        
        print(line)  # 打印调试信息
        
        # 检查是否遇到加密字符串
        if line.startswith("- Encrypted String:"):
            ct_str = line.split(": ")[1]
            key_str = proc.stdout.readline().strip().split(": ")[1]
            
            # 解密密文
            pt_str = strxor(ct_str.encode(), key_str.encode()).decode()

            # 输入解密后的字符串
            proc.stdin.write(pt_str + "\n")
            proc.stdin.flush()
        elif "Incorrect!" in line:
            print("Failed!")
            break
        elif "Your flag:" in line:
            print(proc.stdout.read())  # 打印最终的 flag
            break

# 运行自动化脚本
if __name__ == "__main__":
    solve_challenge()

```

### pexpect版本

```py
#!/usr/bin/env python3

import pexpect
from Crypto.Util.strxor import strxor

def solve_challenge():
    # 启动挑战脚本
    proc = pexpect.spawn('/challenge/run')  # 替换为实际的脚本路径
    
    # 读取挑战过程中的输出并根据提示进行交互
    while True:
        line = proc.readline().decode('utf-8').strip()
        
        if not line:
            continue
        
        print(line)  # 打印当前的输出（调试用）
        
        if line.startswith("- Encrypted String:"):
            ct_str = line.split(": ")[1]
            key_str = proc.readline().decode('utf-8').strip().split(": ")[1]
            
            # 解密密文
            pt_str = strxor(ct_str.encode(), key_str.encode()).decode()
            print(f"Decrypted string: {pt_str}")
            
            # 输入解密后的字符串
            proc.sendline(pt_str)
        
        elif "Incorrect!" in line:
            print("Failed!")
            break
        
        elif "Your flag:" in line:
            print(proc.read().decode('utf-8'))  # 打印最终的 flag
            break

if __name__ == "__main__":
    solve_challenge()

```



# 05-Base64

## 问题描述

源码

```py
#!/opt/pwn.college/python

from base64 import b64encode

flag = open("/flag", "rb").read()

print(f"Base64-Encoded Flag: {b64encode(flag).decode()}")
```

## 思路

学着使用python来进行解码

## EXP

```py
import base64

# Base64 编码的字符串
encoded_string = "cHduLmNvbGxlZ2V7d0RhTVhSb1dzM3FXNFRLVGF1X1BTdjYzc1VKLlFYemN6TXp3Q081SXpXfQo="

# 解码 Base64 字符串
decoded_bytes = base64.b64decode(encoded_string)

# 将字节数据转换为字符串
decoded_string = decoded_bytes.decode('utf-8')

print(f"Decoded string: {decoded_string}")

```

## 截图

![image-20241125154814418](./04%20Cryptography.assets/image-20241125154814418.png)

# 06-One-time Pad

## 问题描述

otp是可以证明的在一定条件下无法被破解的加密系统。

## EXP

```py
import base64

def decrypt_otp(key_b64, ciphertext_b64):
    """
    使用一次性密码本（OTP）解密密文。

    参数：
        key_b64 (str): Base64 编码的密钥。
        ciphertext_b64 (str): Base64 编码的密文。

    返回：
        str: 解密后的明文。
    """
    try:
        # Base64 解码
        key = base64.b64decode(key_b64)
        ciphertext = base64.b64decode(ciphertext_b64)

        # 确保密钥和密文长度一致
        if len(key) != len(ciphertext):
            raise ValueError("密钥长度和密文长度不匹配，无法解密")

        # XOR 解密
        plaintext_bytes = bytes([c ^ k for c, k in zip(ciphertext, key)])

        # 将解密结果转为字符串
        plaintext = plaintext_bytes.decode("utf-8")
        return plaintext

    except Exception as e:
        return f"解密失败: {str(e)}"

# 示例输入
key_b64 = "HsSDSFmRtfAC3TJJrM3gle49x3lxho6wq0bH8dVG5kTL1IVonLMcUIs+HG1/7SQ9kvZ9hxtLjvw="
ciphertext_b64 = "brPtZjr+2Zxnulcy/fu53LkEqC0Ht+OG8yKplJ4k3iyP5/Zarf1RftpmLA4FoF5K0blIzmEc8/Y="

# 解密并输出结果
plaintext = decrypt_otp(key_b64, ciphertext_b64)
print("解密结果:", plaintext)

```

## 截图

![image-20241127130358396](./04%20Cryptography.assets/image-20241127130358396.png)



# 07-Many-time Pad

## 问题描述



## 思路

ciphertext = strxor(flag, key[:len(flag)])

ciphertext    flag     key[:len(flag)]   属于xor关系

ciphertext   plaintext,      key[:len(plaintext)]    属于xor关系

flag 是 x 

key是y

z就是我们第一次收到的

然后我们输入的内容会再和y异或

那我们输入z，不就得到x了吗？

所以这题就是获取题目给的信息然后再发送过去，然后就得到flag。

## 前置知识

```
initial_ciphertext_b64 = p.recvline().strip().split(b": ")[1]
```



`p.recvline()` 是用于接收一行数据的函数，通常出现在与服务端通信的场景中（比如通过 `socket` 或 `pwntools` 进行交互）。它接收来自另一端的完整一行数据（包含换行符 `\n`），例如：

```
plaintext


Copy code
Flag Ciphertext (b64): vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI=
```

------



`strip()` 是一个字符串方法，用于移除字符串开头和结尾的空白字符（包括 `\n`、`\r` 等）。比如：

```
pythonCopy codeline = b"  Flag Ciphertext (b64): vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI=\n"
stripped_line = line.strip()
# stripped_line = b"Flag Ciphertext (b64): vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI="
```

------



`split(b": ")` 是一个将字符串按指定分隔符（此处为 `": "`）分割的方法。它会返回一个列表。例如：

```
pythonCopy codestripped_line = b"Flag Ciphertext (b64): vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI="
split_line = stripped_line.split(b": ")
# split_line = [b"Flag Ciphertext (b64)", b"vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI="]
```

------



`[1]` 是索引操作，从分割后的列表中提取第二个元素（索引从 0 开始）。例如：

```
pythonCopy codeciphertext_b64 = split_line[1]
# ciphertext_b64 = b"vPi9xH6qw3Azcok3+Dk93j8jLmDMT3kQe5zDNpxOyKjfZQUXUAosRNJfT8F0zx9/WM3/NcHSBZI="
```

------



## EXP

纯手搓代码

```py
from base64 import b64encode, b64decode
from Crypto.Random import get_random_bytes
from Crypto.Util.strxor import strxor
from pwn import *

# flag x
# key  y
#      z

def my_data_processing(xxx):
    print(xxx)
    print(type(xxx))

def solve_challenge():
    p = process("/challenge/run")

    b64_z = p.recvline().strip().split(b": ")[1]
    my_data_processing(b64_z)

    str_b64_z = b64_z.decode()
    my_data_processing(str_b64_z)


    z = b64decode(str_b64_z)
    my_data_processing(z)

    p.sendline(str_b64_z)

    x = p.recvline()
    my_data_processing(x)

    x = x.strip().split(b': ')[2]
    my_data_processing(x)

    x=x.decode()
    my_data_processing(x)

    x=b64decode(x)
    my_data_processing(x)

solve_challenge()
```



## 截图

很有成就感，而且用上了pwntools，慢慢学了。

![image-20241127153828074](./04%20Cryptography.assets/image-20241127153828074.png)

# 08-AES

## 思路

这个ECB模式下，aes是采用对数据分块，如果不能对齐的话，就pad，使数据可以分成完整的块。

随后每一个块都与key（同样的key）进行运算。然后把结果拼接在一起。

## EXP

还是手搓，很爽

```py
from base64 import b64encode, b64decode
from Crypto.Random import get_random_bytes
from Crypto.Util.strxor import strxor
from pwn import *
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Random import get_random_bytes
from Crypto.Util.Padding import unpad

# flag x
# key  y
#      z

def my_data_processing(xxx):
    print(xxx)
    print(type(xxx))

def solve_challenge():
    p = process("/challenge/run")

    key = p.recvline().strip().split(b": ")[1]
    key = b64decode(key)
    my_data_processing(key)


    secret = p.recvline().strip().split(b": ")[1]
    secret = b64decode(secret)
    my_data_processing(secret)

    new_aes = AES.new(key=key, mode=AES.MODE_ECB)
    
    flag = unpad(new_aes.decrypt(secret), AES.block_size)
    my_data_processing(flag)

    # new_aes.decrypt()

solve_challenge()
```



## 截图

![image-20241127160304291](./04%20Cryptography.assets/image-20241127160304291.png)

# 09-AES-ECB-CPA

## 问题描述

```py
#!/opt/pwn.college/python

from base64 import b64encode, b64decode
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad
from Crypto.Random import get_random_bytes

flag = open("/flag", "rb").read()

key = get_random_bytes(16)
cipher = AES.new(key=key, mode=AES.MODE_ECB)

while True:
    print("Choose an action?")
    print("1. Encrypt chosen plaintext.")
    print("2. Encrypt part of the flag.")
    if (choice := int(input("Choice? "))) == 1:
        pt = input("Data? ").strip().encode()
    elif choice == 2:
        index = int(input("Index? "))
        length = int(input("Length? "))
        pt = flag[index:index+length]
    else:
        break

    ct = cipher.encrypt(pad(pt, cipher.block_size))
    print(f"Result: {b64encode(ct).decode()}")
```

## 思路

这东西真不好想，跟爆破一样。

就是首先要注意，程序每次运行所有数值都会变化。所以process只能一次。

flag和key加密然后给我们加密后的值

flag当然是可视化字符了。

然后我们可以将所有的可视化字符都加密一遍，得到一个对应的键值对库。

然后我们根据choice2，每次只看flag的一个字符加密后的内容。然后和我们的键值对库比较，value匹配上，那么久知道flag的字符。

## EXP

```py
from base64 import b64encode, b64decode
from Crypto.Random import get_random_bytes
from Crypto.Util.strxor import strxor
from pwn import *
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad

known_plaintext_ciphertext = {}
flag = ""

p = process("/challenge/run")

def my_data_processing(xxx):
    print(f"[*]info : {xxx}")
    print(xxx)
    print(type(xxx))

def choice1(p, data):
    global known_plaintext_ciphertext
    p.recvuntil("Choice? ")
    p.sendline(str(1))
    p.recvuntil("Data? ")
    p.sendline(chr(data))
    result = p.recvline().strip().split(b": ")[1]
    # Save the result to the dictionary and file
    known_plaintext_ciphertext[data] = result
    my_data_processing(known_plaintext_ciphertext)

def choice2(p, index, length):
    p.recvuntil("Choice? ")
    p.sendline(str(2))
    p.recvuntil("Index? ")
    p.sendline(str(index))
    p.recvuntil("Length? ")
    p.sendline(str(length))

    result = p.recvline().strip().split(b": ")[1]
    # Compare result with known plaintext-ciphertext pairs
    for key, value in known_plaintext_ciphertext.items():
        if result == value:
            print(f"[*] Found matching ciphertext for {key}")
            global flag
            flag += chr(key)  # Convert key to string and concatenate
            break  # Once a match is found, stop further checks

def solve_challenge():
    global p
    # 输出所有可视化字符（ASCII 32-126）
    for i in range(32, 127):
        choice1(p, i)
    
    for i in range(0, 100):
        choice2(p, i, 1)  # Execute choice2

if __name__ == "__main__":
    solve_challenge()
    print(f"[*] flag string: {flag}")

```



又是他妈的手搓密码学的一天

## 截图

![image-20241128182132755](./04%20Cryptography.assets/image-20241128182132755.png)

# 10-AES-ECB-CPA-HTTP

## 思路

和上一题思路基本上一模一样，就是我刚开始不知道substr的用法。

只有将flag一个一个的字符的密文显示出来，才能爆破。

第一波获取单个字符的明文密文映射库

第二部就是爆破

## EXP

```py
from pwn import *
from base64 import b64encode, b64decode
from Crypto.Random import get_random_bytes
from Crypto.Util.strxor import strxor
from Crypto.Cipher import AES
from Crypto.Util.Padding import pad, unpad
import requests

known_plaintext_ciphertext = {}
flag = ""

def my_data_processing(xxx):
    print(f"[*]info : {xxx}")
    print(xxx)
    print(type(xxx))

def prepare(i):
    global known_plaintext_ciphertext
    x = chr(i)
    response = requests.get(f"http://challenge.localhost?query='{x}'")
    # print(f"Response Text: {response.text}")
    # print("TYPE OF TEXT : " + str(type(response.text)))
    x = response.text.strip().split("<pre>")
    if len(x) <= 2:
        return
    x = x[2].strip().split("/<pre>")
    x = x[0][:24]
    known_plaintext_ciphertext[chr(i)] = x
    

def compare(index):
    global known_plaintext_ciphertext
    x = f"SUBSTR(flag,{index},1)"
    # x = f"SUBSTR(flag,{index},1)"
    response = requests.get(f"http://challenge.localhost?query={x}")
    x = response.text.strip().split("<pre>")
    print(x)
    if len(x) <= 2:
        return
    x = x[2].strip().split("/<pre>")
    x = x[0][:24]
    # print(x)
    for key, value in known_plaintext_ciphertext.items():
        if x == value:
            print(f"[*] Found matching ciphertext for {key}")
            global flag
            flag += key # Convert key to string and concatenate
            break  # Once a match is found, stop further checks
    
def solve_challenge():
    for i in range(32, 127):
        prepare(i)
    print(known_plaintext_ciphertext)

    # flag长度为55，所以sql的substr的index是1-55，所以range是(1,56)
    for i in range(1,56):
        compare(i)

if __name__ == "__main__":
    solve_challenge()
    #print(known_plaintext_ciphertext)
    print(flag)
```



# 11-AES-ECB-CPA-Suffix

## 前置知识

`pt = flag[-length:]` 是 Python 中的一个切片操作，用于从字符串 `flag` 中提取一个子字符串。具体含义如下：

- `flag` 是一个字符串（可能是某个标志或密文）。
- `length` 是一个整数，表示要提取的子字符串的长度。
- `flag[-length:]` 从字符串 `flag` 的末尾开始，提取长度为 `length` 的子字符串。

例子：

如果 `flag = "this_is_a_test_flag"`，`length = 4`，那么：

```py
pt = flag[-4:]
```

将会从 `flag` 的末尾提取出 `4` 个字符，结果是：

```py
pt = "flag"
```

因此，`pt` 将保存字符串 `"flag"`。

切片的语法是 `[start:end]`，其中 `start` 是开始索引（包含该位置），`end` 是结束索引（不包含该位置）。如果 `start` 是负数，表示从字符串的末尾倒数计数。

python  将所有ascii可视化字符放在一个列表里



ChatGPT said:

ChatGPT

你可以通过以下 Python 代码将所有 ASCII 可视字符（从 32 到 126）放在一个列表中：

```py
# 获取所有 ASCII 可视字符
ascii_visible_chars = [chr(i) for i in range(32, 127)]

# 打印结果
print(ascii_visible_chars)
```

解释：

- `chr(i)` 用于将一个整数 `i` 转换为相应的 ASCII 字符。

- ```py
  range(32, 127)
  ```

   生成从 32 到 126 的整数，这些整数代表了可视的 ASCII 字符。

  - ASCII 码 32 是空格字符，126 是波浪号 `~`。

执行后，`ascii_visible_chars` 将包含所有从空格到波浪号之间的可视字符列表。

ChatGPT said:

ChatGPT

在 Python 中，`process` 通常是指通过 `pwntools` 库创建的进程。你的代码示例中，`process("py", arg"")` 看起来是想用 `process` 启动一个进程并传递参数，但语法有些不对。正确的用法是：

```py
from pwn import *

# 启动进程并传递参数
p = process(["python", "your_script.py", "arg1", "arg2"])
```

解释：

- `process()` 是 `pwntools` 中用来启动子进程的函数。
- `"python"` 是你要运行的命令，通常是 Python 解释器的路径。
- `"your_script.py"` 是你要运行的 Python 脚本。
- `"arg1", "arg2"` 是你要传递给脚本的参数。

如果你不使用 `pwntools`，而只是想要启动一个简单的 Python 进程并传递参数，可以使用 `subprocess` 模块：

```py
import subprocess

# 启动进程并传递参数
p = subprocess.Popen(["python", "your_script.py", "arg1", "arg2"])
```

这里的 `subprocess.Popen()` 也可以用来启动进程并传递参数。

总结：

- 使用 `process()` 时要传递一个包含命令和参数的列表。
- `subprocess.Popen()` 也有类似的用法。

## 思路

我他妈的真牛逼

这就是纯爆破

注意是倒序  pwn.college{this_a_fake_flag}

倒序，爆破的时候注意}是在最后一位，字符要加载}的前边不是后边。

## EXP

```py
from pwn import *
import base64

# 假设你已经能通过 pwn 模块与服务器进行交互
# p = process(["python3", "./run"])
p = process(["/challenge/run"])

context.log_level = "debug"

flag = "}"
ascii_visible_chars = [chr(i) for i in range(32, 127)]  # 可视字符集

def my_data_processing(name, data):
    print(f"[*]info : {name}")
    print(data)
    print("--------------------")

# 获取尾部密文
def get_flag_tail_ciphertext(length):
    p.recvuntil(b"Choice? ")
    p.sendline(b"2")  # 选择2，加密 flag 的尾部
    p.recvuntil(b"Length? ")
    p.sendline(str(length).encode())  # 发送请求的长度
    # print(p.recv())
    
    result = p.recvline().strip().split(b": ")[1]
    # ciphertext = base64.b64decode(result)
    # print(f"[*] Tail ciphertext (length {length}): {ciphertext}")  # 输出密文进行调试
    print(result)
    return result

# 获取普通明文的密文
def encrypt_plaintext(plaintext):
    p.recvuntil(b"Choice? ")
    p.sendline(b"1")  # 选择1，加密自定义的明文
    p.recvuntil(b"Data? ")
    p.sendline(plaintext)  # 发送数据
    result = p.recvline().strip().split(b": ")[1]
    # ciphertext = base64.b64decode(result)
    # print(f"[*] Encrypting plaintext: {plaintext} -> {ciphertext}")  # 输出加密结果进行调试
    print(result)
    return result

# 攻击函数
def solve_challenge():
    global flag

    # 假设flag最大长度为55
    for i in range(2, 56):  # 尝试从1个字符到55个字符
        # 获取当前尾部密文
        flag_tail_ct = get_flag_tail_ciphertext(i)
        print(flag_tail_ct)
        
        for char in ascii_visible_chars:
                tem_flag = char + flag 
                print(tem_flag)
                choice1_result = encrypt_plaintext(tem_flag)
                print(choice1_result)
                if choice1_result == flag_tail_ct:
                        flag = char + flag
                        break

if __name__ == "__main__":
    solve_challenge()
    print(f"[*] Final flag: {flag}")  # 输出最终得到的 flag
    
```



# 12-AES-ECB-CPA-Prefix

## 思路

### 基本概念

- 明文，秘钥，密文

  - 明文：plain_text

  - 秘钥：key

  - 密文：cipher_text

### 加密算法说明

在aes的ecb模式下，明文按照16个字节分块。设明文长度为n

1. 如果n/16能整除，那么明文长度变为n+16(补充16个0x10)。然后加密，加密后ciphertext长度为n+16
2. 如果n/16不能整除。假设最后一个块有x个字节，补充16-x个字节，每个字节的值都是16-x。明文长度变为n+16-x。加密后密文长度为n+16-x。

这一题`不考虑n/16能整除然后还要补充`的这个情况。也就是说：

1. 如果n/16能整除，那么明文长度还是n。然后加密。加密后ciphertext长度还是n。
2. 如果n/16不能整除。假设最后一个块有x个字节，补充16-x个字节，每个字节的值都是16-x。明文长度变为n+16-x。加密后密文长度为n+16-x。

### 1.获取到padding长度

首先第一步，获取到使得flag长度整除16的padding的长度。设这个padding的长度为n:

- 如果明文长度整除16，那么多出一个字节就会导致多出一个块。
  1. 当n=0的时候，有4个block
  2. 当n=0 + 1 的时候，有5个block

- 如果明文长度不整除16，那么就要找到这个n：

  1. 当n的时候，有4个block

  1. 当n+1的时候，有5个block

遍历就可以找到了。

### 2.解密步骤

找到这个值之后，我们就可以开始解密了。

假设flag长度为54，那么padding长度就是10。

1. 第1步，获取到flag的倒数第1个字节的加密值。

   我们需要填充11个字符。

   然后我们会在block5得到flag的最后一个字节的加密值。

2. 第2步，我们需要遍历所有的可视化字符，获取到它们加密后的内容，构成一个密码本，形如：

   ```
   {"a" : "bakjbfoaou", "b" : "cakjbfoaou"........}
   ```

   

3. 第3步，我们进行对比，获取到flag的倒数第1个字节。

4. 第4步，获取到flag的倒数第2个字节的加密值。

   我们需要填充12个字符。

   然后我们会在block5得到flag的倒数2个字节的加密值。

5. 第5步，我们需要右侧添加已经找到的字节，遍历所有的可视化字符，获取到它们加密后的内容，构成一个密码本，形如：{"a}" : "bakjbfoaou", "b}" : "cakjbfoaou"........}

6. 第6步，我们进行对比，获取到flag的倒数第2个字节。

7. 第7步，一直循环，直至我们找到flag的倒数第16个字节。

8. 随后，我们需要截取已知的flag的长度，保持单字节爆破。

   第8步，我们要明白：我们始终只需要block5的加密内容。

9. 第9步，我们需要截取已知的flag的长度为前15，保持单字节爆破。

   右侧添加已经找到的字节，遍历所有的可视化字符，获取到它们加密后的内容，构成一个密码本，形如：

   ```
   {"aeadbeefdeadbeef":"aakjbfoaou","beadbeefdeadbeef":"bakjbfoaou","ceadbeefdeadbeef":"cakjbfoaou"........}
   ```

   

10. 第10步，我们进行对比，获取到flag的倒数某个字节。



---



## EXP

```py
from pwn import *
import base64


context.log_level = "debug"


p = process(["/challenge/run"])


flag = ""


ascii_visible_chars = ['}', '!', '#', '$', '%', '&', '(', ')', '*', '+', ',', '-', '.', '/', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ':', ';', '<', '=', '>', '?', '@', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '[', '\\', ']', '^', '_', '`', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '{', '|',  '~']


def query(x):
    p.recvuntil(b"Choice? ")
    p.sendline(b"2")
    p.recvuntil(b"Data? ")
    p.sendline(b'a' * x)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    return len(result)
    
    
def get_the_length_of_flag():
    target_value = -1
    for i in range(0,20) :
        result = query(i)
        if result == 108:
            target_value = i
            break
    print(target_value)
    return target_value - 1


def choice1(data):
    p.recvuntil(b"Choice? ")
    p.sendline(b"1")
    p.recvuntil(b"Data? ")
    p.sendline(data)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    print(result)
    return result

def choice2(data):
    p.recvuntil(b"Choice? ")
    p.sendline(b"2")
    p.recvuntil(b"Data? ")
    p.sendline(data)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    result = base64.b64decode(result)
    print(result)
    print(len(result))
    block5 = base64.b64encode(result[64:])
    print(block5)
    # p.recvuntil(b"Block 5: ")
    # block5 = p.recvuntil(b"\n")[:-1]
    # print(block5)
    return block5


def solve():
    # 声明flag'
    global flag
    
    # 第一步，获取到padding的长度
    flag_length = get_the_length_of_flag()
    
    # 第一轮遍历
    # flag长度为0
    # 确认pad的长度范围
    for i in range(flag_length + 1, flag_length + 64):
        padding = b'a' * i
        # 获取到block5的内容
        block5 = choice2(padding)
        for x in ascii_visible_chars:
            temp = flag if len(flag) <= 15 else flag[:15]
            temp = x + flag
            encode_content = choice1(temp)
            if block5 == encode_content:
                flag = x + flag
                print("find one flag!!!!!!!!!!!!!!!!!!!!!!!!!")
                break

                
''' 
就是找到flag的长度，一定是四个block。然后我们补齐前边，让flag最后一个字符成为第五块，然后和已知可视化字符匹配，然后再pad一个，获取倒数第二个，一直到16个，然后再pad一个，这时候就是6个block，第六个block是}。然后第五个block的后15个我们都知道。我们再得到一个字符，然后再pad，在pad，直到32个flag字符都获取到。然后再pad，第七个block是}。然后第五个block的后15个我们都知道。然后再搞flag的48个字符。
'''

if __name__ == "__main__" :
    solve()
    


```



## 截图

我他妈的是个天才！

![image-20241207165436563](./04%20Cryptography.assets/image-20241207165436563.png)



# 13-AES-ECB-CPA-Prefix-2

## 思路

在aes的ecb模式下，明文按照16个字节分块。设明文长度为n

1. 如果n/16能整除，那么明文长度变为n+16(补充16个0x10)。然后加密，加密后ciphertext长度为n+16
2. 如果n/16不能整除。假设最后一个块有x个字节，补充16-x个字节，每个字节的值都是16-x。明文长度变为n+16-x。加密后密文长度为n+16-x。

与上一题的区别就在于，查找这个padding的长度时，要改一改你的算法。

## EXP

```py
from pwn import *
import base64


context.log_level = "debug"


p = process(["/challenge/run"])


flag = ""


ascii_visible_chars = ['}', '!', '#', '$', '%', '&', '(', ')', '*', '+', ',', '-', '.', '/', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ':', ';', '<', '=', '>', '?', '@', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '[', '\\', ']', '^', '_', '`', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '{', '|',  '~']


def query(x):
    p.recvuntil(b"Choice? ")
    p.sendline(b"2")
    p.recvuntil(b"Data? ")
    p.sendline(b'a' * x)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    return len(result)
    
    
def get_the_length_of_flag():
    target_value = -1
    for i in range(0,20) :
        result = query(i)
        if result == 108:
            target_value = i
            break
    print(target_value)
    return target_value


def choice1(data):
    p.recvuntil(b"Choice? ")
    p.sendline(b"1")
    p.recvuntil(b"Data? ")
    p.sendline(data)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    print(result)
    return result

def choice2(data):
    p.recvuntil(b"Choice? ")
    p.sendline(b"2")
    p.recvuntil(b"Data? ")
    p.sendline(data)
    p.recvuntil(b"Result: ")
    result = p.recvuntil(b"\n")[:-1]
    print(result)
    print(len(result))
    result = base64.b64decode(result)
    print(result)
    print(len(result))
    block5 = base64.b64encode(result[64:])
    print(block5)
    # p.recvuntil(b"Block 5: ")
    # block5 = p.recvuntil(b"\n")[:-1]
    # print(block5)
    return block5


def solve():
    # 声明flag'
    global flag
    
    # 第一步，获取到padding的长度
    flag_length = get_the_length_of_flag()
    
    # 第一轮遍历
    # flag长度为0
    # 确认pad的长度范围
    for i in range(flag_length + 1, flag_length + 64):
        padding = b'a' * i
        # 获取到block5的内容
        block5 = choice2(padding)
        for x in ascii_visible_chars:
            temp = flag if len(flag) <= 15 else flag[:15]
            temp = x + flag
            encode_content = choice1(temp)
            if block5 == encode_content:
                flag = x + flag
                print("find one flag!!!!!!!!!!!!!!!!!!!!!!!!!")
                break
                
                
if __name__ == "__main__" :
    solve()

```



## 截图

如图

![image-20241207231726665](./04%20Cryptography.assets/image-20241207231726665.png)

# 14-AES-ECB-CPA-Prefix-Miniboss

## 思路

明文按照16个字节分块。设明文长度为n

1. 如果n/16能整除，那么明文长度变为n+16(补充16个0x10)。然后加密。加密后ciphertext长度为n+16
2. 如果n/16不能整除，那就看最后一个块有x个字节。那就补充16-x个16-x。然后明文长度就变为n+16-x。加密后密文长度为n+16-x。

首先第一步，获取到padding的长度。

然后获取某个字节的密文，然后我们自己构造和爆破。

## EXP

```py
from pwn import *
import base64


# context.log_level = "debug"


p = process(["/challenge/run"])


flag = ""


ascii_visible_chars = ['}', '!', '#', '$', '%', '&', '(', ')', '*', '+', ',', '-', '.', '/', '0', '1', '2', '3', '4', '5', '6', '7', '8', '9', ':', ';', '<', '=', '>', '?', '@', 'A', 'B', 'C', 'D', 'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L', 'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T', 'U', 'V', 'W', 'X', 'Y', 'Z', '[', '\\', ']', '^', '_', '`', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z', '{', '|',  '~']


def query(x):
    p.recvuntil(b"Data? ")
    p.sendline(base64.b64encode(b'a' * x))
    p.recvuntil(b"Ciphertext: ")
    result = p.recvuntil(b"\n")[:-1]
    return len(result)
    
    
def get_the_length_of_init_padding():
    target_value = -1
    for i in range(0,20) :
        result = query(i)
        if result > 88:
            target_value = i
            break
    # print(target_value)
    return target_value


# 爆破专用
def choice_1(data):
    p.recvuntil(b"Data? ")
    p.sendline(base64.b64encode(data))
    p.recvuntil(b"Ciphertext: ")
    result = p.recvuntil(b"\n")[:-1]
    # print(result)
    # print(len(result))
    result = base64.b64decode(result)
    # print(result)
    # print(len(result))
    result = base64.b64encode(result[:16])
    # print(result)
    # print(len(result))
    return result


# 获取到flag的某个字节的加密内容
def choice_2(data):
    p.recvuntil(b"Data? ")
    p.sendline(base64.b64encode(data))
    p.recvuntil(b"Ciphertext: ")
    result = p.recvuntil(b"\n")[:-1]
    # print(result)
    # print(len(result))
    result = base64.b64decode(result)
    # print(result)
    # print(len(result))
    result = base64.b64encode(result[64:80])
    # print(result)
    # print(len(result))
    return result


def solve():
    global flag

    init_length = get_the_length_of_init_padding()

    for i in range(init_length + 1, init_length + 64):
        result2 = choice_2(b'a' * i)
        for x in ascii_visible_chars:
            if len(flag) < 15:
                # 这就需要我们自己来构造
                padding = x + flag
                number = 16 - len(padding)
                value = bytes([number])
                payload = padding.encode() + value * number
            else:
                payload = (x + flag[:15]).encode()
            
            result1 = choice_1(payload)

            if result1 == result2:
                flag = x + flag
        print(flag)
                


if __name__ == "__main__" :
    solve()

```

## 截图

太酷了

![image-20241208193941234](./04%20Cryptography.assets/image-20241208193941234.png)

# 15-AES-ECB-CPA-Prefix-Boss



# 16-AES-CBC

# 17-AES-CBC Tampering

# 18-AES-CBC Resizing

# 19-AES-CBC-POA

# 20-AES-CBC-POA-Encrypt

# 21-DHKE

# 22-DHKE-to-AES

# 23-RSA 1











# 24-RSA 2

# 25-RSA Signatures

# 26-SHA 1

# 27-SHA 2

# 28-RSA 3

# 29-RSA 4

# 30-TLS 1

# 31-TLS 2

# 
