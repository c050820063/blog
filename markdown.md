[hexo官网](https://hexo.io/)
[blog主页](https://c050820063.github.io./)

## 基本操作

- 创建文章
```
  $ hexo new "My New Post"
```

- 生成文件
```
  $ hexo generate
```

- 生成本地服务器
1. 安装hexo-server
```
  npm install hexo-server --save
```
2. 启动服务器
```
  $ hexo server -p 5000
```

- 一键部署
```
  $ hexo generate
  $ hexo deploy
```

## bug记录
Q：hexo g生成的文件为空
A：node版本过高、降版本