### Android Studio设置Gradle代理的正确姿势

```
在项目中的gradle.propertities文件中加入下面两行

systemProp.https.proxyHost=127.0.0.1
systemProp.https.proxyPort=8123
```