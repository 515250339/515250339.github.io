### Vue脚手架安装

安装node.js.-->双击安装包-->![node安装](.\django相关截图\node安装.png)

-->一直下一步，直到安装 完成，默认已经添加到环境变量，不需要再重复添加



#### 切换cnpm

```node
# 进入cmd命令行，输入以下内容

npm install -g cnpm --registry=https://registry.npm.taobao.org
```

#### 安装vue-cli

```node
cnpm install -g vue-cli  # 全局安装
```

#### vue项目创建

```
# vue-init webpack 项目名
vue-init webpack mdvue
```

![Vue项目创建](.\django相关截图\Vue项目创建.png)

#### 进入项目

```node
cd mdvue   # 进入项目目录
cnpm install  # 安装依赖

```

![vue依赖包安装](.\django相关截图\vue依赖包安装.png)

#### 启动项目

```node
npm run dev  # 启动项目
```

