---
title: Docker版思源笔记加ChatGPT的超强搭配使用~
date: 2023-03-11 16:16:27
categories:
 - [我的NAS捣鼓笔记, 极空间]
tags: 
 - NAS
 - 极空间
 - 思源笔记
---

## 前言
前几天捣鼓了[自助编译极空间Z4思源笔记（siyuan）Docker项目](https://blog.aayu.today/nas/zspace/20230224/)，思源笔记的开发者D大和V姐更新真的是太频繁了，几乎每周都会有新的功能和特性，最让我感动的是我凌晨去GitHub的Issues上给他们提了些[改进意见](https://github.com/siyuan-note/siyuan/issues/7615)，结果当天就把功能完善了，所以真的很想让我吹一波🤣

最新版本的思源笔记已经更新到[v2.7.9](https://github.com/siyuan-note/siyuan/releases/tag/v2.7.9)了，在该版本中完善了`针对内容块的人工智能辅助支持`，对，就是可以在思源笔记里使用OpenAI的Chat API对笔记内容：
* [x] 续写
* [x] 翻译
  * [x] 中文
  * [x] 日文
  * [x] 韩文
  * [x] 英文
  * [x] 西班牙文
  * [x] 法文
  * [x] 德文
* [x] 提取摘要
* [x] 头脑风暴
* [x] 修正语法和拼写
* [x] 自定义操作...

:::primary
有了这些功能后，我觉得思源笔记已经很适合进行学术论文的归纳和整理了，真的是未来可期啊~
:::

## 使用
我也紧跟思源笔记官方的脚步，从v2.7.6开始每个发行版本我都会把它编译成适配于极空间Z4的[Docker镜像](https://hub.docker.com/repository/docker/ylsislove/siyuan)，所以如果有小伙伴也用的是极空间Z4，就可以用我编译好的镜像去尝试一下啦~

:::info
其他的部署环境可以直接用思源笔记官方的Docker镜像就行：b3log/siyuan:latest
本人编译的镜像只适合极空间Z4设备哦
:::

拉取最新的v2.7.9镜像版本，容器配置如下

![](https://image.aayu.today/uploads/2023/03/11/202303111602935.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111603901.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111603755.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111604932.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111604137.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111608817.png)
{.gallery  data-height="240"}

:::info
国内使用openai api key是需要点手段的，可以通过配置`SIYUAN_OPENAI_API_BASE_URL`或`SIYUAN_OPENAI_API_PROXY`环境变量来解决，这两个环境变量使用一个就行，不然多层代理后网络响应时间会变长。
* SIYUAN_OPENAI_API_BASE_URL：默认值为`https://api.openai.com/v1`，可以配置成`https://你的代理域名/v1`
* SIYUAN_OPENAI_API_PROXY：如果自己会挂代理，就可以配置成诸如`socks5://127.0.0.1:10808`的值
:::

以上配置好后，应用容器即可开始愉快的玩耍啦~

## 体验
### 续写
![](https://image.aayu.today/uploads/2023/03/11/202303111623782.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111624662.png)
{.gallery  data-height="240"}

### 翻译和提取摘要
![](https://image.aayu.today/uploads/2023/03/11/202303111632566.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111634737.png)
{.gallery  data-height="240"}

### 头脑风暴
![](https://image.aayu.today/uploads/2023/03/11/202303111636897.png)

### 修正语法和拼写
![](https://image.aayu.today/uploads/2023/03/11/202303111643940.png)

### 自定义操作
![](https://image.aayu.today/uploads/2023/03/11/202303111644804.png)
![](https://image.aayu.today/uploads/2023/03/11/202303111645273.png)
{.gallery  data-height="240"}

## 总结
未来可期~
