# Umi UI 插件开发

## 示例

index.js

```js
// 普通的 umi 插件写法，新增 api.onUISocket 和 api.addUIPlugin 接口
export default api => {
  // 处理 socket 通讯数据
  api.onUISocket(({ action, send, log }) => {
    // 通过 action 处理
    // 处理完后 send 数据到客户端
    send({ type, payload });
    // 过程中的日志通过 log 打到客户端
    log(`Adding block Foo/Bar...`);
  });
  // 添加编辑态的插件
  api.addUIPlugin(require.resolve('./dist/client.umd'));
};
```

ui.js（通过 [father-build](https://github.com/umijs/father/tree/master/packages/father-build) 打包到 `dist/ui.umd.js`）

```js
// 这个文件打 umd 到 ./dist/client.umd.js，external react、react-dom 和 antd，用 father-build 很容易打出来
export default api => {
  const {
    // 调用服务端方法
    callRemote,
  } = api;

  function Blocks() {
    return <h1>Blocks</h1>;
  }
  // 添加 panel，类似 vscode 点击左边的 Icon 后切换 Panel
  api.addPanel({
    title: '区块管理',
    icon: 'home',
    path: '/blocks',
    component: Blocks,
  });
  // 添加尾部的 tab，可添加类似 vscode 的 Output、Terminal、Problems 等功能
  api.addFooterTab();
  // 更多功能...
};
```

## 服务端接口

可访问[  所有插件接口和属性](https://umijs.org/plugin/develop.html)，以下是几个相关的。

### `api.onUISocket(handler: Function)`

处理 socket 数据相关，比如：

```bash
api.onUISocket(({ type, payload }, { log, send, success, failure }) => {
  if (type === 'config/fetch') {
    send({ type: `${type}/success`, payload: getConfig() });
  }
});
```

注：

1. 按约定，如果客户端用 `api.callRemote` 调用服务端接口，处理完数据需 send 加 `/success` 或 `/failure` 后缀的数据表示成功和失败。

#### `send({ type, payload })`

向客户端发送消息。

#### `success(payload)`

`` send({ type: `${type}/success` }) `` 的快捷方式。

#### `failure(payload)`

`` send({ type: `${type}/failure` }) `` 的快捷方式。

#### `progress(payload)`

`` send({ type: `${type}/progress` }) `` 的快捷方式。

#### `log(level, message)`

在控制台和客户端同时打印日志。

示例：

```js
log('info', 'abc');
log('error', 'abc');
```

### `api.addUIPlugin(clientJSPath: String)`

注册 UI 插件，指向客户端文件。

```bash
api.addUIPlugin(require.resolve('./dist/ui'));
```

注：

1. 文件需是 umd 格式

## 客户端接口

### `api.callRemote({ type, payload, onProgress })`

调服务端接口，并等待 type 加上 `/success` 或 `/failure` 消息的返回。如果有进度的返回，可通过 `onProgress` 处理回调。

### `api.listenRemote({ type, onMessage })`

监听 socket 请求，有消息时通过 `onMessage` 处理回调。

### `api.send({ type, payload })`

发送消息到服务端。

### `api.addPanel({ title, icon, path, component })`

添加客户端 Panel。

### `api.addLocales()`

添加全局国际化信息。

比如：

```bash
api.addLocales({
  'zh-CN': {
    'org.sorrycc.react.name': '陈成',
  },
  'en-US': {
    'org.sorrycc.react.name': 'chencheng',
  },
});
```

### `api.getLocale()`

返回当前语言，`zh-CN`、`en-US` 等。

### `api.showLogPanel()`

显示日志 Panel。

### `api.hideLogPanel()`

隐藏日志 Panel。

### `api.TwoColumnPanel`

比如：

```js
const { TwoColumnPanel } = api;

function Configuration() {
  return (
    <TwoColumnPanel
      sections={[
        { title: '基本配置', description, icon, component: C1 },
        { title: 'umi-plugin-react 配置', description, icon, component: C2 },
      ]}
    />
  );
}

api.addPanel({
  component: Configuration,
});
```