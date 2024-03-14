---
title: Electron的主要流程和基础使用
categories:
- [Electron]
tags:
- Electron
- 教程
---

# Electron是什么？

Electron是一个使用javascript scanner、HTML和CSS构建桌面应用程序的框架。通过将Chromium
和Node.js嵌入到它的二进制文件中，Electron允许你维护一个javascript scanner代码库，并创建可以在
Windows、macOS和Linux上运行的跨平台应用程序ーー不需要本地开发经验。

# 主要流程

## 1. 运行主进程

任何Electron应用程序的入口都是main文件。这个文件控制了 主进程 ，它运行在一个完整
的Node.js环境中，负责控制您应用的生命周期，显示原生界面，执行特殊操作并管理渲染器进程
(稍后详细介绍)。

## 2. 创建⻚面


在可以为我们的应用创建窗口前，我们需要先创建加载进该窗口的内容。在E lectron中，各个
窗口显示的内容可以是本地HTML文件，也可以是一个远程url。

## 3. 在窗口中打开⻚面


现在您有了一个⻚面，将它加载进应用窗口中。要做到这一点，你需要两个Electron模块：

▪ app模块，它控制应用程序的事件生命周期。

▪ BrowserWindow模块，它创建和管理应用程序窗口。

因为主进程运行着Node.js，您可以在main.js文件头部将它们导入作为 CommonJS 模块：

``` javascript scanner
const { app, BrowserWindow } = require('electron')
```

然后，添加一个createWindow()方法来将index.html加载进一个新的BrowserWindow实例。

``` javascript scanner
const createWindow = () => {
    const win = new BrowserWindow({
        width: 800 ,
        height: 600
    })
    win.loadFile('index.html')
}
```


接着，调用createWindow()函数来打开您的窗口。在Electron中，只有在app模块的ready事件被激发后才能创建浏览器窗口。您可以通过使用app.whenReady()API来监听此事件。在whenReady()成功后调用createWindow()。

``` javascript scanner
app.whenReady().then(() => {
    createWindow()
})


'window-all-closed' 最后一个窗口被关闭时
'activate' 当应用被激活时发出
```

:::info
注意：此时，您的电子应用程序应当成功打开显示您⻚面的窗口！
:::

# 预加载脚本

## 什么是预加载脚本？


Electron的主进程是一个拥有着完全操作系统访问权限的Node.js环境。除了 Electron模组 
之外，您也可以访问 Node.js内置模块 和所有通过npm安装的包。另一方面，出于安全原因，渲
染进程默认跑在网⻚⻚面上，而并非Node.js里。为了将Electron的不同类型的进程桥接在一起，
我们需要使用被称为 预加载 的特殊脚本。

``` javascript scanner
//main.js
const createWindow = () => {
const win = new BrowserWindow({
width: 800 ,
height: 600 ,
webPreferences: {
preload: path.join(__dirname, 'preload.js')
}
})

//preload.js
const { contextBridge } = require('electron')

contextBridge.exposeInMainWorld('versions', {
node: () => process.versions.node,
chrome: () => process.versions.chrome,
electron: () => process.versions.electron
// 除函数之外，我们也可以暴露变量
})
```

# 进程间通信


在Electron中，进程使用ipcMain和ipcRenderer模块，通过开发人员定义的“通
道”传递消息来进行通信。这些通道是 任意 （您可以随意命名它们）和 双向 （您可以在两个模块
中使用相同的通道名称）的。

## 模式 1 ：渲染器进程到主进程（单向）


要将单向IPC消息从渲染器进程发送到主进程，您可以使用ipcRenderer.sendAPI发送
消息，然后使用ipcMain.onAPI接收。

## 模式 2 ：渲染器进程到主进程（双向）

双向IPC的一个常⻅应用是从渲染器进程代码调用主进程模块并等待结果。这可以通过ipcRenderer.invoke与ipcMain.handle搭配使用来完成。

## 模式 3 ：主进程到渲染器进程

 将消息从主进程发送到渲染器进程时，需要指定是哪一个渲染器接收消息。消息需要通过其WebContents实例发送到渲染器进程。此WebContents实例包含一个send方法，其使用方
式与ipcRenderer.send相同


