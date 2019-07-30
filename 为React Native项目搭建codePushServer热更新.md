##  为React Native项目搭建codePushServer热更新

-------

    React Native的热更新是一个很酷的功能，可以实时更新你的应用，不需要经过打包，可以直接推送代码进行实时更新，完全避免让用户退出应用下载安装的尴尬，让应用有更多的可确定性。
 
-------

[具体流程]()

#### Step1. 配置相关环境

#### Step2.  注册CodePushServer 账号

#### Step3. 在CodePushServer 注册app

#### Step4. 在Xcode与AndroidStudio中修改develpment key

#### Step5. 发布一个应用更新到服务器

#### Step6. app收到升级推送



###  一、 配置相关环境
* 1.安装MySql（以下基于brew安装，也可以通过下载安装包进行安装）
    ·打开终端输入：`mysql：brew install mysql `
    ·配置与初始化：`mysql_secure_installation`
    
```shell
$ mysql_secure_installation
 
Securing the MySQL server deployment.
 
Connecting to MySQL using a blank password.
 
VALIDATE PASSWORD PLUGIN can be used to test passwords
and improve security. It checks the strength of password
and allows the users to set only those passwords which are
secure enough. Would you like to setup VALIDATE PASSWORD plugin?
 // 这个选yes的话密码长度就必须要设置为8位以上，但我只想要6位的
Press y|Y for Yes, any other key for No: N   
Please set the password for root here.
 
New password:            // 设置密码
 
Re-enter new password:     // 再一次确认密码
By default, a MySQL installation has an anonymous user,
allowing anyone to log into MySQL without having to have
a user account created for them. This is intended only for
testing, and to make the installation go a bit smoother.
You should remove them before moving into a production
environment.
 // 移除不用密码的那个账户
Remove anonymous users? (Press y|Y for Yes, any other key for No) : Y    
Success.
 
 
Normally, root should only be allowed to connect from
'localhost'. This ensures that someone cannot guess at
the root password from the network.
 //是否禁止远程登录
Disallow root login remotely? (Press y|Y for Yes, any other key for No) : n
 
 ... skipping.
By default, MySQL comes with a database named 'test' that
anyone can access. This is also intended only for testing,
and should be removed before moving into a production
environment.
 
 // 是否删除test库
Remove test database and access to it? (Press y|Y for Yes, any other key for No) : y
 - Dropping test database...
Success.
 
 - Removing privileges on test database...
Success.
 
Reloading the privilege tables will ensure that all changes
made so far will take effect immediately.
 
Reload privilege tables now? (Press y|Y for Yes, any other key for No) : y
Success.
 
All done!
```
######                                 ·常见Mysql命令
    登陆mysql：`mysql -u root -p`
    启动mysql：`brew services start mysql@5.7`
    停止mysql：`brew services stop mysql@5.7`

*  输入 npm install -g code-push-cli 即可。
   安装完成后输入  code-push -v 查看当前版本，我的版本是 2.1.9。
*  安装code-push-server
    
  1.clone代码  `git clone https://github.com/lisong/code-push-server.git`  
  2.cd进入项目并安装 `cd code-push-server && npm install`
  3.修改配置文件 config/config.js 文件
  
```js
db: {
    username: process.env.RDS_USERNAME || "root",
    password: process.env.RDS_PASSWORD || "123456",//你的MySQL访问密码
    database: process.env.DATA_BASE || "codepush",//如果你init的时候指定了数据库名字的话，也需要改
    host: process.env.RDS_HOST || "127.0.0.1",
    port: process.env.RDS_PORT || 3306,
    dialect: "mysql",
    logging: false,
    operatorsAliases: false,
  },
  qiniu：{//七牛云的云服务相关配置
    accessKey: "", //个人面板 > 秘钥管理 > AK
    secretKey: "",//个人面板 > 秘钥管理 > SK
    bucketName: "",//"faweapp",//储存空间名称
    downloadUrl: "" //自定义域名
   },
   upyun: {//又拍云的云服务相关配置
   ......
  },
  s3: {//亚马逊的云服务相关配置
  ......
},
 oss: {//阿里云的云服务相关配置
   accessKeyId: "",
    secretAccessKey: "",
    endpoint: "https://oss-cn-qingdao.aliyuncs.com",
    bucketName: "mybucket",//储存空间名称
    prefix: "storage", // 目录文件夹名称
    downloadUrl: "http://mybucket.oss-cn-qingdao.aliyuncs.com/storage", //下载地址
    
 },
  tencentcloud: {//腾讯云的云服务相关配置
  ......
},
  // Config for local storage when storageType value is "local".
  local: {//本地的服务配置-->如果搭建本地服务器则必须配置 storageDir与            downloadUrl
  storageDir: process.env.STORAGE_DIR || "/Users/dev02/Desktop/working/demo/pushServer/storage",//
  downloadUrl: process.env.LOCAL_DOWNLOAD_URL || "http://127.0.0.1:3000/download",
   public: '/download'
  },
```
4.初始化数据库 
  ` ./bin/db init --dbhost localhost --dbuser root --dbpassword 123456 --dbname codepush`
  
### 二、安装和注册CodePushServer 账号
* 1.下载配置codePush

 1️⃣code-push管理  输入命令 code-push register http://api.code-push.com #注册一个CodepushServer 账号（登陆成功后一定复制保存好弹出的Token）

 2️⃣本地版本管理配置 ：cd 到CodePushServer目录，执行 `./bin/www`开启本地服务
 code-push login http://127.0.0.1:3000
 登陆成功后复制好Token粘贴到终端
 ![](https://upload-images.jianshu.io/upload_images/1319536-85e5e09cd73abcc5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/585/format/webp)
 
 成功登陆后，你的session文件将会写在 /Users/guanMac/.code-push.config
 
 #####               有关CodePush账户相关的命令

``` shell
code-push login 登陆
code-push logout 注销
code-push access-key ls 列出登陆的token
code-push access-key rm <accessKye> 删除某个 access-key
```
* 2.创建或使用已有RN项目

```shell * 
    cd ./ 存放项目目录
    react-native init TestRNUpdateDemo
    cd ./TestRNUpdateDemo
    npm install --save react-native-code-push  #安装react-native-code-push
    react-native link react-native-code-push   #与项目连接，提示输入配置就先忽略吧
```

-------
或者直接clone此demo [点我](https://github.com/lisong/code-push-demo-app)
 
-------


### 3.在CodePushServer 注册app
 由于CodePush的iOS与Android的双系统是分开管理热更新的，那么为了区分这两个平台的版本管理，将在应用后面板添加 <-platform>

```shell  code-push app add 工程名-android #android版
  code-push app add 工程名-ios #ios版
Successfully added the "TestRNUpdateDemo" app, along with the   following default deployments:
┌────────────┬──────────────────────────────────  ─────┐
│ Name       │ Deployment Key                        │
├────────────┼──────────────────────────────────  ─────┤
│ Production │ Praagnzym0XuijhvH1LRNSdK4kPO4ksvOXqog │
├────────────┼──────────────────────────────────  ─────┤
│ Staging    │ jcgp79y1MT8YSZzPmfs60LbY8vYP4ksvOXqog │
└────────────┴──────────────────────────────────  ─────┘
#分别获得iOS和安卓的Deployment Key，推送的时候通过key将app和服务器端关联，推送是分开的

```
### 常见CodePush命令
 

```shell
 code-push app list 可以查看创建好的APP
 code-push deployment ls  APP_NAME  -k 查看Deployment Key
 code-push app add 在账号里面添加一个新的app
 code-push app remove 或者 rm 在账号里移除一个app
 code-push app rename 重命名一个存在app
 code-push app list 或则 ls 列出账号下面的所有app
 code-push app transfer 把app的所有权转移到另外一个账号
//查看历史版本
code-push deployment history APP_NAME Staging/Production
  
//清空历史版本
code-push deployment clear APP_NAME Staging
//查看key
code-push deployment ls APP_NAME -k
```
### 4.在Xcode与AndroidStudio中修改develpment key
* Android添加Deployment Key

```java
 在MainApplication.java文件中
@Override
protected List<ReactPackage> getPackages() {
  return Arrays.<ReactPackage>asList(
      new MainReactPackage(),
      new CodePush(
       "填入Deployment Key ",
       MainApplication.this,
       BuildConfig.DEBUG,
       "填入你的电脑IP:eg. http://192.168.0.7:3000" 
    )
);
}
```

* iOS添加Deployment Key

```XML
在info.plist中添加key
<key>CodePushDeploymentKey</key>
<string>Deployment Key</string>
<key>CodePushServerURL</key>
<string>填入你的电脑IP:eg. http://192.168.0.7:3000</string>
 



```
 
*  替换index.js文件
 
```js

/**
 * Sample React Native App
 * https://github.com/facebook/react-native
 * @flow
 */

import React, { Component } from 'react';
import {
  AppRegistry,
  Dimensions,
  Image,
  StyleSheet,
  Text,
  TouchableOpacity,
  Platform,
  View,
} from 'react-native';

import CodePush from "react-native-code-push";

//var Dimensions = require('Dimensions');

const instructions = Platform.select({
  ios: 'Press Cmd+R to reload,\n' +
    'Cmd+D or shake for dev menu',
  android: 'Double tap R on your keyboard to reload,\n' +
    'Shake or press menu button for dev menu',
});

export default class App extends Component<{}> {

  constructor() {
    super();
    this.state = { restartAllowed: true ,
                   syncMessage: "我是小更新" ,
                   progress: false};
  }

  // 监听更新状态
  codePushStatusDidChange(syncStatus) {
    switch(syncStatus) {
      case CodePush.SyncStatus.CHECKING_FOR_UPDATE:
        this.setState({ syncMessage: "检查更新." });
        break;
      case CodePush.SyncStatus.DOWNLOADING_PACKAGE:
        this.setState({ syncMessage: "正在下载安装包." });
        break;
      case CodePush.SyncStatus.AWAITING_USER_ACTION:
        this.setState({ syncMessage: "等待用户操作." });
        break;
      case CodePush.SyncStatus.INSTALLING_UPDATE:
        this.setState({ syncMessage: "安装更新." });
        break;
      case CodePush.SyncStatus.UP_TO_DATE:
        this.setState({ syncMessage: "目前已经是最新版本.", progress: false });
        break;
      case CodePush.SyncStatus.UPDATE_IGNORED:
        this.setState({ syncMessage: "用户取消更新.", progress: false });
        break;
      case CodePush.SyncStatus.UPDATE_INSTALLED:
        this.setState({ syncMessage: "更新完成.", progress: false });
        break;
      case CodePush.SyncStatus.UNKNOWN_ERROR:
        this.setState({ syncMessage: "发生错误.", progress: false });
        break;
    }
  }


  codePushDownloadDidProgress(progress) {
    this.setState({ progress });
  }

  // 允许重启后更新
  toggleAllowRestart() {
    this.state.restartAllowed
      ? CodePush.disallowRestart()
      : CodePush.allowRestart();

    this.setState({ restartAllowed: !this.state.restartAllowed });
  }

  // 获取更新数据
  getUpdateMetadata() {
    CodePush.getUpdateMetadata(CodePush.UpdateState.RUNNING)
      .then((metadata: LocalPackage) => {
        this.setState({ syncMessage: metadata ? JSON.stringify(metadata) : "Running binary version", progress: false });
      }, (error: any) => {
        this.setState({ syncMessage: "Error: " + error, progress: false });
      });
  }

  /** Update is downloaded silently, and applied on restart (recommended) 自动更新，一键操作 */
  sync() {
    CodePush.sync(
      {},
      this.codePushStatusDidChange.bind(this),
      this.codePushDownloadDidProgress.bind(this)
    );
  }

  /** Update pops a confirmation dialog, and then immediately reboots the app 一键更新，加入的配置项 */
  syncImmediate() {
    CodePush.sync(
      { installMode: CodePush.InstallMode.IMMEDIATE, updateDialog: true },
      this.codePushStatusDidChange.bind(this),
      this.codePushDownloadDidProgress.bind(this)
    );
  }

  render() {

   let progressView;

    if (this.state.progress) {
      progressView = (
        <Text style={styles.messages}>{this.state.progress.receivedBytes} of {this.state.progress.totalBytes} bytes received</Text>
      );
    }

    return (
      <View style={styles.container}>

        <Text style={styles.welcome}>
         可以修改此处文字，查看是否更新成功！
        </Text>

        <TouchableOpacity onPress={this.sync.bind(this)}>
          <Text style={styles.syncButton}>Press for background sync</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={this.syncImmediate.bind(this)}>
          <Text style={styles.syncButton}>Press for dialog-driven sync</Text>
        </TouchableOpacity>

        {progressView}
        
        <TouchableOpacity onPress={this.toggleAllowRestart.bind(this)}>
          <Text style={styles.restartToggleButton}>Restart { this.state.restartAllowed ? "allowed" : "forbidden"}</Text>
        </TouchableOpacity>

        <TouchableOpacity onPress={this.getUpdateMetadata.bind(this)}>
          <Text style={styles.syncButton}>Press for Update Metadata</Text>
        </TouchableOpacity>

        <Text style={styles.messages}>{this.state.syncMessage || ""}</Text>

      </View>
    );
  }
}

const styles = StyleSheet.create({
  container: {
    flex: 1,
    alignItems: "center",
    backgroundColor: "#F5FCFF",
    paddingTop: 50
  },
  image: {
    margin: 30,
    width: Dimensions.get("window").width - 100,
    height: 365 * (Dimensions.get("window").width - 100) / 651,
  },
  messages: {
    marginTop: 30,
    textAlign: "center",
  },
  restartToggleButton: {
    color: "blue",
    fontSize: 17
  },
  syncButton: {
    color: "green",
    fontSize: 17
  },
  welcome: {
    fontSize: 20,
    textAlign: "center",
    margin: 20
  },
});


```

### 5.  发布一个应用更新到服务器

打包bundle结束后，就可以通过CodePush发布更新了。在控制台输入
` 
code-push release<应用名称> <Bundles所在目录> <对应的应用版本>
	 --deploymentName 更新环境
	 --description 更新描述
	 --mandatory 是否强制更新 `
	 
	 
	 
	 
	 #####常见其他更新命令
	 CodePush还可以进行很多种更新控制
### 添加一个更新log
`code-push release-react MyApp-iOS ios -m --description "添加了注册功能"`

### 只打包一个js文件
`code-push release-react MyApp-iOS ios --entryFile MyApp.js --sourcemapOutput ../maps/MyApp.map`

### 只更新25%的用户
`code-push release-react MyApp-Android android --rollout 25% --dev true
`
### 只更新1.0版本的客户端
`code-push release-react MyApp-Android android --targetBinaryVersion "~1.0.0"`
	 
	 
##### 注意
1. CodePush默认是更新 staging 环境的，如果是staging，则不需要填写 deploymentName。
2. mandatory为true时强制，客户端会更新，默认false 不强制更新。
3. targetBinaryVersion  版本号指的是当前需要更新的版本，比如客户端安装的是1.0.0版本，targetBinaryVersion填写为1.0.0，那么只有1.0客户端会更新此版本，不要误解。
4. 版本号最好为三位数，如1.0.0，两位数可能会报错
5. 如果本次推送的js与上个版本的js没有变化，推送成功则会报错。
 ![](https://raw.githubusercontent.com/crazycodeboy/RNStudyNotes/master/React%20Native%E5%BA%94%E7%94%A8%E9%83%A8%E7%BD%B2%E3%80%81%E7%83%AD%E6%9B%B4%E6%96%B0-CodePush%E6%9C%80%E6%96%B0%E9%9B%86%E6%88%90%E6%80%BB%E7%BB%93/images/%E5%AF%B9%E5%BA%94%E4%B8%80%E4%B8%AA%E7%89%88%E6%9C%AC%E6%9C%89%E4%B8%A4%E4%B8%AAbundle%E7%9A%84md5%E5%AE%8C%E5%85%A8%E4%B8%80%E6%A0%B7.png)
 
 

 
