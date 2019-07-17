### 问题：
**gitbook**不支持**markdown**的`[TOC]`命令自动生成目录。
### 解决方案：
使用**gitbook**插件[gitbook-plugin-toc](https://www.npmjs.com/package/gitbook-plugin-toc)。
在`book.json`中添加
```
{
    "plugins": ["toc"]
}
```
在`book.json`所在目录运行`gitbook install`，安装完成后，在使用`[TOC]`命令的地方使用`<!-- toc -->`代替。即可自动生成文档目录。