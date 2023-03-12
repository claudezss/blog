---
title: 'Call Python Function by Dynamical Name'
date: '2023-03-12T13:06:38+08:00'
draft: false
description: 'python function'
author: 'Yan'
tags: ["python"]
theme: "dark"
---

# Sample


```python
def func1():
print(2)
def func2():
print(2):
def func3():    
print(3)

func_list = ["func1", "func2", "func3"]

for func_name in func_list:    # execute function by function name    
	globals()[func_name]()
# output:
# 1
# 2
# 3
```

