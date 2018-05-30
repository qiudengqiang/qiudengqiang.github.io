---
title:  Xcode-打包证书创建流程
date: 2018-04-29 11:48:37
tags: [Xcode]
categories: Xcode
---

## 创建CSR证书
### 点击spotlight输入keychain打开钥匙串
![](http://p7xd6yrmx.bkt.clouddn.com/WX20180415-192128.ed09497c15d24f6bb84e812effb1d0f9.png)  
### 生成CSR文件

### **注意**：
![](http://p7xd6yrmx.bkt.clouddn.com/DraggedImage.ae2ad2036be3471b9b1549406a95e44b.png)

## 创建Cer证书
### [登录 apple developer](https://developer.apple.com/account/)

### 使用CSR文件创建Development和Distribution的CER证书并下载
![](http://p7xd6yrmx.bkt.clouddn.com/DraggedImage.ca56c2a73e474422ac1aaac410172bc6.png)

### 双击下载好的cer证书，然后导出对应的p12文件（dev/dis）
![](http://p7xd6yrmx.bkt.clouddn.com/DraggedImage.bc375b929a6f463f8d85d6f92e6bc853.png)

## 创建Provisioning Profile
这里要创建是三种profile（dev/dis/adhoc）
![](http://p7xd6yrmx.bkt.clouddn.com/DraggedImage.8b0bda299adf43029f16d552dceac353.png)
