### welCropper 微信小程序截图工具

<img src="https://raw.githubusercontent.com/tomfriwel/welCropper/master/documents/screenshot2.png" width=600 />
----
<img src="https://raw.githubusercontent.com/tomfriwel/welCropper/master/documents/result.gif" width=300 />

#### wepy 版本(别人根据我的这个封装的,可以去看看,我并没有测试过,大家有需求的可以去试试)

[callmesoul/wepy-corpper](https://github.com/callmesoul/wepy-corpper)

#### 文件目录结构，要在测试机上运行，工程目录选择文件夹`project`
```
./
├── documents
│   ├── hierarchy.png
│   ├── result.gif
│   └── screenshot.jpeg
├── project
│   ├── app.js
│   ├── app.json
│   ├── app.wxss
│   ├── pages
│   │   ├── index
│   │   │   ├── index.js
│   │   │   ├── index.json
│   │   │   ├── index.wxml
│   │   │   └── index.wxss
│   │   └── test
│   │       ├── test.js
│   │       ├── test.json
│   │       ├── test.wxml
│   │       └── test.wxss
│   ├── utils
│   │   └── util.js
│   └── welCropper
│       ├── welCropper.js
│       ├── welCropper.wxml
│       └── welCropper.wxss
└── readme.md
```

* 保证图片质量，也可以选择压缩图
* 支持图片旋转
* 自由选择截图框

#### 两种模式
通过`showCropper`的`mode`设定
* mode:'rectangle' 返回图片
* mode:'quadrangle' 并不返回图片，只返回在图片中的四个点，用于perspective correction（可以查找OpenCV相关资料）


#### Documents

<img src="https://raw.githubusercontent.com/tomfriwel/welCropper/master/documents/hierarchy.png" width=600 />

* 使用`movable-view`的原因是不用自己实现拖拽，直接使用官方提供的这个组件。

因为`cropper`的数据和事件是直接绑定到`Page`上的，所以数据和事件命名应该避免一下名字（之后会想办法避免这种情况）及其相关解释：

data中的名字：

* cropperData
* cropperMovableItems
* cropperChangableData

函数名：

* showCropper
* hideCropper
* originalChange
* cropImage
* loadImage
* clearCanvas
* drawImage
* drawOriginalImage
* drawLines
* setupMoveItem
* moveEvent
* endEvent

外部只用到`showCropper`和`hideCropper`

```
/**
    src:输入图片地址
    callback(res):点击“完成”按钮后毁掉函数，毁掉函数中会有截图地址
*/
showCropper({
    src,    //字符串, 图片path
    mode,   //字符串, "rectangle" 或 "quadrangle". quadrangle只会返回4个点的坐标. rectangle返回截图path
    sizeType,   //数组, ['original', 'compressed'], 默认original
    callback    //回调函数, callback(res): mode=rectangle, res=path; mode=quadrangle, res=[[x,y], [x,y], [x,y], [x,y]]
})

```

#### 使用

将`welCropper`复制到自己的工程当中（以`/pages/index/index`为例）

##### `wxml`引入并调用：
```
<!-- 引入组件 -->
<import src="/welCropper/welCropper" />

<!-- 调用组件 -->
<template is="welCropper" data="{{data:cropperData, cropperMovableItems:cropperMovableItems, cropperChangableData:cropperChangableData}}"></template>

<!-- 用于选择图片，传入cropper中 -->
<button bindtap='selectTap'>select image</button>
```

##### `wxss`引入：
```
@import "/welCropper/welCropper.wxss";
```

##### `js`引入和使用：
```
// 获取显示区域长宽
const device = wx.getSystemInfoSync()
const W = device.windowWidth
const H = device.windowHeight - 50

let cropper = require('../../welCropper/welCropper.js');

console.log(device)

Page({
    data: {
    },
    onLoad: function () {
        var that = this
        // 初始化组件数据和绑定事件
        cropper.init.apply(that, [W, H]);
    },
    selectTap() {
        var that = this

        wx.chooseImage({
            count: 1, // 默认9
            sizeType: ['original', 'compressed'], // 可以指定是原图还是压缩图，默认二者都有
            sourceType: ['album', 'camera'], // 可以指定来源是相册还是相机，默认二者都有
            success(res) {
                const tempFilePath = res.tempFilePaths[0]
                console.log(tempFilePath)

                // 将选取图片传入cropper，并显示cropper
                // mode=rectangle 返回图片path
                // mode=quadrangle 返回4个点的坐标，并不返回图片。这个模式需要配合后台使用，用于perspective correction
                let modes = ["rectangle", "quadrangle"]
                let mode = modes[0]   //rectangle, quadrangle
                that.showCropper({
                    src: tempFilePath,
                    mode: mode,
                    sizeType: ['original', 'compressed'],   //'original'(default) | 'compressed'
                    callback: (res) => {
                        if (mode == 'rectangle') {
                            console.log("crop callback:" + res)
                            wx.previewImage({
                                current: '',
                                urls: [res]
                            })
                        }
                        else {
                            wx.showModal({
                                title: '',
                                content: JSON.stringify(res),
                            })

                            console.log(res)
                        }

                        // that.hideCropper() //隐藏，我在项目里是点击完成就上传，所以如果回调是上传，那么隐藏掉就行了，不用previewImage
                    }
                })
            }
        })
    }
})
```
