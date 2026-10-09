---
title: "Hello World"
date: 2026-01-01
draft: false
tags: ["随笔"]
categories: ["随笔"]
---

欢迎来到我的笔记站！这是第一篇文章，用来验证站点是否正常构建和部署。

## 这篇站点是怎么搭的

- **框架**：[Hugo](https://gohugo.io/)（静态站点生成器）
- **主题**：[PaperMod](https://github.com/adityatelange/hugo-PaperMod)（以 git submodule 方式引入）
- **托管**：GitHub Pages，由 GitHub Actions 自动构建部署

## 代码高亮效果

```python
def greet(name: str) -> str:
    """打个招呼"""
    return f"Hello, {name}!"


if __name__ == "__main__":
    print(greet("zedward"))
```

## 下一步

把 `content/posts/` 里的这个示例文章换成你自己的内容就行。
