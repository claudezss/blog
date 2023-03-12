---
title: 'Access OneDrive via Microsoft Graph API'
date: '2022-10-27T13:06:38+08:00'
draft: false
description: 'some description of the article'
author: 'author of this article'
tags: ["python", "microsoft graph api"]
theme: "dark"
---

# Preparation 

1. Create Microsoft account and subscribe OneDrive service
2. Create Azure account


```python
url="https://login.microsoftonline.com/consumers/oauth2/v2.0/authorize?client_id=xxxd&response_type=token&redirect_uri=http://localhost:8000&scope=Files.Read Files.Read.All Files.ReadWrite Files.ReadWrite.All offline_access"
```

