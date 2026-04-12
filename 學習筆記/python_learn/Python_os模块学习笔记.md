# Python os 模块学习笔记

## 1. os 是什么？

`os` 是 `operating system`（操作系统）的缩写，是 Python 的标准库，不需要额外安装。

**简单理解：os 就是让你用代码代替手动操作文件管理器。**

新建文件夹、删除文件、重命名、查看文件夹内容、检查文件是否存在……这些你平时在电脑上手动做的事，os 模块都能用代码完成。

### 为什么要用 os？

在爬虫场景下特别有用。比如：

- 要下载 1000 首歌，不可能手动建文件夹 → `os.mkdir()` 一行搞定
- 下载前检查文件是否已下载过 → `os.path.exists()` 避免重复下载
- 查看已下载了多少文件 → `os.listdir()`

### 导入注意事项

```python
# ✅ 正确做法
import os

# ✅ 也可以只导入需要的
from os import mkdir, listdir

# ❌ 千万不要这样做！
from os import *
# 这会导致 os.open() 覆盖内置的 open() 函数，造成预料之外的错误
```

> **规则：任何模块都不建议用 `from xxx import *`**，因为你不知道它里面有什么名字，可能会悄悄覆盖掉已有的变量或内置函数，出了 bug 还特别难查。

---

## 2. os.name — 获取操作系统类型

```python
import os
print(os.name)
```

只有三种可能的返回值：

| 返回值 | 含义 |
|--------|------|
| `'posix'` | Mac 和 Linux |
| `'nt'` | Windows |
| `'java'` | Jython（基本没人用） |

### 实际用途

```python
if os.name == 'nt':
    os.system('cls')      # Windows 清屏
else:
    os.system('clear')    # Mac/Linux 清屏
```

### 更细的区分

`os.name` 分不清 Mac 和 Linux，需要更细的区分可以用 `platform` 模块：

```python
import platform
print(platform.system())
# 'Darwin'  → Mac
# 'Linux'   → Linux
# 'Windows' → Windows
```

---

## 3. os.environ — 环境变量

`os.environ` 是一个字典，存着系统里所有的环境变量。

### 什么是环境变量？

环境变量是操作系统预先设置好的一些全局信息。当你的电脑启动、你登录账户的时候，系统就自动设置了一堆环境变量。这不是 Python 设置的，是操作系统本身就有的。

### 查看所有环境变量

```python
for key, value in os.environ.items():
    print(f'{key} = {value}')
```

常见的环境变量：

| 变量名 | 含义 |
|--------|------|
| `HOME` | 用户主目录（如 `/Users/qiuzhulin`） |
| `PATH` | 系统查找命令的路径列表 |
| `USER` | 当前用户名 |
| `SHELL` | 终端类型（如 `/bin/zsh`） |
| `LANG` | 系统语言设置 |

### 读取

```python
os.environ.get('HOME')       # 推荐，不存在返回 None
os.environ['HOME']           # 不存在会报错 KeyError
os.environ.get('HOME', '/默认路径')  # 不存在返回默认值
```

### 设置和删除

```python
os.environ['MY_KEY'] = 'hello'  # 设置（仅当前程序运行期间有效）
del os.environ['MY_KEY']        # 删除
```

> 注意：代码里设置的环境变量，程序结束就没了，不会真的改系统的环境变量。

### 最常见用途：存放敏感信息

避免把 API 密钥、数据库密码直接写在代码里：

```python
# ❌ 危险做法：密钥写死在代码里，上传 GitHub 就泄露了
api_key = "sk-abc123456789"

# ✅ 安全做法：从环境变量读取
api_key = os.environ.get('API_KEY')
if not api_key:
    print('请先设置 API_KEY 环境变量')
```

### 另一个用途：区分环境

```python
# 本地没设置就用 localhost，服务器上设置了就用服务器的地址
db_host = os.environ.get('DB_HOST', 'localhost')
```

### 安全性

`os.environ` 只是读取系统已有的环境变量，不会改动系统，自己本地学习随便玩。但注意 **不要把读到的敏感信息截图发到网上**。

---

## 4. 文件夹操作

### 4.1 os.mkdir() — 创建单层文件夹

```python
os.mkdir('新文件夹')          # 创建一个文件夹
os.mkdir('a/b/c')            # ❌ 报错！只能建一层
```

如果文件夹已存在会报错 `FileExistsError`，所以通常搭配判断：

```python
if not os.path.exists('下载'):
    os.mkdir('下载')
```

### 4.2 os.makedirs() — 递归创建多层文件夹

```python
os.makedirs('a/b/c')         # ✅ 一次性把 a、b、c 三层都建好
```

加上 `exist_ok=True` 避免重复创建报错：

```python
os.makedirs('下载/音乐/周杰伦', exist_ok=True)
```

> **实际使用中 `makedirs` 更常用**，一行代码就能替代 `if not exists` + `mkdir` 的两行写法。

### 4.3 os.rmdir() — 删除空文件夹

```python
os.rmdir('空文件夹')          # 只能删空的，里面有东西会报错
```

要删除非空文件夹，需要用 `shutil.rmtree()`。

### 4.4 os.removedirs() — 递归删除

```python
os.removedirs('a/b/c')       # 从最下级开始逐级删除，遇到非空目录即停止
```

---

## 5. 文件操作

### 5.1 os.remove() — 删除文件

```python
os.remove('文件.txt')         # 删除后无法恢复，不会进回收站！
```

> 如果指定的是目录而非文件，会抛出 `IsADirectoryError`。

### 5.2 os.rename() — 重命名 / 移动

```python
os.rename('旧名.txt', '新名.txt')              # 重命名
os.rename('a.txt', '子文件夹/a.txt')            # 移动文件
```

> 中间路径必须存在，否则报错。递归版本 `os.renames()` 能创建缺失的中间路径。

---

## 6. 获取路径信息

### 6.1 os.getcwd() — 获取当前工作目录

```python
print(os.getcwd())
# 例如: /Users/qiuzhulin/python_learn
```

`getcwd` = get the current working directory

### 6.2 os.chdir() — 切换工作目录

```python
os.chdir('/Users/qiuzhulin/Documents')
print(os.getcwd())  # /Users/qiuzhulin/Documents

os.chdir('..')       # 切换到上一级目录
```

### 6.3 os.listdir() — 列出目录内容

```python
os.listdir('.')      # 列出当前目录下所有文件和文件夹
os.listdir('/Users') # 列出指定目录
```

> 只列出一层，不会进入子文件夹。

---

## 7. os.walk() — 递归遍历目录树

`os.walk()` 会递归遍历一个目录下的**所有**文件和子文件夹。

假设有这样的目录结构：

```
项目/
├── a.py
├── b.txt
├── 子文件夹1/
│   ├── c.py
│   └── d.txt
└── 子文件夹2/
    └── e.mp3
```

```python
for 当前路径, 文件夹列表, 文件列表 in os.walk('项目'):
    print(f'当前在: {当前路径}')
    print(f'里面的文件夹: {文件夹列表}')
    print(f'里面的文件: {文件列表}')
    print('---')
```

输出：

```
当前在: 项目
里面的文件夹: ['子文件夹1', '子文件夹2']
里面的文件: ['a.py', 'b.txt']
---
当前在: 项目/子文件夹1
里面的文件夹: []
里面的文件: ['c.py', 'd.txt']
---
当前在: 项目/子文件夹2
里面的文件夹: []
里面的文件: ['e.mp3']
---
```

### 实用例子：找出所有 .py 文件

```python
for root, _, files in os.walk('.'):
    for filename in files:
        if filename.endswith('.py'):
            full_path = os.path.join(root, filename)
            print(full_path)
```

> `_` 表示"这个变量我不用"，是 Python 的常见写法。

### os.walk() vs os.listdir() 的区别

| 特性 | os.listdir() | os.walk() |
|------|-------------|-----------|
| 遍历深度 | 只看一层 | 递归遍历所有层 |
| 返回值 | 列表 | 迭代器（三元组） |
| 适用场景 | 简单列出文件 | 搜索整个目录树 |

---

## 8. os.path 子模块（最常用）

`os.path` 中的函数基本上是纯粹的字符串操作，传入的参数甚至不需要是一个有效路径。

### 8.1 os.path.join() — 路径拼接

自动处理不同系统的斜杠问题：

```python
os.path.join('下载', '音乐', '歌曲.mp3')
# Mac/Linux: 下载/音乐/歌曲.mp3
# Windows:   下载\音乐\歌曲.mp3
```

**重要规则：遇到绝对路径，前面的全部丢弃！**

```python
os.path.join("just", "do", "d:/", "python", "dot", "g:/", "com")
# 结果: 'g:/com'
```

过程拆解：

```
"just" + "do"       → just/do
+ "d:/"             → d:/            （绝对路径，前面丢弃）
+ "python" + "dot"  → d:/python/dot
+ "g:/"             → g:/            （又遇到绝对路径，前面丢弃）
+ "com"             → g:/com
```

> 所以 **除了第一个参数，后面的参数不要带绝对路径**。

### 8.2 os.path.abspath() — 获取绝对路径

两种情况：

```python
# 传入绝对路径：直接原样返回（不检查是否存在）
os.path.abspath("a:/just/do/python")
# 'a:\\just\\do\\python'（即使 a 盘不存在）

# 传入相对路径：用当前工作目录拼接
os.path.abspath("ityouknow")
# '/Users/qiuzhulin/python_learn/ityouknow'
# 等价于 os.path.join(os.getcwd(), "ityouknow")
```

> **`abspath` 不会检查路径是否真的存在**，想判断要用 `os.path.exists()`。

### 8.3 os.path.basename() 和 os.path.dirname()

```python
os.path.basename('/Users/qiuzhulin/音乐/歌曲.mp3')
# '歌曲.mp3'（只取文件名）

os.path.dirname('/Users/qiuzhulin/音乐/歌曲.mp3')
# '/Users/qiuzhulin/音乐'（只取目录部分）
```

注意尾部斜杠的影响：

```python
os.path.basename('/ityouknow/IAmBasename/')
# ''（空字符串，因为最后一个分隔符之后没有内容）
```

### 8.4 os.path.split() — 分离目录和文件名

```python
os.path.split('/Users/qiuzhulin/音乐/歌曲.mp3')
# ('/Users/qiuzhulin/音乐', '歌曲.mp3')
```

> `basename()` 和 `dirname()` 本质上就是 `split()` 返回值的第二和第一个元素。

### 8.5 os.path.splitext() — 分离文件名和扩展名

```python
os.path.splitext('歌曲.mp3')
# ('歌曲', '.mp3')
```

实用场景：批量改扩展名

```python
for f in os.listdir('.'):
    name, ext = os.path.splitext(f)
    if ext == '.txt':
        os.rename(f, name + '.md')
```

### 8.6 os.path.exists() — 判断路径是否存在

```python
os.path.exists('.')           # True
os.path.exists('./不存在的')   # False
```

### 8.7 os.path.isfile() 和 os.path.isdir()

```python
os.path.isfile('a.py')    # True（是文件）
os.path.isdir('文件夹')    # True（是文件夹）
```

> 与 `exists()` 不同，**这两个函数会核验路径的有效性**，无效路径始终返回 False。

### 8.8 os.path.isabs() — 判断是否是绝对路径

```python
os.path.isabs('/Users/qiuzhulin')  # True
os.path.isabs('相对路径')            # False
```

> 仅检测格式，不核验路径是否真实存在。

### 8.9 os.path.getsize() — 获取文件大小

```python
size = os.path.getsize('歌曲.mp3')
print(f'文件大小: {size / 1024:.1f} KB')
```

---

## 9. 实战案例：文件整理器

```python
import os

rules = {
    '.mp3': 'music',
    '.jpg': 'images',
    '.png': 'images',
    '.txt': 'docs',
    '.pdf': 'docs',
}

for filename in os.listdir('.'):
    if not os.path.isfile(filename):
        continue
    _, ext = os.path.splitext(filename)
    if ext in rules:
        folder = rules[ext]
        os.makedirs(folder, exist_ok=True)
        os.rename(filename, os.path.join(folder, filename))
        print(f'{filename} → {folder}/')
```

这个脚本用到了 `listdir`、`isfile`、`splitext`、`makedirs`、`rename`、`path.join`，涵盖了 os 最核心的功能。

---

## 10. 相关模块

学完 os 之后建议了解的相关模块：

| 模块 | 用途 |
|------|------|
| `os.path` | 路径拼接、解析、判断（os 的子模块） |
| `shutil` | 高级文件操作：复制文件、复制整个文件夹、删除非空文件夹 |
| `pathlib` | Python 3.4+ 推出的更现代的路径操作方式 |
| `glob` | 用通配符匹配文件（如 `*.txt`） |
| `fileinput` | 批量逐行读取多个文件 |
| `tempfile` | 创建临时文件或路径 |
| `platform` | 获取更详细的系统信息 |

### shutil 常用功能

```python
import shutil

shutil.copy('a.txt', 'backup/a.txt')       # 复制文件
shutil.copytree('文件夹A', '文件夹A备份')     # 复制整个文件夹
shutil.rmtree('非空文件夹')                  # 删除非空文件夹
shutil.move('a.txt', '新位置/a.txt')        # 移动文件
```

---

## 11. 可移植性注意事项

- os 模块大部分功能是**跨平台**的（如 `os.path.join`、`os.mkdir`、`os.listdir`）
- 但有些功能是特定系统独有的（如 `os.fork()` 只能在 Linux/Mac 上用）
- 一旦使用了特定系统的功能，代码的可移植性就会受损

---

## 12. 杂项提示

### Python 3 支持中文变量名

```python
名字 = "小明"
年龄 = 18
print(名字, 年龄)
```

这是合法的！Python 3 从 2008 年发布起就支持 Unicode 变量名。但实际开发中建议用英文命名。

### 文件命名注意

不要把自己的文件命名为 `os.py`，否则 `import os` 时 Python 会导入你自己的文件而不是系统模块。

---

> 参考来源：知乎 - 轩辕御龙《Python os 模块详解》  
> 笔记整理日期：2026 年 4 月
