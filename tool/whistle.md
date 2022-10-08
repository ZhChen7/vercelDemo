

# whistle 抓包工具使用指南

### 说明

**whistle** 是一款跨平台的网络抓包调试工具，基于**node**开发。支持抓包，重放，替换，修改等方式来调试 **http(s),****WebSocket**请求，也可以作为普通的http代理。

其功能和常用的fiddler(windows),Charles(Mac)工具功能相同，不过对于开发者更加友好，操作和调试更加方便，还支持node模块的插件。

- 以下如无特别说明，操作指令支持win/mac双端

# 安装 node安装

- ## windows安装

- -  下载地址：https://nodejs.org/en/download/，选择Windows Installer安装(选择系统版本，可在「我的电脑」右键「计算机属性」查看)；下载完成后一路默认安装即可

## mac安装

1. 首先安装nodejs，前往官网进行nodejs的下载。https://nodejs.org/en/下载后点击安装包进行安装即可；通过npm -v可查看是否安装成功

```undefined
whistle 安装
sudo
npm install -g whistle
```

### 启动: whistle

启动 :  w2 start  

重启 :  w2 restart 停止w2 stop     成功启动后显示下述信息：

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=N2JkZTNjZTAwN2IzM2Y0MDcxNGRjNzYzZTc5Yzk1OTJfa2NqQkl4ZGdRdXk0WUo0R0c1MjkzRFQ4aXFkZDVuYkRfVG9rZW46Ym94Y25vczRyb3RSQnN1WWl2OFA2S1pPWXJjXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)

(http://10.72.12.16:8899/ 是对外IP，用于移动设备访问，需要在移动设备配置网络代理，详见后述步骤) 此时打开浏览器输入127.0.0.1:8899，页面如下图所示：

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=YTk1YWMyOWU1ZWIzMDlmZTBiMThhN2Q2MDM4YzA5MDVfWXRrRzdaeXFCRnRMS0c5Zlc3ajltcFdvYjlmbnRDY2FfVG9rZW46Ym94Y25LaUlyakxvNWh2SzhIR3pVV0JWemFnXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)

## Mock 数据

```undefined
10.213.214.50:9004/crm-teaching-preparation/ resBody://{beike1.json}
```

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=YTk3NzJmODJiM2QwMWQ1OGViNTkwOThiNDYyMTNlMTJfclRBRlcxcGFraGFEdk9LamwxRUpaT29xazM0b0pETVdfVG9rZW46Ym94Y24xaVduaHZZRVd2elp6WjhlanNZOVNkXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)

## 证书&代理配置

**mac** **安装证书**  点击「HTTPS」选项卡，下载证书到本地，双击下载的「rootCA.crt」文件，command+space搜索「钥匙串」，进行钥匙串信任配置，操作步骤如下图所示

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=OGE4M2MyNWU1ZGNlYjc1Y2RmODYxMjk3MDQyNWVhMGJfZXFCY3BXWHRwYlloa3k5UWZqMXNvY3Vyc2pPMmxIdkJfVG9rZW46Ym94Y25wWXRJNW9yS3VneVQ3Y3o5YUZwblJmXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)

## 配合使用 Proxy SwitchyOmega

- 轻松快捷地管理和切换多个代理设置。

- 备用下载地址： https://github.com/FelisCatus/SwitchyOmega/releases

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=NDdkNWJkM2ZlY2I0NDljZGNiZWRmOWQzMDA0MWRkYjVfNEVCRHpXczQzbjdiSWc4TllJdThBOHNrZUZTbkdtZU5fVG9rZW46Ym94Y255aFQ1TERZeG5oNmEwaTdwYTdZbzlkXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)

![img](https://wrpnn3mat2.feishu.cn/space/api/box/stream/download/asynccode/?code=ZTI2N2RmZjdhZDFmYjgyYmQyZjY3M2IwZDcyZTBhOTRfVDJiUklqellwZzIzbEtXd3JIemlSaVVVNFhSQW5JbXVfVG9rZW46Ym94Y25IdHdSNmJlaGZ3YTI3ZTY5ZjdhcEdiXzE2NjUyMjg1NjI6MTY2NTIzMjE2Ml9WNA)
