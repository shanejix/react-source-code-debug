> https://www.shanejix.com/posts/%E6%90%AD%E5%BB%BA%20React%2017%20%E6%BA%90%E7%A0%81%E6%9C%AC%E5%9C%B0%E8%B0%83%E8%AF%95%E7%8E%AF%E5%A2%83/

1. 使用 create-react-app 脚手架创建项目

```shell
npx create-react-app react-source-code-debug
```

2. 弹射 create-react-app 脚手架内部配置

```shell
yarn run eject
```

3. 克隆 react 官方源码 (在项目的根目录下进行克隆)

```shell
git clone --branch v17.0.2 --depth=1 https://github.com/facebook/react.git src/react
```

4. 通过 alias ，链接本地源码

```javascript
// 文件位置: react-source-code-debug/config/webpack.config.js

resolve: {
  // ...
  alias: {
    // Support React Native Web
    // https://www.smashingmagazine.com/2016/08/a-glimpse-into-the-future-with-react-native-for-web/
    'react-native': 'react-native-web',
    // Allows for better profiling with ReactDevTools
    ...(isEnvProductionProfile && {
      'react-dom$': 'react-dom/profiling',
      'scheduler/tracing': 'scheduler/tracing-profiling',
    }),
    ...(modules.webpackAliases || {}),
    + // FIXME: REACT SOURCE CODE DEBUG
    + 'react': path.resolve(__dirname, '../src/react/packages/react'),
    + 'react-dom': path.resolve(__dirname, '../src/react/packages/react-dom'),
    + 'shared': path.resolve(__dirname, '../src/react/packages/shared'),
    + 'react-reconciler': path.resolve(__dirname, '../src/react/packages/react-reconciler'),
    + // 'scheduler': path.resolve(__dirname, '../src/react/packages/scheduler'),
  },
}
```

5. 修改环境变量

```javascript
// 文件位置: react-source-code-debug/config/env.js

// FIXME: REACT SOURCE CODE DEBUG
const stringified = {
  + __DEV__: true,
  + __PROFILE__: true,
  + __UMD__: true,
  + __EXPERIMENTAL__: true,
  'process.env': Object.keys(raw).reduce((env, key) => {
    env[key] = JSON.stringify(raw[key]);
    return env;
  }, {}),
};
```

6. 在根目录创建 `eslintrc.json`文件，内容如下

```json
{
  "extends": "react-app",
  "globals": {
    "__DEV__": true,
    "__PROFILE__": true,
    "__UMD__": true,
    "__EXPERIMENTAL__": true
  }
}
```

7. 修改 ReactFiberHostConfig.js 文件

```javascript
// 文件位置: /react/packages/react-reconciler/src/ReactFiberHostConfig.js

- import invariant from 'shared/invariant';
- invariant(false, 'This module must be shimmed by a specific renderer.');

+ // FIXME: REACT SOURCE CODE DEBUG
+ export * from './forks/ReactFiberHostConfig.dom'
```

8. 修改 ReactSharedInternals.js 文件

```javascript
// 文件位置: /react/packages/shared/ReactSharedInternals.js

- import * as React from 'react';

- const ReactSharedInternals =
-   React.__SECRET_INTERNALS_DO_NOT_USE_OR_YOU_WILL_BE_FIRED;

+ // FIXME: REACT SOURCE CODE DEBUG
+ import ReactSharedInternals from '../react/src/ReactSharedInternals';

+ export default ReactSharedInternals;
```

9. 修改 invariant.js 文件

```javascript
// 文件位置: /react/packages/shared/invariant.js

export default function invariant(condition, format, a, b, c, d, e, f) {

  + // FIXME: REACT SOURCE CODE DEBUG
  + if (condition) {
  +   return;
  + }

  throw new Error(
    'Internal React error: invariant() is meant to be replaced at compile ' +
    'time. There is no runtime version.',
  );
}
```

10. 关闭 eslint 扩展

```javascript
// 文件位置: react/.eslingrc.js [module.exports]

- extends: [
-  'fbjs',
-  'prettier'
- ]
```

11. 解决 eslint 报错

```javascript
// 将 webpack 中 eslint 插件给关掉,修改 src/config/webpack.config.js 文件

// ...

// FIXME: REACT SOURCE CODE DEBUG

// !disableESLintPlugin &&
// new ESLintPlugin({
//   // Plugin options
//   extensions: ['js', 'mjs', 'jsx', 'ts', 'tsx'],
//   formatter: require.resolve('react-dev-utils/eslintFormatter'),
//   eslintPath: require.resolve('eslint'),
//   failOnError: !(isEnvDevelopment && emitErrorsAsWarnings),
//   context: paths.appSrc,
//   cache: true,
//   cacheLocation: path.resolve(
//     paths.appNodeModules,
//     '.cache/.eslintcache'
//   ),
//   // ESLint class options
//   cwd: paths.appPath,
//   resolvePluginsRelativeTo: __dirname,
//   baseConfig: {
//     extends: [require.resolve('eslint-config-react-app/base')],
//     rules: {
//       ...(!hasJsxRuntime && {
//         'react/react-in-jsx-scope': 'error',
//       }),
//     },
//   },
// }),
```

12. 告诉 babel 在转换代码时忽略类型检查

```shell
yarn add @babel/plugin-transform-flow-strip-types -D
```

```javascript
// 文件位置: react-source-code-debug/config/webpack.config.js [babel-loader]

plugins: [
  // ...

  // FIXME: REACT SOURCE CODE DEBUG
  [require.resolve("@babel/plugin-transform-flow-strip-types")],
];
```

13. 新增 eslint 配置

```json
// 在 react 源码文件夹中新建 .eslintrc.json 并添加如下配置

{
  "extends": "react-app",
  "globals": {
    "__DEV__": true,
    "__PROFILE__": true,
    "__UMD__": true,
    "__EXPERIMENTAL__": true
  }
}
```

14. 修改 `react` `react-dom` 引入方式

```javascript
- // import React from 'react';
- // import ReactDOM from 'react-dom';

+ // FIXME: REACT SOURCE CODE DEBUG

+ import * as React from 'react';
+ import * as ReactDOM from 'react-dom';
```

15. 解决 VSCode 中 flow 报错

```javascript
// 新建 .vscode/setting.js

{
  "typescript.validate.enable": false,
  "javascript.validate.enable": false
}
```

16. 解决 **DEV** 报错

```shell
// 删除 node_modules 文件夹，执行 npm install / yarn

// 根目录下

rm -fr node_modules && yarn
```
