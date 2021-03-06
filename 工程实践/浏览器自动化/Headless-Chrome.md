[![返回目录](https://parg.co/UYp)](https://github.com/wxyyxc1992/Web-Series/)

# Headless Chrome 实战：动态渲染、页面抓取与端到端测试

笔者往往是使用 PhantomJS 或者 Selenium 执行动态页面渲染，而在 Chrome 59 之后 Chrome 提供了 Headless 模式，其允许在命令行中使用 Chromium 以及 Blink 渲染引擎提供的完整的现代 Web 平台特性。需要注意的是，Headless Chrome 仍然存在一定的局限，相较于 Nightmare 或 Phantom 这样的工具， Chrome 的远程接口仍然无法提供较好的开发者体验。我们在下文介绍的代码示例中也会发现，目前我们仍需要大量的模板代码进行控制。

# 环境配置

在 Chrome 安装完毕后我们可以利用其包体内自带的命令行工具启动：

```
$ chrome --headless --remote-debugging-port=9222 https://chromium.org
```

笔者为了部署方便，使用[ Docker 镜像](https://hub.docker.com/r/justinribeiro/chrome-headless/)来进行快速部署，如果你本地存在 Docker 环境，可以使用如下命令快速启动：

```
docker run -d -p 9222:9222 justinribeiro/chrome-headless
```

如果是在 Mac 下本地使用的话我们还可以创建命令别名：

```
alias chrome="/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome"
alias chrome-canary="/Applications/Google\ Chrome\ Canary.app/Contents/MacOS/Google\ Chrome\ Canary"
alias chromium="/Applications/Chromium.app/Contents/MacOS/Chromium"
```

如果是在 Ubuntu 环境下我们可以使用 deb 进行安装：

```
# Install Google Chrome
# https://askubuntu.com/questions/79280/how-to-install-chrome-browser-properly-via-command-line
sudo apt-get install libxss1 libappindicator1 libindicator7
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome*.deb  # Might show "errors", fixed by next line
sudo apt-get install -f
```

chrome 命令行也支持丰富的命令行参数，`--dump-dom` 参数可以将 `document.body.innerHTML` 打印到标准输出中：

```
chrome --headless --disable-gpu --dump-dom https://www.chromestatus.com/
```

而 `--print-to-pdf` 标识则会将网页输出位 PDF：

```
chrome --headless --disable-gpu --print-to-pdf https://www.chromestatus.com/
```

初次之外，我们也可以使用 `--screenshot` 参数来获取页面截图：

```
chrome --headless --disable-gpu --screenshot https://www.chromestatus.com/


# Size of a standard letterhead.
chrome --headless --disable-gpu --screenshot --window-size=1280,1696 https://www.chromestatus.com/


# Nexus 5x
chrome --headless --disable-gpu --screenshot --window-size=412,732 https://www.chromestatus.com/
```

如果我们需要更复杂的截图策略，譬如进行完整页面截图则需要利用代码进行远程控制。

# 基于 Puppeteer 的编程控制

[Puppeteer](https://github.com/GoogleChrome/puppeteer) 是 Google

通过  headless 参数来指定是否启用 Headless 模式，默认情况下是启用的。此外，在我们使用 npm 安装 Puppeteer 的时候其会自动下载指定版本的  Chromium 从而保证接口的开箱即用性，也可以通过  executablePath 参数指定启动版本：

```js
const browser = await puppeteer.launch({headless: false}); // default is true



const browser = await puppeteer.launch({executablePath: '/path/to/Chrome'});
```

在大规模部署的情况下，我们需要控制 Puppeteer 连接到远端的服务化方式部署的 Headless Chrome 集群，此时就可以使用 `connect` 函数连接到 Headless Chrome 实例：

```js
puppeteer.connect({
  browserWSEndpoint:
    "ws://{remoteip}:9222/devtools/browser/fa60c034-422d-4f2c-bbeb-17a2cfd690f2"
});
```

# 动态渲染

## 脚本执行

```js
const puppeteer = require("puppeteer");

(async () => {
  const browser = await puppeteer.launch();

  const page = await browser.newPage();

  await page.goto("https://example.com"); // Get the "viewport" of the page, as reported by the page.

  const dimensions = await page.evaluate(() => {
    return {
      width: document.documentElement.clientWidth,

      height: document.documentElement.clientHeight,

      deviceScaleFactor: window.devicePixelRatio
    };
  });

  console.log("Dimensions:", dimensions);

  await browser.close();
})();
```

# 页面抓取

## 页面保存

```
const puppeteer = require('puppeteer');



(async () => {

  const browser = await puppeteer.launch();

  const page = await browser.newPage();

  await page.goto('https://news.ycombinator.com', {waitUntil: 'networkidle'});

  await page.pdf({path: 'hn.pdf', format: 'A4'});



  await browser.close();

})();
```

# 端到端测试
