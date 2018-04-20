<p align="center">
  <img width="320" src="https://vuejs.org/images/logo.png">
</p>

<p align="center">
  <a href="https://github.com/vuejs/vue">
    <img src="https://img.shields.io/badge/vue-2.5.10-brightgreen.svg" alt="vue">
  </a>
  <a href="https://github.com/ElemeFE/element">
    <img src="https://img.shields.io/badge/element--ui-2.3.0-brightgreen.svg" alt="element-ui">
  </a>
  <a href="https://travis-ci.org/PanJiaChen/vue-element-admin" rel="nofollow">
    <img src="https://travis-ci.org/PanJiaChen/vue-element-admin.svg?branch=master" alt="Build Status">
  </a>
  <a href="https://github.com/PanJiaChen/vue-element-admin/blob/master/LICENSE">
    <img src="https://img.shields.io/github/license/mashape/apistatus.svg" alt="license">
  </a>
  <a href="https://github.com/PanJiaChen/vue-element-admin/releases">
    <img src="https://img.shields.io/github/release/PanJiaChen/vue-element-admin.svg" alt="GitHub release">
  </a>
</p>

English | [简体中文](./README.zh-CN.md)


**Note: This project uses element-ui@2.3.0+ version, so the minimum compatible vue@2.5.0+**

## Preparation

You need to install [node](http://nodejs.org/) and [git](https://git-scm.com/) locally. The project is based on [ES2015+](http://es6.ruanyifeng.com/)、[vue](https://cn.vuejs.org/index.html)、[vuex](https://vuex.vuejs.org/zh-cn/)、[vue-router](https://router.vuejs.org/zh-cn/) 和 [element-ui](https://github.com/ElemeFE/element). All data requests for this project are simulated using [Mock.js](https://github.com/nuysoft/Mock). It would be helpful if you have pre-existing knowledge on those.

 **This project is not a scaffolding and is more of an integrated solution.**

 **This project does not support low version browsers (e.g. IE). Please add polyfill yourself if you need them.**

 <p align="center">
  <img width="900" src="https://github.com/gaoxing1017/Jira-Admin/raw/dev/data-statics.png">
</p>

## Features
```
- Login / Logout
- Multi-environment build
- Dynamic sidebar (supports multi-level routing)
- Dynamic breadcrumb
- I18n
- Customizable theme
- Jira DATA Statics
```

## Getting started

```bash
# clone the project
git clone https://github.com/gaoxing1017/Jira-Admin.git

# install dependency
npm install

# develop
npm run dev
```

This will automatically open http://localhost:9527.

## Build
```bash
# build for test environment
npm run build:sit

# build for production environment
npm run build:prod
```

## Advanced
```bash
# --report to build with bundle size analytics
npm run build:prod --report

# --preview to start a server in local to preview
npm run build:prod --preview

# lint code
npm run lint

# auto fix
npm run lint -- --fix
```

## License

[MIT](https://github.com/gaoxing1017/Jira-Admin/blob/master/LICENSE)

Copyright (c) 2017-present GX
