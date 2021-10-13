---

---

# webpack 总结及问题等

1 webpack-dev-server 兼容问题： 

```json
 "webpack": "^5.30.0",
 "webpack-cli": "^4.6.0",
 "webpack-dev-server": "^3.11.2",
```

使用npm start报错：

Error: Cannot find module 'webpack-cli/bin/config-yargs'

升级后webpack5 后，`webpack-dev-server`与 `webpack-cli`不能正常兼容，第一种粗暴的解决方案是直接降级，

第二种，npm script 脚本修改为 webpack serve

```json
"start": "webpack serve",
```

