web-camera
==========

网页截图工具 (by phantomjs)。通过phantomjs来打开渲染网页，对网页进行截图。   

## Usage  

```js
var Camera = require('webcamera');
var fs = require('fs');

/**
 * web camera by Node.js and Phantomjs
 * @param {Object} options 
 *   - path          {String}   default picture dir path
 *   - workerNum     {Number}   child_process max num
 *   - timeout       {Number}   child_process timeout.
 *   - phantom       {String}   phantomjs path
 *   - phantomScript {String}   phantomjs script path, use input arguments as default script
 *   - tfsClient     {Object}   tfs client instance
 *   - tfsOpts       {Object}   tfs options. if do not have tfsClient and tfsOpts, shotTFS become invalid
 */
var camera = Camera.create({
  tfsOpts: {    
    appkey: 'tfscom',
    rootServer: 'restful-store.daily.tbsite.net:3800',
    imageServers: [
      'img01.daily.taobaocdn.net',
      'img02.daily.taobaocdn.net',
      'img03.daily.taobaocdn.net',
      'img04.daily.taobaocdn.net'
    ]    
  }
});

//当处理速度比调用速度低时会触发此事件
camera.on('overload', function (listLength) {
  //listLength为排队等待处理的长度
});

//截图保存到本地
camera.shot('http://www.baidu.com', './baidu.png', function (err, data) {
  //data.should.equal('./baidu.png');
});

//截图作为stream
camera.shotStream('http://www.baidu.com', function (err, s) {
  var datas = [];
  var filePath = './test.jpg';
  var file = fs.createWriteStream(filePath, {encoding: 'binary'});
  s.on('data', function (data) {
    file.write(data.toString('binary'), 'binary');          
  });
  s.on('end', function () {
    console.log('get pictrue ok');
  });
});

//截图上传TFS,指定partition分区（会被 partition % 1000 + 1处理）， 文件名。详情查看https://github.com/fengmk2/tfs#api
camera.shotTFS('http://www.baidu.com',320, 'baidu.png', function (err, data) {
  /*
  data.should.like:
  {name: 'L1/1/320/baidu.png', size: 36889, url: 'img04.daily.taobaocdn.net/L1/1/320/baidu.png'}
  */
});
```

所有的调用都可以在`callback`之前传入参数`options`. 

```js
camera.shotTFS('http://www.baidu.com',320, 'baidu.png', {
  clipRect: {
    top: 0,
    left:0,
    height: 'all',
    width: 'all'
  }  
}, function (err, data) {
  /*
  data.should.like:
  {name: 'L1/1/320/baidu.png', size: 36889, url: 'img04.daily.taobaocdn.net/L1/1/320/baidu.png'}
  */
});
```

|名字|类型|含义|
|----|----|----|
|clipRect|Object|指定截图的矩形区域。有四个属性:top(0), left(0), height(window), width(window)。height和width可以设置为window或者all,window将会截取当前一屏，all会截取网页全部大小|
|viewportSize|Object|设置网页的分辨率，有两个属性:width(1024), height(768)。|
|renderDelay|Number|网页加载完成之后延迟多少毫秒之后截图，默认为0|
|picPath|String|设置图片保存位置，只在`shot`方法时生效，等效于shot方法的第二个参数|
|mimeType|String|设置截图的保存类型（只有在没设置图片保存路径的情况下生效，否则使用图片保存路径的后缀类型），支持png, jpeg, gif.默认为png|
|script|Function|网页加载完成之后可以在网页中执行这个方法。|

## Install  
`npm install webcamera`

## Dependences  
[`phantomjs`](http://phantomjs.org/) >= v1.9 
[`TFS`](http://github.com/fengmk2/tfs) >= v0.1.2

## Notice  
[淘宝CentOS使用](https://github.com/dead-horse/web-camera/blob/master/taobao.md)   

## Debug

```bash
$ tail -f /tmp/phantom_shot.log &
$ phantomjs "phantom/web_camera_phantom.js" "https://github.com/" > github.png
```

github page: [github.png](http://nfs.nodeblog.org/b/0/b06ed6be50682731bfae32d79b25894b.png)

## Licences  
(The MIT License)

Copyright (c) 2013 dead-horse and other contributors

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
