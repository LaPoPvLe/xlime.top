---
title: Egg JS
date: 2017-06-23 12:14:00
tags:
---

# Egg JS 从入门到放弃
---
[TOC]

## 安装
```shell
# 全局安装 egg-init 命令
$ npm i -g egg-init
```

## 环境与配置
### 设置运行环境
1. 通过 `config/env` 文件指定，该文件的内容就是运行环境，如：`prod`：
    *一般使用构建工具来生成这个文件*。
2. 通过 `EGG_SERVER_ENV` 环境变量指定运行环境。
    *更加方便实用，一般在生产环境启用*。

### 获取运行环境
访问变量 `app.config.env` 获取当前应用的运行环境。

> **与 NODE_ENV 的区别**： 之所以使用 `EGG_SERVER_ENV` 是为了使运行环境区分度更高更精准。参考一般项目的开发流程包括：本地开发环境、测试环境和生产环境等，除了本地开发环境和测试环境外，其他的环境可以统称为**服务器环境**。所以服务器环境的 `NODE_ENV` 应该为 `production`。

架构默认支持的运行环境及映射关系
*如果未指定 `EGG_SERVER_ENV` 会根据 `NODE_ENV` 来匹配*

| 指定的 **NODE_ENV** | 默认生成的 **EGG_SERVER_ENV** | 说明     |
| ---------------- | ------------------------ | ------ |
|                  | local                    | 开发本地环境 |
| test             | unittest                 | 单元测试   |
| production       | prod                     | 生产环境   |

> 为了避免混淆，导致出现一些不可预期的错误，建议每个环境都制定 `NODE_ENV` 和 `EGG_SERVER_ENV`。

### 自定义环境
当预定义的环境变量不满足开发需求时，可以自定义环境。
比如自定义**集成测试环境** `sit`:
`NODE_ENV=production` `app.config.env=sit`
启动时会加载
```shell
> config.sit.js
> config.default.js
```

### 与 Koa 的区别

| 框架 | 环境变量字段 | 来源 |
| --- | --- | --- |
| Koa| `app.env` | `process.env.NODE_ENV` |
| EggJS | `app.config.env` | `process.env.EGG_SERVER_ENV` 或 根据 `NODE_ENV` 生成默认值| 

## Config 配置

配置可以自动合并应用、插件、框架的配置，按顺序覆盖，并且可以根据环境维护不同的配置。合并后的配置可从 `app.config` 属性获取。

### 配置的管理

常见的配置管理方案：

| 方案                                       | 优点        | 缺点                           |
| ---------------------------------------- | --------- | ---------------------------- |
| *使用平台管理配置，应用构建时将当前环境的配置放入包内，启动时指定*       | 便捷，不依赖环境  | 无法依次构建多次部署；本地开发如需更改配置十分麻烦。   |
| *使用平台管理配置文件，启动时将当前环境的配置通过环境变量传入*         | 比较优雅      | 对运维要求高且需要部署平台支持；开发环境也面临同样问题。 |
| *使用代码管理配置，在代码中添加多个环境的配置，启动时传入当前环境变量的参数即可* | 简单，无需额外工作 | 无法全局配置；必须修改代码。               |

EggJS 采用的是最后一种方案，**配置即代码**，配置的变更也应该经过 review 后才能发布。应用包本身是可以部署在多个环境的，而只需要制定运行环境即可。

### 多环境支持

不同环境的配置会根据运行环境自行加载并合并，多环境配置文件在 `config` 目录下：

```txt
config
|- config.default.js
|- config.test.js
|- config.unittest.js
`- config.local.js
```

- `config.default.js`：默认配置文件，所有环境都会加载，一般会作为开发环境的默认配置文件。
- `config.<env>.js`：根据 `EGG_SERVER_ENV` 值加载对应的配置文件。如果环境为 `prod` 则加载 `config.prod.js` 并且会覆盖 `config.default.js` 的配置项。

### 设置配置

配置支持**方法**或**对象**两种类型。

- *方法*：配置文件接受 `appInfo` 参数。

  ```javascript
  // 返回方法，其返回值为配置对象
  const path = require('path');
  module.exports = appInfo => {
    return {
      logger: {
        dir: path.join(appInfo.baseDir, 'logs')
      }
    }
  }
  ```

  内置的 **appInfo** 属性
  

  | appInfo | 说明                                       |
  | ------- | ---------------------------------------- |
  | pkg     | package.json。                            |
  | name    | 应用名，等于 pkg.name。                         |
  | baseDir | 应用代码的目录。                                 |
  | HOME    | 用户目录。                                    |
  | root    | 应用的根目录，尽在 local 和 testunit 环境下设置，其他都为 HOME。 |
  > `appInfo.root` 是为了适配不同环境下的开发要求。如生产环境的路径是 `/home/<user>/logs` 作为日志目录，开发环境可以指定其他目录避免污染当前用户 home 目录。 
  
### 配置的加载顺序

  ​