# Android平台签名证书(.keystore)生成指南

### 1、安装JRE环境（推荐使用JRE8环境，如已有可跳过）

[生成指南](https://ask.dcloud.net.cn/article/35777)

### 2、生成签名证书

```bash
# 使用keytool -genkey命令生成证书：
keytool -genkey -alias testalias -keyalg RSA -keysize 2048 -validity 36500 -keystore test.keystore

# testalias是证书别名，可修改为自己想设置的字符，建议使用英文字母和数字
# test.keystore是证书文件名称，可修改为自己想设置的文件名称，也可以指定完整文件路径
# 36500是证书的有效期，表示100年有效期，单位天，建议时间设置长一点，避免证书过期

Enter keystore password:  //输入证书文件密码，输入完成回车  
Re-enter new password:   //再次输入证书文件密码，输入完成回车  
What is your first and last name?  
  [Unknown]:  //输入名字和姓氏，输入完成回车  
What is the name of your organizational unit?  
  [Unknown]:  //输入组织单位名称，输入完成回车  
What is the name of your organization?  
  [Unknown]:  //输入组织名称，输入完成回车  
What is the name of your City or Locality?  
  [Unknown]:  //输入城市或区域名称，输入完成回车  
What is the name of your State or Province?  
  [Unknown]:  //输入省/市/自治区名称，输入完成回车  
What is the two-letter country code for this unit?  
  [Unknown]:  //输入国家/地区代号（两个字母），中国为CN，输入完成回车  
Is CN=XX, OU=XX, O=XX, L=XX, ST=XX, C=XX correct?  
  [no]:  //确认上面输入的内容是否正确，输入y，回车  

Enter key password for <testalias>  
        (RETURN if same as keystore password):  //确认证书密码与证书文件密码一样（HBuilder|HBuilderX要求这两个密码一致），直接回车就可以
```



### 3、查看证书信息

```bash
keytool -list -v -keystore test.keystore  
Enter keystore password: //输入密码，回车
```

