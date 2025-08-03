# 初始化项目
> 未指定 vite 版本。当前版本 `"vite": "^5.4.10"`
> 下载依赖要求`nodejs` ^18.8以上
* 命令执行
```shell
# 在项目目录 执行  . 表示当前目录，不在创建文件夹。当前文件夹名为项目名。
yarn create vite . --template react

# 安装依赖
yarn

# 运行模板，当前已经是一个可执行的react简易模板
yarn dev
```

* 目录结构（自上而下）
  * node_modules : 执行`yarn install` 命令后下载的依赖包存放目录。
  * public : index.html作为项目入口，vite 并没有放在public目录下，而是放在项目的根目录下；若想要移动index.html文件到public目录下需要同时移动vite.config.js文件到同一目录，并在运行命令后指定文件目录。例`vite public`
  * src : 项目文件存放根目录
    * assets : 静态资源文件

* 关键配置文件
  * package.json : 项目核心配置管理文件(项目依赖、运行脚本、插件配置...)
    * scripts : 定义自动执行任务命令。打包、项目本地运行...
    * dependencies : 生产环境依赖
    * devDependencies : 开发环境依赖
    * lint-staged : 指定在 git 提交中对暂存文件运行的任务。 与 Husky 等工具配合使用，可执行提交前质量检查。
    * browserslist : 指定浏览器版本，便于如`babel`、`PostCss`生成兼容浏览器版本的代码。



* 目录结构
```shell
src/
├── components/       # 可复用的 UI 组件
│   ├── Header.js
│   ├── Footer.js
│   └── ExampleCard.js
├── pages/            # 页面级组件
│   ├── Home.js
│   ├── About.js
├── themes/           # 自定义主题
│   └── theme.js
├── App.js            # 应用入口
└── index.js          # React DOM 渲染入口
```

* 安装Material Ui
```javaScript
// 核心包
yarn add @mui/material @emotion/react @emotion/styled

// 依赖react版本
"peerDependencies": {
  "react": "^17.0.0 || ^18.0.0",
  "react-dom": "^17.0.0 || ^18.0.0"
},

// 配置Mui 主题 TODO

```

* 引入图标、字体 两种方式
```javaScript
// 通过在index.html入口文件的<head>标签内引入链接
// google 链接
<link
  href="https://fonts.googleapis.com/css2?family=Roboto:wght@300;400;500;700&display=swap"
  rel="stylesheet"
/>


// 国内链接
  <link href="https://fonts.font.im/css?family=Roboto:300,400,500,700,900" rel="stylesheet" />
  <link href="https://fonts.font.im/css?family=Material+Icons" rel="stylesheet" />

// 使用安装依赖包的方式，将需要的资源下载到本地。
yarn add @mui/icons-material

yarn add @fontsource/roboto
// 在app.jsx入口处引入
import '@fontsource/roboto/300.css';
import '@fontsource/roboto/400.css';
import '@fontsource/roboto/500.css';
import '@fontsource/roboto/700.css';


```



# 知识点 --待整合
* 关于npm install --save-dev 和 --save 的区别
  * --save-dev: 在最终打包的时候不会构建到源码当中去，通常用于`babel`、`webpack`这类用于项目构建、编译的工具依赖库。
  * --save: 代码运行必要的依赖库。