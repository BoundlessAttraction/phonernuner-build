# PhoneRunner 编译工程（DSH Go XCTest 增强用）

这台 iPad（`iPad8,1` / iPadOS 26.6.1）用不了 CoreDevice 远程控制——Apple 硬性要求 iOS 27+。
唯一出路是 DSH Go 的 XCTest 增强，但它需要一个**签名安装的 PhoneRunner**，而这个包只能由 Xcode 编译出来（Windows 编不了）。

这个工程用 GitHub Actions 的免费 macOS 机器替你编，产出未签名的 `PhoneRunnerUITests-Runner.app`，你用全能签签名装上就行。

---

## 一、在 GitHub 上跑一次编译

1. 打开 https://github.com/new ，新建一个仓库：
   - 名字随便，比如 `phonernner-build`
   - 可见性选 **Public**（私有库跑 macOS runner 会扣额度，公开库免费）
   - 不要勾 README / .gitignore，建空仓库
2. 进入仓库 → **Add file → Upload files**
3. 把本目录下**所有内容**拖进去（包括 `.github` 文件夹），Commit
4. 点仓库顶部的 **Actions** 标签
   - 如果提示 "Workflows aren't being run on this forked repository" 之类，点 **I understand my workflows, go ahead and enable them**
   - 左侧选 **Build PhoneRunner** → 右侧 **Run workflow** → **Run workflow**
5. 等 5~10 分钟，刷新页面，点进那次运行
6. 页面底部 **Artifacts** 下载 `PhoneRunner-xctrunner.zip`

> 如果 Actions 标签里没有 workflow：手动新建文件 `.github/workflows/build.yml`，把本仓库同名文件内容粘进去，Commit 即可。

## 二、用全能签装到 iPad

1. 解压下载的 artifact，里面是 `PhoneRunnerUITests-Runner.app`
2. 新建文件夹 `Payload`，把 `.app` 放进去，压成 zip，改名 `PhoneRunner.ipa`
3. 全能签导入这个 ipa → 签名 → 安装
   - **不要改 bundle id**，必须是 `io.github.ccch1mneyyy.dshgo.PhoneRunner.xctrunner`
   - 如果全能签报 App Group 相关的错，说明它读到了旧 entitlements；本工程已改用空 entitlements，正常不会报

## 三、在 DSH Go 里开增强

1. 设置 → 手机操控 → **可选增强 → XCTest 增强**，打开
2. 回到对话，让 AI 读一次屏幕
3. 如果 iPad 弹 **Enable UI Automation**，在系统界面输入锁屏密码确认（只这一次）
4. 之后点按、滚动、输入都走 XCTest 通道

---

## 编译参数（已写死在 workflow 里）

- `xcodebuild build-for-testing`，`-destination 'generic/platform=iOS'`
- `CODE_SIGNING_ALLOWED=NO`：出未签名包，交给全能签
- `USES_XCTRUNNER=YES`：产出 `-Runner.app`
- `TEST_TARGET_NAME=""`：不绑定宿主 App，可跨 App 操控
- entitlements 用空文件（on-device 路径走 testmanagerd，不需要 App Group）
