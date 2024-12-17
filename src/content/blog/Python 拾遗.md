---
title: Python拾遗系列(1)：脚本调用的玄机
description: 这个系列记录针对我个人学的不细致的地方，以前很少项目化编程，在真正编写一些系统性的项目时，会遇到很多之前没注意过的地方。在LLM的帮助下，逐个解决这些细节但很重要的问题，并把它记录在此
pubDatetime: 2024-12-17
tags: 
share: "true"
featured: false
draft: false
---
### Python脚本调用那点事

> 我编写了一个小项目，需要在开发过程中进行一些简单的调试，于是就写了个`test.py`，但是在调用它的时候，遇到了`import`找不到模块的问题。

首先交代一下项目的目录结构：

```python
teaching-with-llm/
├── src/
│   ├── __init__.py
│   └── config.py
└── test/
    ├── __init__.py
    └── test.py
```

其中，我们要运行一下`./test/test.py`这个文件，它用来测试部分项目代码是否正确，里面只有三行：

```python
from src.config import LLMConfig, OllamaConfig, OpenAIConfig

config = LLMConfig()
print(config.openai.api_key)
print(config.ollama.host)
```

然后，采用了如下两个方法运行，对比下面两个方法：

#### 方式1

```bash
(base) softmaple@SoftdeMacBook-Pro % python ./test/test.py
```

如果这样运行，那么会出现下面的错误：

```bash
Traceback (most recent call last):
  File "/Users/softmaple/Documents/teaching-with-llm/./test/test.py", line 1, in <module>
    from src.config import LLMConfig, OllamaConfig, OpenAIConfig
ModuleNotFoundError: No module named 'src'
```

问题在于，程序找不到`src`文件夹。但是确实是在项目根目录运行的`test.py`，为什么会找不到呢？

> **因为Python在运行脚本的时候，会把脚本所在目录`test/`添加到`sys.path`中 -> 并没有添加项目根目录 -> 系统找不到`src`，当然也就找不到下面的 `src/config`**

明白了这点，要想使用这个方法也很简单，就干脆把项目根目录也加入就行了！因此在`test.py`文件中，加入：

```python
import sys
from pathlib import Path

sys.path.append(str(Path(__file__).parent.parent))
```

这样就将项目根目录添加进了系统变量中了，再运行就能找到了。注意：

- 这样添加的变量，只在本段Python程序进程中有效，关闭了也就没了，不会真正添加到系统环境变量中！
- 对于上述语句的理解，`Path(__file__).parent.parent`，这个`__file__`指的是本语句所在的文件，第一层`parent`回到文件所在目录，下一个回到上级目录，也就是项目根目录，注意比常规我理解多一层目录！

#### 方式2

```bash
(base) softmaple@SoftdeMacBook-Pro % python -m test.test
```

这是告诉Python将脚本当作模块的来执行。运行时，会将当前工作目录添加到`sys.path`中，因此都可以找到！
注意：如果是这个方式运行，那么要注意在所有的文件夹下，都添加一个`__init__.py`，让Python把文件看作模块。