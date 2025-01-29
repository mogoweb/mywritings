# Chromium 中的 WebUI

这段时间在基于 Chromium 做浏览器的定制工作，少不了需要修改 Chromium 的 UI。Chromium 中的 UI 主要有两大部分组成，一部分是原生 UI，也就是使用 C++ 等语言，利用操作系统原生 UI 框架开发的界面，另一部分则是采用 Web 技术开发的界面，称之为 WebUI。

WebUI 开发起来比较麻烦，因为涉及到与 C++ 代码的交互，让前端开发人员开发，需要安装 Chromium 的编译环境，而且 WebUI 使用了 Chromium 特有的框架，和前端开发人员的技术栈并不同，没法直接交给前端开发人员。如果让 C++ 开发人员开发，有需要懂前端开发。也不知道谷歌为啥整出这么个东西，感觉挺麻烦的。

这与我之前在项目中用的 H5 UI 开发方案有所不同。尽管 H5 UI 开发也涉及到与 C++ 代码的交互，但交互主要限于业务逻辑层面。界面的开发完全由前端负责，C++ 代码只负责通过 JS 和 C++ 之间的交互提供数据。

那什么是 Chromium WebUI 呢？

在 Chromium 中，WebUI（Web User Interface）是一个用于构建 Web 应用界面（UI）的框架，广泛应用于浏览器的设置界面、扩展管理、历史记录等功能。这些界面通常由 HTML、CSS 和 JavaScript 编写，并且需要与 Chromium 内部的 C++ 代码进行交互。WebUI 使得开发者能够通过 Web 技术来构建复杂的 UI，同时保持与 Chromium 内部系统的高效通信。

## WebUI 和 Web 页面的不同

WebUI 比 Web 页面的权限更高，以便管理Chrome浏览器本身。例如，要实现设置界面，必须访问许多隐私和安全敏感的服务，普通 Web 页面并不允许访问这些服务。

WebUI 通常使用特定的 URL，可以通过安全子系统的检查：

* 允许渲染器加载 chrome:// 开头的URL，访问浏览器的内部资源（如 chrome://resources/ ）。
* 允许浏览器通过 CallJavascriptFunction() 在渲染器中执行任意 JavaScript 代码。
* 允许通过 chrome.send() 等方法在渲染器和浏览器之间进行通信。
* 在显示图片或执行 JavaScript 时忽略安全设置。

## WebUI 的架构

WebUI 的架构可分为以下几个主要部分：

* WebUI 页面：每个 WebUI 页面都是一个典型的 Web 应用页面，包含 HTML、CSS 和 JavaScript。通常通过浏览器的 chrome://URL 可以访问这些页面，如 chrome://settings 。
* C++ 后端：Chromium 的 C++ 代码提供 WebUI 页面所需的业务逻辑和数据支持。C++ 后端负责通过 IPC（进程间通信）机制与 Web 页面进行交互，并处理各种浏览器内部功能（如设置、网络请求等）。
* UI 组件库：WebUI 提供了一套封装良好的 UI 组件库，如按钮、复选框、输入框等，这些组件与浏览器的 UI 风格高度一致，并且能够快速构建可交互的页面。

## WebUI 代码结构

WebUI 页面主要由以下部分组成：

+ WebUI 页面资源： 这些资源存放在 `chrome/browser/resources` 目录下。
+ C++ 控制器： 与 WebUI 页面交互的 C++ 代码，位于 `chrome/browser/ui/webui/`。

## 创建 WebUI 页面

### 1. 创建文件夹结构

我们首先需要在 `chrome/browser/resources` 和 `chrome/browser/ui/webui` 中为新页面创建文件夹。比如，为了创建一个名为 "hello_world" 的页面，我们在这两个目录下分别创建 `hello_world` 文件夹。

### 2. 编写 WebUI 页面代码

我们将通过一组文件来创建一个简单的 WebUI 页面。

`chrome/browser/resources/hello_world/hello_world.html`

```html
<!DOCTYPE HTML>
<html>
  <meta charset="utf-8">
  <link rel="stylesheet" href="hello_world.css">
  <hello-world-app></hello-world-app>

  <script type="module" src="app.js"></script>

</html>
```

`chrome/browser/resources/hello_world/hello_world.css`

```css
body {
  margin: 0;
}
```

`chrome/browser/resources/hello_world/app.css`

```css
#example-div {
  color: blue;
}
```

`chrome/browser/resources/hello_world/app.html.ts`

```typescript
import { html } from '//resources/lit/v3_0/lit.rollup.js';
import type { HelloWorldAppElement } from './app.js';

export function getHtml(this: HelloWorldAppElement) {
  return html`
    <h1>Hello World</h1>

    <div id="example-div">${this.message_}</div>

  `;
}
```

`chrome/browser/resources/hello_world/app.ts`

```typescript
import './strings.m.js';
import { loadTimeData } from 'chrome://resources/js/load_time_data.js';
import { CrLitElement } from '//resources/lit/v3_0/lit.rollup.js';
import { getCss } from './app.css.js';
import { getHtml } from './app.html.js';

export class HelloWorldAppElement extends CrLitElement {
  static get is() {
    return 'hello-world-app';
  }

  static override get styles() {
    return getCss();
  }

  override render() {
    return getHtml.bind(this)();
  }

  static override get properties() {
    return {
      message_: { String },
    };
  }

  protected message_: string = loadTimeData.getString('message');
}

customElements.define(HelloWorldAppElement.is, HelloWorldAppElement);
```

### 3. 配置构建文件

为了正确编译 TypeScript 并生成 JavaScript 文件，我们需要添加一个 `BUILD.gn` 文件。

`chrome/browser/resources/hello_world/BUILD.gn`

```plain
import("//ui/webui/resources/tools/build_webui.gni")

build_webui("build") {
  grd_prefix = "hello_world"

  static_files = [ "hello_world.html", "hello_world.css" ]
  non_web_component_files = [ "app.ts", "app.html.ts" ]
  css_files = [ "app.css" ]

  ts_deps = [
    "//third_party/lit/v3_0:build_ts",
    "//ui/webui/resources/js:build_ts",
  ]
}
```

### 4. 将 WebUI 页面资源添加到项目中

为了将新创建的 WebUI 页面资源添加到构建配置中，我们需要更新以下文件。

更新 `chrome/browser/resources/BUILD.gn`

```plain
group("resources") {
  public_deps += [
    "hello_world:resources"
  ]
}
```

更新 `tools/gritsettings/resource_ids.spec`

```plain
# START chrome/ WebUI resources section
"<(SHARED_INTERMEDIATE_DIR)/chrome/browser/resources/hello_world/resources.grd": {
  "META": {"sizes": {"includes": [5]}},
  "includes": [2085],
}
```

更新 `chrome/chrome_paks.gni`

```plain
template("chrome_extra_paks") {
  sources += [
    "$root_gen_dir/chrome/hello_world_resources.pak",
  ]
  deps += [
    "//chrome/browser/resources/hello_world:resources",
  ]
}
```

## 添加 C++ 处理请求的 WebUI 类

在 C++ 中，我们需要创建一个 WebUI 控制器类来处理对新页面（`chrome://hello-world/`）的请求。这个类通常会继承自 `WebUIController`。

`chrome/browser/ui/webui/hello_world/hello_world_ui.h`

```cpp
#ifndef CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_HELLO_WORLD_H_
#define CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_HELLO_WORLD_H_

#include "content/public/browser/web_ui_controller.h"

// The WebUI for chrome://hello-world
class HelloWorldUI : public content::WebUIController {
 public:
  explicit HelloWorldUI(content::WebUI* web_ui);
  ~HelloWorldUI() override;
};

#endif  // CHROME_BROWSER_UI_WEBUI_HELLO_WORLD_HELLO_WORLD_H_
```

`chrome/browser/ui/webui/hello_world/hello_world_ui.cc`

```cpp
#include "chrome/browser/ui/webui/hello_world/hello_world_ui.h"
#include "chrome/common/webui_url_constants.h"
#include "content/public/browser/web_ui_data_source.h"

HelloWorldUI::HelloWorldUI(content::WebUI* web_ui)
    : content::WebUIController(web_ui) {
  content::WebUIDataSource* source = content::WebUIDataSource::CreateAndAdd(
      web_ui->GetWebContents()->GetBrowserContext(),
      chrome::kChromeUIHelloWorldHost);

  // 添加必要的资源
  webui::SetupWebUIDataSource(
      source,
      base::make_span(kHelloWorldResources, kHelloWorldResourcesSize),
      IDR_HELLO_WORLD_HELLO_WORLD_HTML);

  source->AddString("message", "Hello World!");
}

HelloWorldUI::~HelloWorldUI() = default;
```

更新 `chrome/browser/ui/BUILD.gn`

```plain
static_library("ui") {
  sources = [
    "webui/hello_world/hello_world_ui.cc",
    "webui/hello_world/hello_world_ui.h",
  ]
}
```

### 注册 URL 常量

我们需要为新的 WebUI 页面注册 URL 常量，以便浏览器能够识别。

`chrome/common/webui_url_constants.cc`

```cpp
const char kChromeUIHelloWorldURL[] = "chrome://hello-world/";
const char kChromeUIHelloWorldHost[] = "hello-world";
```

`chrome/common/webui_url_constants.h`

```cpp
extern const char kChromeUIHelloWorldURL[];
extern const char kChromeUIHelloWorldHost[];
```

完成所有配置后，您可以编译并运行 Chromium，然后访问 `chrome://hello-world/` 查看页面。如果一切顺利，您应该能看到 "Hello World!" 消息。

## 小结

上面的例子只是一些关键代码，并非一个完整的例子，有兴趣的朋友可以看看 chromium 源码中的例子。即使这样，看了上面的内容，大家是不是也觉得，创建一个简单的 WebUI 页面其实并不轻松？尤其是如果你深入研究 Chromium 中设置页面的源码，你会发现，这一切变得更加复杂。因为这些页面不仅仅是简单的 HTML 和 CSS，它们混合了 Web Components 和 Polymer 框架，这使得修改和定制变得异常痛苦。每次想要改动一处小细节，往往需要对多个技术栈、不同的框架和抽象层次进行深入理解。

事实上，这种复杂性几乎逼迫每个程序员都必须成为“六边形战士”，不仅要熟悉 C++ 编程，还要理解前端技术、框架以及如何在它们之间实现无缝衔接。为了搞定 WebUI，我不得不硬着头皮去学习前端相关的知识。这对我来说，本是一个前端开发者的领域，却因为 Chromium 的特殊需求，让我这个 C++ 程序员也必须在前端的海洋中挣扎一番。


