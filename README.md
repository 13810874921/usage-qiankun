# 乾坤微服务配置 #
## vue(主) + vue（子） 


## 主应用 ##
    考虑成sidebar/menu，来跳转子应用

### 内置路由
- 无需在router/index.js下面配置子应用路由
- 只需注册子应用
    ```js
    registerMicroApps()
    ```




### 外接项目
    ```js
    //与放置的位置无关，无需写registerMicroApps()，由官网demo可见
    loadMicroApp({
        name: 'vue',
        entry: '//127.0.0.1:8088',
        container: '#vue',
        }, {
        sandbox: {
            strictStyleIsolation: false //是否启用沙盒，false时以普通html载入
        }
    });
    ```
    container的名字需要在 App.vue里面写<div id="vue"></div>


### 内置路由 vs 外接项目 区别
    外接容器直接加载，而内置路由需要通过跳转加载

### 路由设置 
    history

### 页面跳转方式
    同主应用所用框架


## 子应用 ##
### 基本前提
1. 设置跨域请求，让主应用可以访问
2. src 菜单下增加publick-path.js  [非必须]




### 项目配置
dev环境下：config/index.js(或vue.config.js)指定端口 ，以便主应用可以查询到
### webpack配置
- devServer  增加跨域
    ``` js
    headers: {
      'Access-Control-Allow-Origin': '*',
    }
    ```
- *重点* webpack.base.conf.js
    1. output必须增加
    ```js
        const packageName = require('../package').name;
        ...
        output:{
            library: `${packageName}-[name]`,
            libraryTarget: 'umd',
            jsonpFunction: `webpackJsonp_${packageName}`,
        }
    ```
    否则报错：Error: [qiankun] You need to export lifecycle functions in vue entry at getLifecyclesFromExports。
### main.js配置（项目入口文件）
- render
    ```js
    let router=null;
    let instance = null;
    function render(props = {}) {
        const { container } = props;
        router = new VueRouter({
            base: window.__POWERED_BY_QIANKUN__ ? '/vue' : '/',
            mode: 'history',
            routes,
        });
        console.log(container,'????????????................',window.__POWERED_BY_QIANKUN__)
        instance = new Vue({
            router,
            // store,
            render: h => h(App),
        }).$mount(container ? container.querySelector('#app') : '#app');
    }

    if (!window.__POWERED_BY_QIANKUN__) {
        render();
    }
    ```
- store问题，[非必须]，维持项目原有

- 加入qiankun生命周期
    ```js
    export async function bootstrap() {
        console.log('[vue] vue app bootstraped');
    }

    export async function mount(props) {
        console.log('[vue] props from main framework', props);
        // storeTest(props);
        render(props);
    }

    export async function unmount() {
        instance.$destroy();
        instance = null;
        router = null;
    }
    ```
