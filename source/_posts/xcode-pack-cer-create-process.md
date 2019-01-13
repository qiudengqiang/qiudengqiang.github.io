---
title: Xcode - 打包证书创建流程
date: 2018-04-29 11:48:37
tags: [Xcode]
categories: Xcode
---

## 创建CSR证书
### 点击spotlight输入keychain打开钥匙串
![](https://github.com/qiudengqiang/blog-images/blob/master/csr_keychain.png)  
### 生成CSR文件
![](https://github.com/qiudengqiang/blog-images/blob/master/csr_create.png)  
### **注意**：
![](https://github.com/qiudengqiang/blog-images/blob/master/csr_alert.png)

## 创建Cer证书
### [登录 apple developer](https://developer.apple.com/account/)

### 使用CSR文件创建Development和Distribution的CER证书并下载
![](https://github.com/qiudengqiang/blog-images/blob/master/cer_create.png)

### 双击下载好的cer证书，然后导出对应的p12文件（dev/dis）
![](https://github.com/qiudengqiang/blog-images/blob/master/cer_export_p12.png)

## 创建Provisioning Profile
这里要创建是三种profile（dev/dis/adhoc）
![](https://github.com/qiudengqiang/blog-images/blob/master/cer_profile_list.png)
