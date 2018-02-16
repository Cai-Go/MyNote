# JSON 配置
嗯，就是 **.json**  后缀的 JSON 配置文件。
在项目的根目录有一个 app.json 和 project.config.json，此外在 pages/logs 目录下还有一个 logs.json。

## 小程序配置 app.json
app.json 是对当前小程序的全局配置，包括了小程序的所有页面路径、界面表现、网络超时时间、底部 tab 等。
````json
{
  "pages": [],
  "window": {},
  "tabBar": {
    "list": [{}, {}]
  },
  "networkTimeout": {},
  "debug": true
}
````
含义：

- pages字段 —— 用于描述当前小程序所有页面路径，String Array 类型，必填项。
    + 接受一个数组，每一项都是字符串，来指定小程序由哪些页面组成。每一项代表对应页面的【路径+文件名】信息，数组的第一项代表小程序的初始页面。小程序中新增/减少页面，都需要对 pages 数组进行修改。
- window字段 —— 小程序所有页面的顶部背景颜色，文字颜色，Object 类型，选填项。
    + 用于设置小程序的状态栏、导航条、标题、窗口背景色。
    
    | 属性        | 类型    |   默认值  |  描述  |
    | --------    | -----:  |   :----:  | :----: |
    |navigationBarBackgroundColor|HexColor|#000000|导航栏背景颜色|
    |navigationBarTextStyle|String|white|导航栏标题颜色|
    |navigationBarTitleText|String|           |导航栏标题文字内容|
    |navigationStyle|String|default|导航栏样式，仅支持 default/custom。custom 模式可自定义导航栏，只保留右上角胶囊状的按钮|
    |backgroundColor|HexColor|#ffffff|窗口的背景色|
    |backgroundTextStyle|String|dark|下拉背景字体、loading 图的样式，仅支持 dark/light|
    |enablePullDownRefresh|Boolean|false|是否开启下拉刷新|
    |onReachBottomDistance|Number|50|页面上拉触底事件触发时距页面底部距离，单位为px|
   
   >注：navigationStyle 只在 app.json 中生效。开启 custom 后，低版本客户端需要做好兼容。开发者工具基础库版本切到 1.7.0（不代表最低版本，只供调试用） 可方便切到旧视觉。

- tabBar ——— 设置底部 tab 的表现，Object 类型，选填项。
    + 当设置 position 为 top 时，将不会显示 icon。
    + tabBar 中的 list 是一个数组，只能配置最少2个、最多5个 tab，tab 按数组的顺序排序。
    + 属性说明：
    
    |属性|类型|必填|默认值|描述|
    | --------    | -----:   | :----: |:----: |:----: |
    | color|HexColor | 是 |  | tab 上的文字默认颜色 |
    | selectedColor | HexColor | 是 |  | tab 上的文字选中时的颜色 |
    | backgroundColor | HexColor | 是 |  | tab 的背景色 |
    | borderStyle|String |否|black| tabbar上边框的颜色， 仅支持 black/white |
    | list | Array | 是 |  | tab 的列表，详见 list 属性说明，最少2个、最多5个 tab |
    | position | String | 否 | bottom | 可选值 bottom、top |

    其中 list 接受一个数组，数组中的每个项都是一个对象，其属性值如下：

    |属性|类型|必填|说明|    
    | --------   | -----:   | :----: |:----: |
    | pagePath   | String   |  是    |页面路径，必须在 pages 中先定义|
    | text       | String   |  是    | tab 上按钮文字 |
    | iconPath   | String   |  否    | 图片路径，icon 大小限制为40kb，建议尺寸为 81px * 81px，当 postion 为 top 时，此参数无效，不支持网络图片 |
    | selectedIconPath | String | 否 | 选中时的图片路径，icon 大小限制为40kb，建议尺寸为 81px * 81px ，当 postion 为 top 时，此参数无效 |

- networkTimeout ————  设置网络超时时间，Object 类型，选填项。
    + 属性说明：

    |属性|类型|必填|说明|    
    | --------   | -----:   | :----: |:----: |
    | request   | Number   |  否    | wx.request的超时时间，单位毫秒，默认为：60000 |
    | connectSocket | Number |  否   | wx.connectSocket的超时时间，单位毫秒，默认为：60000  |
    | uploadFile | Number | 否 |    wx.uploadFile的超时时间，单位毫秒，默认为：60000 |
    | downloadFile |  Number | 否 | wx.downloadFile的超时时间，单位毫秒，默认为：60000 |

- debug ————  设置是否开启 debug 模式，Boolean 类型，选填项。
    + 可以在开发者工具中开启 debug 模式，在开发者工具的控制台面板，调试信息以 info 的形式给出，其信息有Page的注册，页面路由，数据更新，事件触发 。


## 工具配置 project.config.json
这个没事好说的，就是省去了你如果换了开发环境得每次重新配置的麻烦。你只要载入同一个项目的代码包，开发者工具就自动会帮你恢复到当时你开发项目时的个性化配置，其中会包括编辑器的颜色、代码上传时自动压缩等等一系列选项。

在项目根目录使用 project.config.json 文件对项目进行配置。

|字段名|类型|说明|    
| --------   | -----:   | :----: |
| miniprogramRoot | Path String | 指定小程序源码的目录(需为相对路径) | 
| qcloudRoot | Path String | 指定腾讯云项目的目录(需为相对路径) |
| setting | Object | 项目设置 |
| libVersion | String  | 基础库版本 |
| appid | String | 项目的 appid，只在新建项目时读取 |
| projectname | String | 项目名字，只在新建项目时读取 |   

setting 中可以指定以下设置：

|字段名|类型|说明|    
| --------   | -----:   | :----: |
| es6 | Boolean | 是否启用 es5 转 es6 | 
| postcss | Boolean | 上传代码时样式是否自动补全 |
| minified | Boolean | 上传代码时是否自动压缩 |
| urlCheck | Boolean  | 是否检查安全域名和 TLS 版本 |


## 页面配置 page.json
用来表示 pages/logs 目录下的 logs.json 这类和小程序页面相关的配置，让开发者可以独立定义每个页面的一些属性。

每一个小程序页面也可以使用.json文件来对本页面的窗口表现进行配置。 页面的配置比app.json全局配置简单得多，只是设置 app.json 中的 window 配置项的内容，页面中配置项会覆盖 app.json 的 window 中相同的配置项。
页面的.json只能设置 window 相关的配置项，以决定本页面的窗口表现，所以无需写 window 这个键。如下所示：
````json
{
  "navigationBarBackgroundColor": "#ffffff",
  "navigationBarTextStyle": "black",
  "navigationBarTitleText": "微信接口功能演示",
  "backgroundColor": "#eeeeee",
  "backgroundTextStyle": "light"
}
````
