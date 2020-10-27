```
// 这里根据移动端原生的 userAgent 来判断当前是 Android 还是 ios
const u = navigator.userAgent;
// Android终端
const isAndroid = u.indexOf('Android') > -1 || u.indexOf('Adr') > -1;
// IOS 终端
// const isIOS = !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/);

/**
 * 配合 IOS 使用时的初始化方法
 */
const iosFunction = callback => {
  if (window.WebViewJavascriptBridge) {
    return callback(window.WebViewJavascriptBridge);
  }
  if (window.WVJBCallbacks) {
    return window.WVJBCallbacks.push(callback);
  }
  window.WVJBCallbacks = [callback];
  var WVJBIframe = document.createElement('iframe');
  WVJBIframe.style.display = 'none';
  WVJBIframe.src = 'wvjbscheme://__BRIDGE_LOADED__';
  document.documentElement.appendChild(WVJBIframe);
  setTimeout(function() {
    document.documentElement.removeChild(WVJBIframe);
  }, 0);
};

/**
 * 配合 Android 使用时的初始化方法
 */
const androidFunction = callback => {
  if (window.WebViewJavascriptBridge) {
    callback(window.WebViewJavascriptBridge);
  } else {
    document.addEventListener(
      'WebViewJavascriptBridgeReady',
      function() {
        callback(window.WebViewJavascriptBridge);
      },
      false
    );
  }
};

window.setupWebViewJavascriptBridge = isAndroid ? androidFunction : iosFunction;

// isAndroid &&
//   window.setupWebViewJavascriptBridge(function(bridge) {
//     // 注册 H5 界面的默认接收函数（与安卓交互时，安卓端可以不调用函数名，直接 send 数据过来，就能够在这里接收到数据）
//     bridge.init(function(msg, responseCallback) {
//       console.log(msg);
//       responseCallback && responseCallback('JS 返回给原生的消息内容');
//     });
//   });

// // js调用原生注册的方法
// export function callHandler(name, data, callback) {
//   window.setupWebViewJavascriptBridge(bridge => {
//     bridge.callHandler(name, data, callback);
//   });
// }

// // APP调js方法 （参数分别为:js提供的方法名  回调）
// export function registerHandler(name, callback) {
//   window.setupWebViewJavascriptBridge(bridge => {
//     bridge.registerHandler(name, (data, responseCallback) => {
//       callback(data, responseCallback);
//     });
//   });
// }
```
