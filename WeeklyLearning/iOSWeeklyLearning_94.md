# iOS 摸鱼周报 #94 | 前端项目开发流程学习

![](https://cdn.zhangferry.com/Images/moyu_weekly_cover.jpeg)

### 本期概要

> * 本期话题：Apple 展示无障碍软件辅助新功能
> * 本周学习：浏览器插件 Glarity 学习
> * 内容推荐：推荐近期的一些优秀博文，涵盖：#file 的行为变更、如何使用 @_exported 属性、In-App Purchase 实践总结等方面的内容。
> * 摸一下鱼：跟自己和解、接受自己、做自己的人很酷

## 本期话题

### [Apple 展示无障碍软件辅助新功能](https://www.apple.com.cn/newsroom/2023/05/apple-previews-live-speech-personal-voice-and-more-new-accessibility-features/ "Apple 展示无障碍软件辅助新功能")

[@远恒之义](https://github.com/eternaljust)：Apple 一直秉承制造所有人都能顺畅使用产品的理念，为迎接本周四的全球无障碍宣传日（每年五月第三个星期四），展示了一套为认知能力、视力、听力与肢体活动能力而设计的软件辅助功能。这些功能将于今年晚些时候推出，预计会在秋季的 iOS 17 正式版系统中上线。

针对有认知障碍的用户，Apple 推出 Assistive Access，把电话、信息、相机、照片与音乐等 App 整合为一个单独的 App，并提炼出这些常用 App 的基本功能，以减轻用他们的认知负担。这相当于是系统层级的长辈模式，简化原本复杂的交互逻辑，突出放大文字按钮操作。

对于有语言能力障碍的用户，通过 Live Speech 新功能，他们可以在 iPhone 上打电话时键盘输入文字，当前设备会将文字转成语音并朗读出来。此外，对于面临失语风险的用户，他们可使用 iPhone 录制 15 分钟的音频，通过机器学习的 Personal Voice 技术，创建与自己嗓音相似的 AI 语音，后续就能使用此语音进行文字朗读。

面对失明或低视力用户，iPhone 内置的放大器将新增 Point and Speak 功能，结合相机与激光雷达扫描，视障用户在使用微波炉等家用电器时，iPhone 能识别在按键区移动的手指，朗读出手指指向按键上的功能文字。

## 本周学习 

整理编辑：[zhangferry](https://zhangferry.com)

[Glarity](https://github.com/sparticleinc/chatgpt-google-summary-extension "Glarity") 是 AI 辅助插件中做的非常好的一个项目，一起来学习下它是如何开发以及如何设计的。这一期主要介绍该插件的技术选项，以及前端的一些开发概念。后面还会有一期来介绍它对于特定功能的实现逻辑以及设计解析。

![](https://cdn.zhangferry.com/Images/Glarity-tech.png)

### 技术选项

#### 项目框架

该项目主要采用 TypeScript、React、Tailwind CSS 完成，这几个东西是对传统前端三大项的升级。

TypeScript 可以理解为对 JavaScript 的扩展。JS 设计的比较早，且是作为浏览器的辅助脚本使用，但现代对 JS 的诉求已经越来越多，开发的项目也越来越大。为了解决这个问题，微软发明了 TypeScript，这门语言的主要目的是在 JS 的基础之上添加了静态类型，用于解决 JS 类型系统过于弱化的问题，同时 TypeScript 还支持可选类型、接口、泛型、枚举等编程特性。TypeScript 无法独立执行，它需要先编译成 JavaScript。

React 可以理解为对 HTML 的封装。通过原始的 HTML 去写前端样式，是非常痛苦的，既没有组件复用的能力，这会导致大一点的项目很难维护；也没有虚拟 DOM 的概念，直接操作 DOM 对性能损耗是非常大的。同时为了方便地在 JS 中操作类 HTML 标签语法，让 HTML 和 JS 在写法上融合，React 引入了 JSX，与此同时该写法对应到 TypeScript 就是 tsx 文件。

```typescript
// src/content-script/index.tsx
async function Run() {
  const userConfig = await getUserConfig()
  const siteRegex = new RegExp(
    Object.values(config)
      .map((v) => {
        return v.regex
      })
      .join('|'),
  )
  const container = document.createElement('section')
  container.className = 'glarity--summary'
  document.body.prepend(container)
  render(
    <PageSummary
      pageSummaryEnable={userConfig.pageSummaryEnable}
      pageSummaryWhitelist={userConfig.pageSummaryWhitelist}
      pageSummaryBlacklist={userConfig.pageSummaryBlacklist}
      siteRegex={siteRegex}
    />,
    container,
  )
}
```

Tailwind CSS 可以理解为对常用 CSS 的封装，它提供了很多封装好的基础控件，便于快速开发。另一方面 CSS 作为配置文件，难扩展，不易维护。SCSS 是一种 CSS 预处理器，它能融合一部分编程语言的能力，比如变量定义、继承、嵌套、计算等这些功能，类似 TypeScript 之于 JavaScript，CSS 套上一层，以达到便于维护的目的就有了 SCSS。所有的 SCSS 文件最终也需要转成 CSS。

```scss
// content-script/dark.scss
.glarity--container.gpt--dark {
  .glarity--chatgpt {
    color: white;
    border-color: #3c4043;
    background-color: #0d1117;

    .glarity--main__loading {
      background-color: #0d1117;
    }
  }
}
```

#### 开发流程

**包管理**

衡量一个领域发展是否繁荣，完全可以通过它里面轮子的数量来判断，而前端无疑是轮子最多的开发领域。前端最常用的包管理工具有 npm 和 yarn，yarn 由 Facebook 推出，作为 npm 的替代者，主要目的是用于加速包安装。维护项目依赖了哪些库的文件是 `package.json`，它里面主要记录了当前项目的信息以及包依赖，该文件只有极个别字段在两个包管理工具是不兼容，所以大部分情况我们都可以选择 yarn 来管理这些包。

```json
//package.json
{
  "name": "chatgpt-google-summary-extension",
  "author": "givebest",
  "scripts": {
    "dev": "cross-env NODE_ENV='dev' node build.mjs",
    "lint": "eslint --ext .js,.mjs,.jsx ."
  },
  "dependencies": {
  	"gpt3-tokenizer": "^1.1.5",
    "inter-ui": "^3.19.3",
    "jquery": "^3.6.3",
  },
  "devDependencies": {
    "cross-env": "^7.0.3",
    "esbuild": "^0.17.4",
    "eslint": "^8.30.0"
  }
}
```

**代码风格**

每个语言都有自己的代码风格，相应的也都有维护代码风格的 Lint 工具。Glarity 使用了 Prettier 和 ESLint 来管理代码风格，前者主要作用于代码风格约束，如每行最大宽度 100，代码结尾不使用分号，tab 宽度为 2 等。后者主要用于代码质量，语法错误等。

Lint 工具还可以和 Git hook 绑定，利用一个前端库 husky，在 pre-commit 时添加 prettier 操作，这样每次执行 commit 的时候就能保证自动格式化代码了。

```bash
# pre-commit
. "$(dirname -- "$0")/_/husky.sh"
npx lint-staged

# package.json
{
  "lint-staged": {
    "src/*.{js,jsx,ts,tsx,mjs}": [
      "npx prettier --write",
      "npx eslint --fix"
    ]
  }
}
```

这里的 lint-staged 是一个 git 暂存工具，用于优化 lint 速度。

**编译打包**

通常编译打包的目的都是为了让最终产物能在目标系统运行，同时做一些代码优化。对前端项目来说，这个目标系统就是浏览器，浏览器能够直接识别的是 JavaScript/HTML/CSS 文件，所以最终的产物都是这些内容。

常用的工具有 webpack、esbuild 等，前者功能更丰富，适用于较负责项目；后者更快速，配置也比较简单。它们会把对 TypeScript、React、SCSS 还有其他依赖框架处理为标准的 JavaScript/HTML/CSS。TypeScript 的编译对应一个配置文件 `build.mjs`，可以指定环境变量、文件编译范围、导出等编译配置。

```javascript
let buildConfig = {
  entryPoints: [
    'src/content-script/index.tsx',
    'src/background/index.ts',
    'src/options/index.tsx',
    'src/popup/index.tsx',
  ],
  bundle: true,
  outdir: outdir,
  minify: true,
  drop: ['console', 'debugger']
}

async function runEsbuild() {
  await esbuild.build(buildConfig)
}
```

打包产物不只是格式转换，还会有代码修剪，导入很多包之后的 node_modules 通常会非常大，但真正需要的仅仅是一小部分。打包工具会把解析每个文件的导入模块，根据依赖关系，仅导入被需要的模块到包里。

```javascript
// node_modules/preact/dist/preact.module.js
function h(n2, l3) {
  for (var u3 in l3)
    n2[u3] = l3[u3];
  return n2;
}
function p(n2) {
  var l3 = n2.parentNode;
  l3 && l3.removeChild(n2);
}
```

以上是 `popup.js` 里的一小段代码，注释里标注了这是来源于 `preact` 这个库的代码。为了优化代码体积和解析速度，变量名被简化为无意义的 n2、l3 等，同时还实现了代码混淆。

#### 浏览器插件

**插件组成**

Chrome 插件需要这几个文件：

* manifest.json：插件的描述文件，填入插件名称、icon、权限等信息。

* popup：插件弹窗，一般用来做交互的入口，需要对应的 JS、HTML、CSS 文件。

* options：设置界面，用于配置插件能力，这里用于控制 GPT 模型选择，页面主题等。Chrome 会提供一个标准的 options 入口，插件 icon 右击之后的 `Options选项`，但页面内容需要全新定制。该页面入口可以在 manifest.json 里配置：

	```json
	"options_ui": {
    "page": "options/options.html"
  }
	```

* content.js 和 background.js：前者用于实现跟页面内容之间的交互，可以理解为「前端逻辑」，后者用于数据处理和与 content.js 的通信，可以理解为「后端逻辑」。当前页面内容的读取都在 content.js 里，主要包括页面 DOM 元素解析、 prompt 处理；background.js 接受数据并发起 OpenAI 的请求，然后将结果回传给 content.js。

**插件功能**

* 数据存储： `chrome.storage.sync`提供 set 和 get 方法，用于存储一些配置。
* 页面间通信：通过`chrome.runtime.sendMessage`和`chrome.runtime.onMessage.addLister`来实现不同场景的消息传递。

原生的这些方法使用都比较繁琐，Glarity 使用了 `webextension-polyfill` 这个库，它把Chrome 提供的 API 都进行了一层封装，使用起来会简单很多。

## 内容推荐

推荐近期的一些优秀博文，涵盖：#file 的行为变更、如何使用 @_exported 属性、In-App Purchase 实践总结等方面的内容。

整理编辑：[东坡肘子](https://www.fatbobman.com/)

1、[使用 @_exported 属性避免大规模重构](https://www.polpiella.dev/how-we-avoided-a-big-refactor-with-the-exported-attribute "使用 @_exported 属性避免大规模重构") -- 作者：Pol Piella Abadia

[@东坡肘子](https://www.fatbobman.com/): 本文介绍了如何在实际项目中使用 Swift 的 @_exported 属性，以最小化变更的影响并降低引入错误的风险。文章提供了一个用例，其中 @_exported 属性被用来减少重构所影响的文件数量。这种方法的优点是可以减少应用程序中的更改数量，从而降低引入错误的风险。尽管这种方法对于特定的用例非常有效，但在 Swift 单库之外使用 @_exported 属性是不被鼓励的。因此，在使用此方法时，需要仔细考虑其长期影响，并确保使用此属性的场景是必要的，并且仅在必要时使用。

2、[Swift 5.8 中 #file 的行为变更](https://sarunw.com/posts/file-behavior-change/ "Swift 5.8 中 #file 的行为变更") -- 作者：Sarunw

[@东坡肘子](https://www.fatbobman.com/): Swift 5.8 带来了 #file 字面表达式行为的新变化，本文对此进行了介绍。在之前的版本中，#file 和 #filePath 返回相同的结果，即它所在文件的路径。然而，在 Swift 5.8 中，#file 已经被修改为仅返回文件名和模块，而不包括路径。由于这个变化是一项破坏性变化，可能会影响当前代码，在默认情况下该行为被禁用，用户需要使用特性标志启用它（在 Swift 6 中将强制开启）。为了更具体地描述这个行为，开发人员还可以使用 #filePath 和 #fileID 字面表达式。前者返回它所在文件的路径，而后者返回文件名和模块。

3、[云音乐中 In-App Purchase 实践总结篇](https://juejin.cn/post/7233948883045941303 "云音乐中 In-App Purchase 实践总结篇") -- 作者：0linatan0 网易云音乐技术团队

[@东坡肘子](https://www.fatbobman.com/): IAP 的使用曾经备受开发者批评，其中包括商品创建流程过于繁琐和接入自动续费时遇到的许多问题。本文总结了网易云音乐在 In-App Purchase 实践中遇到的问题以及解决方案，包括票据验证、自动续费、退款等内容。作者还介绍了他们开发的基础库 NEStoreKit。通过对业务流程进行抽象，各团队可以快速接入，从而保障支付履约完成、完善交易场景并记录各交易日志。

4、[WWDC 2023，我期待 Core Data 带来的新变化](https://www.fatbobman.com/posts/What-I-Hope-to-See-for-Core-Data-at-WWDC-2023/ "WWDC 2023，我期待 Core Data 带来的新变化") -- 作者：东坡肘子

[@东坡肘子](https://www.fatbobman.com/): 本文列举了作者期待在 WWDC 2023 中看到 Core Data 带来的新变化。其中包括传说中的 Swift 重制版（尽管可能性极低）、用 Swift 重制部分 API、支持更多 SQLite 新特性、更好的 Model Editor 体验、完善 Core Data with CloudKit 的部分 API 以及改善 Core Data with CloudKit 的同步表现。作者希望苹果能够继续发扬这个拥有悠久历史的框架，焕发其第二春。

5、[macOS Swift 原生项目集成 Python3 运行环境](https://juejin.cn/post/7229310327268032569 "macOS Swift 原生项目集成 Python3 运行环境") -- 作者：别nil了

[@东坡肘子](https://www.fatbobman.com/): 与 Swift 相比，Python 拥有更多的第三方库。如果能在 Swift 中使用这些库，开发者可以节省大量时间和精力。本文介绍了在 macOS Swift 原生项目中集成 Python3 运行环境的方法，内容涉及：设置 Python.xcframework、添加 SystemConfiguration.Framework、检查 python-stdlib、创建 Python 头文件、添加 Run Script、检查 Python3 运行环境、使用 pip3 安装第三方依赖库、Python 第三方依赖库的调用方法等方面，并对有关 App Store 审核和打包的 bug 等内容也做了探讨。

## 摸一下鱼

整理编辑：[zhangferry](https://zhangferry.com)

上周六左耳朵耗子突发心梗去世，我这前几天还看他发的推，一个说话风格犀利的形象明明还很清晰，人却说没就没了。程序员、猝死，这两个组合关键词又一次警醒大家对健康、对工作方式的重视。我看到一则推文非常有感触，来自 Bear Liu：

1、现在的时代，不要想着抓住所有机会，不错过风口。生活是一条大河，个人无法停住每一滴水，特别是在技术领域。

2、人到了一定年纪，身体会有一些信号，自己如果注意，是可以感觉得到的。比如久坐腰会痛，下蹲膝盖会疼，要观察自己身体的这些信号，而且要能get它们：那是身体在试图引起你的注意。

3、不要太逼自己。觉得不去健身房 HIIT 就是虚度光阴，不熬夜读书就是浪费生命。放松，躺平，不可耻，也不用回避。较劲太紧，弦是会断的。只要断了，就没有机会了。

4、人总是有选择的。很多时候，其实只要看到“这只是份工作”，也就还好。对于创业者，更应该知道自己弦的张力。

5、如果是很少抽离的人，需要学习抽离（比如坐在电脑前几小时不动就是），如果是很少专注的人，应该学习专注。

6、一些身体的运动（可能所有的运动），是有助于心理健康的。这才是运动的outcome，而不是数字，成绩或是酷炫的照片。

7、中途放弃，不可耻，也不是错误。一本书读到中途不想读，可以放弃，工作、项目、感情，都可以是一段旅程，终点和起点由自己的心来决定，而不是外界的某个指标。

8、活着的时候好好活，因为我们会死很久。

以前总感觉那些通宵写代码，工作狂，比自己很多优秀也比自己努力很多的人非常酷酷。AI 这波趋势，看到很多人做出了很多很棒的东西，会反思自己也想到过那个点子，为什么却没有做出来，自己自制力为什么那么差。现在想想，那些东西放在整个人生阶段还有那么重要吗？陪伴亲人、感受生活感受世界的更多面，不太沉浸到一个小小的世界里，同时心态轻松、跟自己和解、接受自己、做自己，也是很酷的。

## 关于我们

iOS 摸鱼周报，主要分享开发过程中遇到的经验教训、优质的博客、高质量的学习资料、实用的开发工具等。周报仓库在这里：https://github.com/zhangferry/iOSWeeklyLearning ，如果你有好的的内容推荐可以通过 issue 的方式进行提交。另外也可以申请成为我们的常驻编辑，一起维护这份周报。另可关注公众号：iOS成长之路，后台点击进群交流，联系我们，获取更多内容。

### 往期推荐

[iOS 摸鱼周报 #93 | AIGC 尝试](https://mp.weixin.qq.com/s/ios0QYKmnYtJ8URvZLJ1TA)

[iOS 摸鱼周报 #92 | Swift Foundation 预览版发布](https://mp.weixin.qq.com/s/AQaY2DA2h8S-XEYoQ0u7Ew)

[iOS 摸鱼周报 #91 | 免费的网站托管平台 Vercel](https://mp.weixin.qq.com/s/93YLa8ankkEVcp4pop2A6A)

[iOS 摸鱼周报 #90 | 面相任务的 GPT 项目诞生](https://mp.weixin.qq.com/s/Bx8N9HqMP5HE9mzy6l3QVA)

![](https://cdn.zhangferry.com/Images/WechatIMG384.jpeg)
