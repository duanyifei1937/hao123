# 团队 BookMarks

## Install

依赖 `^@node8.0`

```js
npm install
```

## Build

```js
npm run build
```

## Output

`/dist`

## Edit

*   `~ cd /bookmarks`
*   `vi XXXX.json`, xxxx 为目录集

```json
{
    "name": "显示的名字",
    "url": "跳转的URL",
    "icon": "图标" // http://图标url || fa-icon@https://fontawesome.com/icons || ''
}
```

## DEMO

toolbox.testabc.net
![xx](https://raw.githubusercontent.com/duanyifei1937/Picture-bed/master/blog-img/20190721135355.png)


## Deploy
在本地`npm run build`, 生成static file in dist/, 将dist/下内容copy至Nginx访问路径下；
* 使用helm部署；
* 使用Jenkins流程化；
