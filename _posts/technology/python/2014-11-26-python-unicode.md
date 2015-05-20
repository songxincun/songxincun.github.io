---
layout: post
title:  "Python 字符编码问题"
date:   2014-11-26 14:55:23
categories: python
---

在对字符串进行操作的时候提示：
> UnicodeDecodeError: 'ascii' codec can't decode byte 0xe4 in position 0: ordinal not in range(128)

对于这种字符编码的问题有个一劳永逸的方法，就是在开头加上：

```python
import sys

reload(sys)
sys.setdefaultencoding('utf8')
```

[jekyll-gh]: https://github.com/jekyll/jekyll
[jekyll]:    http://jekyllrb.com

