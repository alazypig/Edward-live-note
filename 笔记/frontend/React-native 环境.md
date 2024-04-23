### 安装 iOS依赖
```bash
pod install --verbose --no-repo-update
cocopads 最好和项目中的版本一致
```

### 打包安卓release版本
```bash
cd android
# build google release
./gradlew assembleGoogleRelease
# pwd build file
app/build/outputs/apk/google/release/

# build google aab
./gradlew bundleGoogleRelease
# pwb google aab
app/build/outputs/bundle/googleRelease/

# run
cd ..
npx react-native run-android --variant=release
```

###  android gradle版本设置
```txt
android studio -> 右上角setting -> project structure -> project -> Gradle Version 选择项目中对应版本
```

### SDK版本设置
```txt
setting -> build tools -> gradle -> gradle project -> gradle JDK
使用项目对应的版本
```

### gradle更新依赖
```txt
file -> sync project with gradle files
需要连接国际互联网
```