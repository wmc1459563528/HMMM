# hmmm-complete
我的两个功能（标签管理，题组管理）完成了。 ！！！我的功能做完了（精选题库）！！！！

## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

### Lints and fixes files
```
npm run lint
```

### Customize configuration
See [Configuration Reference](https://cli.vuejs.org/config/).



我叫张三，我的工作完成了。




### remove git hook
```json
,
  "gitHooks": {
    "pre-commit": "lint-staged"
  },
  "lint-staged": {
    "*.{js,jsx,vue}": [
      "vue-cli-service lint",
      "git add"
    ]
  }
```
