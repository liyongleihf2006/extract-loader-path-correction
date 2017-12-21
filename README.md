extract-loader-pathCorrection
----
对[extract-loader](https://www.npmjs.com/package/extract-loader)的一个修改,主要是校正引用文件路径不正确的问题

关于extract-loader的使用方式和api等请查看原项目[extract-loader](https://www.npmjs.com/package/extract-loader)

----


**为什么要进行修改?**

首先看下项目结构

现在项目的目录结构如下

```
  app
   | index.tpl.html
   | vender
        | css
           | index.css
           | image
               | test.png
```  
index.tpl.html 中的内容如下

```
  <html>
    <head>
        <link rel="stylesheet" href="./vender/index.css">
    </head>
    <body>   
    </body>
  </html>
```  
index.css 中的内容如下

```
  html{
    background: url(./image/test.png);
  }
```  
index.js 是空文件

webpack.config.js 中的内容如下

```
var path = require('path');
var HtmlWebpackPlugin = require("html-webpack-plugin");
module.exports={
    entry:{
       bundle:path.join(__dirname,"app/index.js")
    },
    output:{
        path:path.join(__dirname,"dist"),
        filename:"[name].js"
    },
    module:{
        rules:[{
            test:/\.png$/,
            use: [{
                loader:'file-loader',
                options: {
                    name: '[path][name].[ext]'
                }
            }],
            include:[path.join(__dirname, 'app')]  
        },{
            test:/\.html$/,
            use: [{
                loader:'html-loader',
                options: {
                    attrs: ['img:src', 'link:href','script:src','audio:src'],
                }
            }],
            include:[path.join(__dirname, 'app')]  
        },{
            test: /\.css$/,
            use: [
                {
                    loader: "file-loader",
                    options: {
                        name: "[path][name].[ext]"
                    }
                },
                {
                    loader: "extract-loader"
                },
                {
                    loader: 'css-loader'
                }
            ],
            include:[path.join(__dirname, '/app')]
        }]
    },
    plugins: [
        new HtmlWebpackPlugin({
            template:path.join(__dirname,"app/index.tpl.html")
        })
    ]
 }
```  

输出到 dist 中目录结构如下

```
app
  | index.css
  | vender
      | image
          | test.png
bundle.js
index.html
```  
index.html 中的内容如下

```
<html>
    <head>
        <link rel="stylesheet" href="app/vender/index.css">
    </head>
    <body>   
    <script type="text/javascript" src="bundle.js"></script></body>
</html>
```  


index.css 中的内容如下

```
html{
    background: url(app/vender/image/test.png);
}
```  
上面向目标文件夹(dist)输出的内容中index.css引用图片test.png的路径 app/vender/image/test.png 是有问题的,这时候我们希望test.png的引用路径应该是./image/test.png，这才是正确的.

因此对[extract-loader](https://www.npmjs.com/package/extract-loader)的进行了修改.

将 webpack.config.js 中的 loader: "extract-loader" 改为 loader: "extract-loader-path-correction"

```
{
  loader: "extract-loader-path-correction"
},
```  
   
再次进行打包后项目结构没有发生变化,但index.css中的内容变成了这样

```
html{
    background: url(./image/test.png);
}
```  
这正是我们所想要的!

----

该修改仅仅是使得引用路径进行修正,对[extract-loader](https://www.npmjs.com/package/extract-loader)的其他功能以及配置等没有任何影响.

上面的解释中只是使用了图片作为示例,实际上可以修正引入的任何资源的路径,比如字体文件,svg文件等等等等...

当然想要正确引入各种资源文件还需要使用各种loader才行,其他的疑惑和使用方式请前往[extract-loader](https://www.npmjs.com/package/extract-loader)查看