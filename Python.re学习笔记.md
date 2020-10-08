`pattern=re.compile(regex)`
编译正则表达式生成pattern

`match=pattern.match(str)`
返回首次匹配信息，要求从str[0]开始就要匹配regex。不匹配返回None

`match=pattern.search(str)`
返回首次匹配信息，从str任意处匹配都可以。

`match.group()`
返回str满足regex的子串

`match.group(index)`
index>0时返回regex中第几个括号里的内容

`match.groups()`
返回列表[match.group(1...)]

`pattern.findall(str)`
返回所有匹配的二维数组，x轴为第几次匹配，y轴是匹配内第几个括号

<u>re静态函数：</u>
> match=re.match(regex,str)
match=re.search(regex,str)
list=re.findall(regex,str)

需要转义的特殊字符：$()*+.[?\^{|


Demo程序[]().py：
```python
import re
test = 'c12345.py'
regex = '([0-9]{3,})\.py$'
pattern = re.compile(regex)
match = pattern.match(test)
if match:  # Not matches
    print(match.group())

match = pattern.search(test)
if match:
    print(match.group())
    print(match.group(1))
    print(match.groups())

match = pattern.findall('c1053.pyaugm300.py')
if match:
    print(match)

```