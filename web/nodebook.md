
### 在js中使用console.log时eslint warning:Warning - Unexpected console statement.添加如下配置，便可解决。
```
rules: {
    'no-console': 'off',
},
```