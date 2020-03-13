# react-native-platform-compatibility-issues
记录一些工作中遇到的兼容性问题

### Android
#### 1、如果无设置具体宽度，部分安卓手机会出现文字超出容器情况。

      lgNormalText: {
        flex: 1,
        flexWrap: 'wrap',
        fontSize: S(28),
      }

解决方法：

给Text或Text容器设置具体宽度，不能使用flex：1

      lgNormalText: {
        width: S(508),
        flexWrap: 'wrap',
        fontSize: S(28),
      }



#### 2、安卓上图片地址无改变时，图片不会刷新。

解决方法：

- 后台返回新图片地址需是唯一的
- 前端给图片地址参数加上时间戳

const newImageUri = `https://www.image.com?a=b&timestamp=${new Date().getTime()}`


#### 3、官方Switch组件新*trackColor*属性在部分安卓无效

    <Switch
      onValueChange={onChange}
      value={isChecked}
      style={styles.switchStyle}
      onTintColor={ButtonColor.SWITCH_BACKGROUND_GREEN} // 忽略警告使用trackColor替换，在安卓无效，暂不替换。
    />

解决方法：使用onTintColor属性



#### 4、TextInput在安卓上默认有padding，IOS没有

解决方法：安卓上手动将TextInput的padding设置为0



#### 5、在render函数中渲染沉浸式状态栏时，在安卓上总是被原生的状态栏样式渲染覆盖

    <SafeAreaContainer>
      <KsNavigation navigation={this.props.navigation} navigationTitle={'通知'} />
      <StatusBar hidden={false} backgroundColor={'#fff'} />
    </SafeAreaContainer>

解决方法：在组件声明周期componentDidMount中使用StatusBar方法改变状态栏

    componentDidMount() {
      StatusBar.setTranslucent(true);
      StatusBar.setBackgroundColor('transparent');
      StatusBar.setBarStyle('light-content');
    }



#### 6、WebView加载本地静态HTML文件时，需要加上baseUrl: ''，否则部分安卓机型会出现文字乱码

    <WebView
      source={{ html: pointsProductInfo.appIntroduce, baseUrl: '' }}
    />



#### 7、人民币符号明文使用‘¥’，在部分安卓机型展示会有问题

    <Text>¥</Text>

解决方法：

    <Text>&yen;</Text>



#### 8、Android键盘会把底部的元素顶上去,如下，KsButton会被顶上去

解决方法：Keyboard监听键盘显示/隐藏，动态控制KsButton的隐藏/显示

    <View>
	  <FlatList data={data} renderItem={this.renderItem} />
      <KsButton text="保存" onPress={this.onPress} />
    </View>

    if (isAndroid) {
      this.keyboardDidShowListener = Keyboard.addListener('keyboardDidShow', () => {});
  	  this.keyboardDidHideListener = Keyboard.addListener('keyboardDidHide', () => {});
	}



#### 9、ES6、ES7...等js原生提供的方法，在不同的机型不同版本上支持不同，如flat在iOS12以下不支持，会造成闪退

解决方法：

1. 在[Can I use](https://www.caniuse.com/)上查询是否支持使用

2. 若有兼容性问题，可使用lodash提供的方法替换



#### 10、Image组件在设置了resizeMode后设置borderRadius在android上无效

	<Image
  	  source={image.icon}
      style={{ borderRadius: 10 }}
      resizeMode="stretch"
	/>

解决方法：borderRadius设置在父组件View上

	<View style={{ borderRadius: 10 }}>
  	  <Image
        source={image.icon}
        resizeMode="stretch"
  	  />
	</View>



#### 11、文字设置了加粗后需要设置字体，否则部分Android机型上可能会有不同的表现

解决方法：增加fontFamily: 'System'

	title: {
  	  fontWeight: 'bold',
  	  fontFamily: 'System',
	}



#### 12、Text显示文字时，如果有段中有英文、中文、标点符号时，换行规则：

    1） Text在显示中文的时候 标点符号不能显示在一行的行首和行尾，如果一个标点符号刚好在一行的行尾，该标点符号就会连同前一个字符跳到下一行显示；

    2）一个英文单词不能被显示在两行中（ Text在显示英文时，标点符号是可以放在行尾的，但英文单词也不能分开 ）；

    3）全角和半角的问题，汉字无论全角还是半角都是占2个字节，英文和符号在半角是占一个字节，全角是占两个字节。


解决方法：参考https://www.iteye.com/blog/niufc-1729792 这篇博客

### 13、ImageBackground组件设置style样式borderRadius在Android 上无效果

解决方法：给ImageBackground组件属性imageStyle上设置borderRadius

	<ImageBackground
	   imageStyle={{ borderRadius: xxx }}	
  	>...</ImageBackground>
	
### 14、在react-native ^0.61.1版本上给View设置borderRadius在Android上无效果

解决方法： 给需要设置borderRadius的View组件嵌套在一个带有backgroundColor属性的View组件里

	<View>
	   style={{
	   	...
		backgroundColor: 'rgba(0,0,0,.005)'
	   }}
  	>
	  <View style={{
	  	...
		borderRadius: ...
	  }}/>
	</View>

	
### IOS

#### 1、在WebView中，IOS获取window.postMessage传递参数时，在onMessage中解析参数需使用双重decodeURIComponent解码

	const injectedJavaScript = `
	(function () {
  	  var height = null;
  	  function changeHeight() {
    	if (document.body.scrollHeight != height) {
      	  height = document.body.scrollHeight;
          if (window.postMessage) {
            window.postMessage(JSON.stringify({
              type: 'setHeight',
              data: height,
            }))
          }
        }
  	  }
  	  setTimeout(changeHeight, 500);
	}());`;

	<WebView
      originWhitelist={['*']}
      source={{ html: pointsProductInfo.appIntroduce, baseUrl: '' }}
      onMessage={(event) => {
        try {
      	  const eventData = isIOS() ? decodeURIComponent(decodeURIComponent(event.nativeEvent.data)) : event.nativeEvent.data;
      	  const action = JSON.parse(eventData);
      	  const { type, data } = action;
      	  if (Object.prototype.hasOwnProperty.call(this.OnMessageActions, type)) {
            this.OnMessageActions[type](data);
      	  }
      	} catch (error) {
      	  console.error('ShowWebViewPage onMessage错误', error);
      	}
  	  }}
	/>



#### 2、微信 会屏蔽分享的红包 元等文字 分享的图片不能超过32K



