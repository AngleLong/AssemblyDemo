#组件化开发
##参考资源
1. [Android组件化方案 ](http://blog.csdn.net/guiying712/article/details/55213884#1组件模式和集成模式的转换)

##为什么要组件化开发
###解决问题
- 实际业务变化非常快，但是单一工程的业务模块耦合度太高，牵一发而动全身； 
- 对工程所做的任何修改都必须要编译整个工程； 
- 功能测试和系统测试每次都要进行； 
- 团队协同开发存在较多的冲突.不得不花费更多的时间去沟通和协调，并且在开发过程中，任何一位成员没办法专注于自己的功能点，影响开发效率； 
- 不能灵活的对业务模块进行配置和组装；

###好处
- 加快业务迭代速度，各个业务模块组件更加独立，不再出现业务耦合情况； 
- 稳定的公共模块采用依赖库方式，提供给各个业务线使用，减少重复开发和维护工作量； 
- 迭代频繁的业务模块采用组件方式，各业务研发可以互不干扰、提升协作效率，并控制产品质量； 
- 为新业务随时集成提供了基础，所有业务可上可下，灵活多变； 
- 降低团队成员熟悉项目的成本，降低项目的维护难度； 
- 加快编译速度，提高开发效率； 
- 控制代码权限，将代码的权限细分到更小的粒度；

##具体实现方案
###1.组件模式和集成模式的转换
####android studio 中的Module主要又两种属性
- **application属性**,可以单独运行的Android程序,也就是我们的APP
    
    ```
     //这个是在build.gradle中进行设置的(就是最顶上那一行)
     apply plugin: 'com.android.application'
     ```
- **library属性**,不可以单独运行,一般是Android程序依赖的库文件
    ```
    //这个是在build.gradle中进行设置的(就是最顶上那一行)
    apply plugin: ‘com.android.library’
    ```
    
####组件化和集成模式的转换
说到这里我不想去讲解gradle的语法,因为我也不会,但是可以告诉你怎么去弄!!!

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)
> Module的属性是在每个组件的 build.gradle 文件中配置的，当我们在组件模式开发时，业务组件应处于application属性，这时的业务组件就是一个 Android App，可以独立开发和调试；而当我们转换到集成模式开发时，业务组件应该处于 library 属性，这样才能被我们的“app壳工程”所依赖，组成一个具有完整功能的APP；
但是我们如何让组件在这两种模式之间自动转换呢？总不能每次需要转换模式的时候去每个业务组件的 Gralde 文件中去手动把 Application 改成 library 吧？如果我们的项目只有两三个组件那么这个办法肯定是可行的，手动去改一遍也用不了多久，但是在大型项目中我们可能会有十几个业务组件，再去手动改一遍必定费时费力，这时候就需要程序员发挥下懒的本质了。
试想，我们经常在写代码的时候定义静态常量，那么定义静态常量的目的什么呢？当一个常量需要被好几处代码引用的时候，把这个常量定义为静态常量的好处是当这个常量的值需要改变时我们只需要改变静态常量的值，其他引用了这个静态常量的地方都会被改变，做到了一次改变，到处生效；根据这个思想，那么我们就可以在我们的代码中的某处定义一个决定业务组件属性的常量，然后让所有业务组件的build.gradle都引用这个常量，这样当我们改变了常量值的时候，所有引用了这个常量值的业务组件就会根据值的变化改变自己的属性；可是问题来了？静态常量是用Java代码定义的，而改变组件属性是需要在Gradle中定义的，Gradle能做到吗？
Gradle自动构建工具有一个重要属性，可以帮助我们完成这个事情。每当我们用AndroidStudio创建一个Android项目后，就会在项目的根目录中生成一个文件 gradle.properties，我们将使用这个文件的一个重要属性：在Android项目中的任何一个build.gradle文件中都可以把gradle.properties中的常量读取出来；那么我们在上面提到解决办法就有了实际行动的方法，首先我们在gradle.properties中定义一个常量值 isModule（是否是组件开发模式，true为是，false为否）：

#####在gradle.properties中添加是否组件化的判断
**PS:首先gradle.properties这个文件有一个重要的功能:在Android项目中的任何一个build.gradle文件中都可以把gradle.properties中的常量读取出来(但是读取出来的时候是String格式的字符串,这一点要切记)**

在gradle.properties中添加
```
# 是否组件化的标识,注意啊这里要用"#"号注释
isModule = false 
```
然后在相应的Module中的build.gradle去添加相应的逻辑
```
//这里说明一下就是如果是组件化的
if (isModule.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}
```
这样写完了之后就会在每次改变**isModule*赋值的化就可以更改相应的模块是否能单独运行了(为了保证单独组件的单独运行,会创建一个公用的Module,存放网络请求或者图片处理等一些公共的资源,从而确保module的单独运行)

####组件之间AndroidManifest合并问题
这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)
>但是大家应该注意到这个问题是在组件开发模式和集成开发模式之间转换引起的问题，而在上一节中我们已经解决了组件模式和集成模式转换的问题，另外大家应该都经历过将 Android 项目从 Eclipse 切换到 AndroidStudio 的过程，由于 Android 项目在 Eclipse 和 AndroidStudio开发时 AndroidManifest.xml 文件的位置是不一样的，我们需要在build.gradle 中指定下 AndroidManifest.xml 的位置，AndroidStudio 才能读取到 AndroidManifest.xml，这样解决办法也就有了，我们可以为组件开发模式下的业务组件再创建一个 AndroidManifest.xml，然后根据isModule指定AndroidManifest.xml的文件路径，让业务组件在集成模式和组件模式下使用不同的AndroidManifest.xml，这样表单冲突的问题就可以规避了。

#####创建一个新的AndroidManifest.xml
具体的解决方案就是在相应的Module中在根路径添加一个Module文件夹,直接创建一份AndroidManifest.xml,这里面的这个文件,直接像你平时写项目一样去写就可以了

#####修改相应的build.gradle
```
在android标签内创建
 sourceSets {
        main {
            if (isModule.toBoolean()) {/*是组件化*/
                manifest.srcFile 'src/main/AndroidManifest.xml'
                java {/*集成开发模式下排除debug文件加中的所有java文件*/
                    exclude 'debug/**'
                }
            } else {
                manifest.srcFile 'src/main/module/AndroidManifest.xml'
            }
        }
    }
```

####全局Context的获取及组件数据初始化
其实我理解的全局Context的理解就是->先创建一个所有module都引用的一个module(common这个module是每个module都会引用的一个module,这里要处理重复的问题,后面补充),这个module就存放一些都能用到的东西,然后引入到module或者主项目中,这样所有的module都会包含一些公共的内容:如请求网络,图片处理等一些相应的内容,这样既保证了每个module的单独运行,也保证了项目的耦合程度

这里有一个存在一个技巧,在前面没有去理解,就是在**组件之间AndroidManifest合并问题**处有这样一段代码
```
 java {/*集成开发模式下排除debug文件加中的所有java文件*/
                    exclude 'debug/**'
                }
```
这段代码的主要问题就是解决全局Context的问题,为什么呢??? 

我们想一下场景啊!当你某一个模块单独运行的时候,存在登陆获取token的问题,但是你要是单独运行的时候呢,可能这个模块没有token你怎么办呢?其实解决办法就是创建一个debug文件夹,这里呢可以自己创建一个相应的Application去请求登陆的接口这样你的module单独运行的时候就会有token了具体值
(这里我看张华洋的博客时我觉的处理挺好的,他在公共的类中去创建了一个BaseApplication所有module中的Application(在debug文件夹下创建的,但是要在清单文件中去配置啊!!!)都去继承这个Application,这样既解决了没有Application的情况,又解决了集成打包时候多次创建Application的情况,觉得比较不错所以借鉴一下)
上面那行代码会在集成开发模式的情况下把debug文件夹下的内容删除,这里就成功的解决了单独运行的token问题,也解决了集成打包的时候会出现多个Application的问题

####library依赖问题
解决library的重复引用问题又两种方式:
- 根据组件名称排除
- 根据包名排除
```
    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        compile("com.jude:easyrecyclerview:$rootProject.easyRecyclerVersion") {
            exclude module: 'support-v4'//根据组件名排除
            exclude group: 'android.support.v4'//根据包名排除
        }
    }
```
这里说明一点:基础的控件一般都在common中去引用,然后住项目和其他的项目都导入common就可以了,但是这里应该存在一个问题就是Common存在导入多次的问题
其实在构建APP的过程中Gradle会自动将重复的arr包排除，APP中也就不会存在相同的代码了；

####组件之间的调用和通信
这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)
> 在组件化开发的时候，组件之间是没有依赖关系，我们不能在使用显示调用来跳转页面了，因为我们组件化的目的之一就是解决模块间的强依赖问题，
假如现在要从A业务组件跳转到业务B组件，并且要携带参数跳转，这时候怎么办呢？
而且组件这么多怎么管理也是个问题，这时候就需要引入“路由”的概念了，由本文开始的组件化模型下的业务关系图可知路由就是起到一个转发的作用。
这里我将介绍开源库的“ActivityRouter” ，有兴趣的同学情直接去ActivityRouter的Github主页学习：ActivityRouter，ActivityRouter支持给Activity定义 URL，
这样就可以通过 URL 跳转到Activity，并且支持从浏览器以及 APP 中跳入我们的Activity，而且还支持通过 url 调用方法。
下面将介绍如何将ActivityRouter集成到组件化项目中以实现组件之间的调用；

这里涉及到路由的问题后续在进行补充

####组件之间资源名称冲突的问题
这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)
> 因为我们拆分出了很多业务组件和功能组件，在把这些组件合并到“app壳工程”时候就有可能会出现资源名冲突问题，
例如A组件和B组件都有一张叫做“ic_back”的图标，这时候在集成模式下打包APP就会编译出错，解决这个问题最简单的办法就是在项目中约定资源文件命名规约，
比如强制使每个资源文件的名称以组件名开始，这个可以根据实际情况和开发人员制定规则。当然了万能的Gradle构建工具也提供了解决方法，
通过在在组件的build.gradle中添加如下的代码：
    
    ```
    //设置了resourcePrefix值后，所有的资源名必须以指定的字符串做前缀，否则会报错。
    //但是resourcePrefix这个值只能限定xml里面的资源，并不能限定图片资源，所有图片资源仍然需要手动去修改资源名。
    resourcePrefix "girls_"
    ```
    
但是设置了这个属性后有个问题，所有的资源名必须以指定的字符串做前缀，否则会报错，
而且resourcePrefix这个值只能限定xml里面的资源，并不能限定图片资源，所有图片资源仍然需要手动去修改资源名；
所以我并不推荐使用这种方法来解决资源名冲突。

###2.组件化项目的工程类型
**在组件化工程模型中主要有：app壳工程、业务组件和功能组件3种类型，而业务组件中的Main组件和功能组件中的Common组件比较特殊，下面将分别介绍。**
####1.app壳工程
app壳工程相当于一个空壳的项目,没有任何业务代码,也不能又Activity,但是它又必须单独划分一个组件,而且不能融合其他组件中;
因为它又几点重要的功能:

- **app壳工程中声明了我们Android应用的Application** 但是这个Application必须是继承Common组件中的BaseApplication(如果无需事先自己的Application可以直接在表单生命BaseApplication),
因为只有这样,在打包应用后才能让BaseApplication中的Context生效,当然你也可以在这个Application中初始化我们工程中使用的库文件,还可以在这里面解决Android引用方法数超过65535的限制,
对于崩溃事件的捕获和发送也可以在这里面声明

- **app壳工程的AndroidManifest.xml是我们Android应用的根表单** 应用的名,图标依稀是否支持备份等等属性都是在这份表单中配置的,
其他组件中的表单最终在集成开发模式下都被合并到这份AndroidManifest.xml中

- **app壳工程的build.gradle是比较特殊的** app壳不管是在集成开发模式还是组件开发模式,它的属性始终都是:
com.android.application,因为最终其它的组件都要被app壳工程所依赖,被打包进app壳工程中,这一点从组件化工程模型图中就能体现出来,
所以app壳工程是不需要单独调试单独开发的.另外Android应用的打包签名,以及build.gradle和defaultConfig都需要在这里面配置,
而它的dependencies则需要根据isModule的值分别依赖不同的组件，在组件开发模式下app壳工程只需要依赖Common组件，
或者为了防止报错也可以根据实际情况依赖其他功能组件，而在集成模式下app壳工程必须依赖所有在应用Application中声明的业务组件，并且不需要再依赖任何功能组件。

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

一份**壳**工程的build.gradle文件
```
apply plugin: 'com.android.application'

static def buildTime() {
    return new Date().format("yyyyMMdd");
}

android {
    signingConfigs {
        release {
            keyAlias 'guiying712'
            keyPassword 'guiying712'
            storeFile file('/mykey.jks')
            storePassword 'guiying712'
        }
    }

    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion
    defaultConfig {
        applicationId "com.guiying.androidmodulepattern"
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
        multiDexEnabled false
        //打包时间
        resValue "string", "build_time", buildTime()
    }

    buildTypes {
        release {
            //更改AndroidManifest.xml中预先定义好占位符信息
            //manifestPlaceholders = [app_icon: "@drawable/icon"]
            // 不显示Log
            buildConfigField "boolean", "LEO_DEBUG", "false"
            //是否zip对齐
            zipAlignEnabled true
            // 缩减resource文件
            shrinkResources true
            //Proguard
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //签名
            signingConfig signingConfigs.release
        }

        debug {
            //给applicationId添加后缀“.debug”
            applicationIdSuffix ".debug"
            //manifestPlaceholders = [app_icon: "@drawable/launch_beta"]
            buildConfigField "boolean", "LOG_DEBUG", "true"
            zipAlignEnabled false
            shrinkResources false
            minifyEnabled false
            debuggable true
        }
    }


}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    annotationProcessor "com.github.mzule.activityrouter:compiler:$rootProject.annotationProcessor"
    if (isModule.toBoolean()) {
        compile project(':lib_common')
    } else {
        compile project(':module_main')
        compile project(':module_girls')
        compile project(':module_news')
    }
}
```

####2.功能组件和Common组件
功能组件是为了支撑业务组件的某些功能而独立划分出来的组件,功能组件和第三方库是一样的,
拥有如下特征:

#####功能组件
- 功能组件的AndroidManifest.xml是一张空表,这张表只有功能组件的包名;

- 功能组件不管是在集成开发还是组件开发模式下属性始终是:com.android.library,
所以功能组件不需要读取gradle.properties 中的isModule值的;另外功能组件的build.gradle也无需设置buildType是,
只需要dependencies这个共鞥组件需要的jar包和开源库.

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

一份 **普通** 的功能组件的 build.gradle文件：
```
apply plugin: 'com.android.library'

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
    }

}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
}
```
#####Common组件
Common组件除了有功能组件的普遍属性外，还具有其他功能：
- Common组件的 AndroidManifest.xml 不是一张空表，这张表中声明了我们 Android应用用到的所有使用权限 uses-permission 和 uses-feature，放到这里是因为在组件开发模式下，所有业务组件就无需在自己的 AndroidManifest.xm 声明自己要用到的权限了。
- Common组件的 build.gradle 需要统一依赖业务组件中用到的 第三方依赖库和jar包，例如我们用到的ActivityRouter、Okhttp等等。
- Common组件中封装了Android应用的 Base类和网络请求工具、图片加载工具等等，公用的 widget控件也应该放在Common 组件中；业务组件中都用到的数据也应放于Common组件中，例如保存到 SharedPreferences 和 DataBase 中的登陆数据；
- Common组件的资源文件中需要放置项目公用的 Drawable、layout、sting、dimen、color和style 等等，另外项目中的 Activity 主题必须定义在 Common中，方便和 BaseActivity 配合保持整个Android应用的界面风格统一。

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

一份 **Common** 的功能组件的 build.gradle文件：
```
apply plugin: 'com.android.library'

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
    }

}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    //Android Support
    compile "com.android.support:appcompat-v7:$rootProject.supportLibraryVersion"
    compile "com.android.support:design:$rootProject.supportLibraryVersion"
    compile "com.android.support:percent:$rootProject.supportLibraryVersion"
    //网络请求相关
    compile "com.squareup.retrofit2:retrofit:$rootProject.retrofitVersion"
    compile "com.squareup.retrofit2:retrofit-mock:$rootProject.retrofitVersion"
    compile "com.github.franmontiel:PersistentCookieJar:$rootProject.cookieVersion"
    //稳定的
    compile "com.github.bumptech.glide:glide:$rootProject.glideVersion"
    compile "com.orhanobut:logger:$rootProject.loggerVersion"
    compile "org.greenrobot:eventbus:$rootProject.eventbusVersion"
    compile "com.google.code.gson:gson:$rootProject.gsonVersion"
    compile "com.github.chrisbanes:PhotoView:$rootProject.photoViewVersion"

    compile "com.jude:easyrecyclerview:$rootProject.easyRecyclerVersion"
    compile "com.github.GrenderG:Toasty:$rootProject.toastyVersion"

    //router
    compile "com.github.mzule.activityrouter:activityrouter:$rootProject.routerVersion"
}
```
####3.业务组件和Main组件
#####业务组件
业务组件就是根据业务逻辑的不同拆分出来的组件，业务组件的特征如下：
- 业务组件中要有两张AndroidManifest.xml，分别对应组件开发模式和集成开发模式，这两张表的区别请查看 组件之间AndroidManifest合并问题 小节。
- 业务组件在集成模式下是不能有自己的Application的，但在组件开发模式下又必须实现自己的Application并且要继承自Common组件的BaseApplication，并且这个Application不能被业务组件中的代码引用，因为它的功能就是为了使业务组件从BaseApplication中获取的全局Context生效，还有初始化数据之用。
- 业务组件有debug文件夹，这个文件夹在集成模式下会从业务组件的代码中排除掉，所以debug文件夹中的类不能被业务组件强引用，例如组件模式下的 Application 就是置于这个文件夹中，还有组件模式下开发给目标 Activity 传递参数的用的 launch Activity 也应该置于 debug 文件夹中；
- 业务组件必须在自己的 Java文件夹中创建业务组件声明类，以使 app壳工程 中的 应用Application能够引用，实现组件跳转，具体请查看 组件之间调用和通信 小节；
- 业务组件必须在自己的 build.gradle 中根据 isModule 值的不同改变自己的属性，在组件模式下是：com.android.application，而在集成模式下com.android.library；同时还需要在build.gradle配置资源文件，如 指定不同开发模式下的AndroidManifest.xml文件路径，排除debug文件夹等；业务组件还必须在dependencies中依赖Common组件，并且引入ActivityRouter的注解处理器annotationProcessor，以及依赖其他用到的功能组件。

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

一份**普通**业务组件的 build.gradle文件：
```
if (isModule.toBoolean()) {
    apply plugin: 'com.android.application'
} else {
    apply plugin: 'com.android.library'
}

android {
    compileSdkVersion rootProject.ext.compileSdkVersion
    buildToolsVersion rootProject.ext.buildToolsVersion

    defaultConfig {
        minSdkVersion rootProject.ext.minSdkVersion
        targetSdkVersion rootProject.ext.targetSdkVersion
        versionCode rootProject.ext.versionCode
        versionName rootProject.ext.versionName
    }

    sourceSets {
        main {
            if (isModule.toBoolean()) {
                manifest.srcFile 'src/main/module/AndroidManifest.xml'
            } else {
                manifest.srcFile 'src/main/AndroidManifest.xml'
                //集成开发模式下排除debug文件夹中的所有Java文件
                java {
                    exclude 'debug/**'
                }
            }
        }
    }

    //设置了resourcePrefix值后，所有的资源名必须以指定的字符串做前缀，否则会报错。
    //但是resourcePrefix这个值只能限定xml里面的资源，并不能限定图片资源，所有图片资源仍然需要手动去修改资源名。
    //resourcePrefix "girls_"


}

dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    annotationProcessor "com.github.mzule.activityrouter:compiler:$rootProject.annotationProcessor"
    compile project(':lib_common')
}
```
#####Main组件

Main组件集成模式下的AndroidManifest.xml是跟其他业务组件不一样的，Main组件的表单中声明了我们整个Android应用的launch Activity，这就是Main组件的独特之处；所以我建议SplashActivity、登陆Activity以及主界面都应属于Main组件，也就是说Android应用启动后要调用的页面应置于Main组件。

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

一份**Main**业务组件的 build.gradle文件：

```
        <activity
            android:name=".splash.SplashActivity"
            android:launchMode="singleTop"
            android:screenOrientation="portrait"
            android:theme="@style/SplashTheme">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />

                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>
```
###3.组件化项目的混淆方案
**组件化项目的Java代码混淆方案采用在集成模式下集中在app壳工程中混淆，各个业务组件不配置混淆文件。** 集成开发模式下在app壳工程中build.gradle文件的release构建类型中开启混淆属性，其他buildTypes配置方案跟普通项目保持一致，Java混淆配置文件也放置在app壳工程中，各个业务组件的混淆配置规则都应该在app壳工程中的混淆配置文件中添加和修改。
之所以不采用在每个业务组件中开启混淆的方案，是因为 组件在集成模式下都被 Gradle 构建成了 release 类型的arr包，一旦业务组件的代码被混淆，而这时候代码中又出现了bug，将很难根据日志找出导致bug的原因；另外每个业务组件中都保留一份混淆配置文件非常不便于修改和管理，这也是不推荐在业务组件的 build.gradle 文件中配置 buildTypes （构建类型）的原因。

###4.工程的build.gradle和gradle.properties文件
####组件化工程的build.gradle文件

在组件化项目中因为每个组件的 build.gradle 都需要配置 compileSdkVersion、buildToolsVersion和defaultConfig 等的版本号，而且每个组件都需要用到 annotationProcessor，为了能够使组件化项目中的所有组件的 build.gradle 中的这些配置都能保持统一，并且也是为了方便修改版本号，我们统一在Android工程根目录下的build.gradle中定义这些版本号，当然为了方便管理Common组件中的第三方开源库的版本号，最好也在这里定义这些开源库的版本号，然后在各个组件的build.gradle中引用Android工程根目录下的build.gradle定义的版本号，组件化工程的 build.gradle 文件代码如下：

这里是引用张华洋的说明:[博客地址](http://blog.csdn.net/guiying712)

```
buildscript {
    repositories {
        jcenter()
        mavenCentral()
    }

    dependencies {
        //classpath "com.android.tools.build:gradle:$localGradlePluginVersion"
        //$localGradlePluginVersion是gradle.properties中的数据
        classpath "com.android.tools.build:gradle:$localGradlePluginVersion"
    }
}

allprojects {
    repositories {
        jcenter()
        mavenCentral()
        //Add the JitPack repository
        maven { url "https://jitpack.io" }
        //支持arr包
        flatDir {
            dirs 'libs'
        }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

// Define versions in a single place
//时间：2017.2.13；每次修改版本号都要添加修改时间
ext {
    // Sdk and tools
    //localBuildToolsVersion是gradle.properties中的数据
    buildToolsVersion = localBuildToolsVersion
    compileSdkVersion = 25
    minSdkVersion = 16
    targetSdkVersion = 25
    versionCode = 1
    versionName = "1.0"
    javaVersion = JavaVersion.VERSION_1_8

    // App dependencies version
    supportLibraryVersion = "25.3.1"
    retrofitVersion = "2.1.0"
    glideVersion = "3.7.0"
    loggerVersion = "1.15"
    eventbusVersion = "3.0.0"
    gsonVersion = "2.8.0"
    photoViewVersion = "2.0.0"

    //需检查升级版本
    annotationProcessor = "1.1.7"
    routerVersion = "1.2.2"
    easyRecyclerVersion = "4.4.0"
    cookieVersion = "v1.0.1"
    toastyVersion = "1.1.3"
}
```

####组件化工程的gradle.properties文件
在组件化实施流程中我们了解到gradle.properties有两个属性对我们非常有用：

- 在Android项目中的任何一个build.gradle文件中都可以把gradle.properties中的常量读取出来，不管这个build.gradle是组件的还是整个项目工程的build.gradle；
- gradle.properties中的数据类型都是String类型，使用其他数据类型需要自行转换；

利用gradle.properties的属性不仅可以解决集成开发模式和组件开发模式的转换，而且还可以解决在多人协同开发Android项目的时候，因为开发团队成员的Android开发环境（开发环境指Android SDK和AndroidStudio）不一致而导致频繁改变线上项目的build.gradle配置。

在每个Android组件的 build.gradle 中有一个属性：buildToolsVersion，表示构建工具的版本号，这个属性值对应 AndroidSDK 中的 Android SDK Build-tools，正常情况下 build.gradle 中的 buildToolsVersion 跟你电脑中 Android SDK Build-tools 的最新版本是一致的，比如现在 Android SDK Build-tools 的最新的版本是：25.0.3，那么我的Android项目中 build.gradle 中的 buildToolsVersion 版本号也是 25.0.3，但是一旦一个Android项目是由好几个人同时开发，总会出现每个人的开发环境 Android SDK Build-tools 是都是不一样的，并不是所有人都会经常升级更新 Android SDK，而且代码是保存到线上环境的（例如使用 SVN/Git 等工具），某个开发人员提交代码后线上Android项目中 build.gradle 中的 buildToolsVersion 也会被不断地改变。 

另外一个原因是因为Android工程的根目录下的 build.gradle 声明了 Android Gradle 构建工具，而这个工具也是有版本号的，而且 Gradle Build Tools 的版本号跟 AndroidStudio 版本号一致的，但是有些开发人员基本很久都不会升级自己的 AndroidStudio 版本，导致团队中每个开发人员的 Gradle Build Tools 的版本号也不一致。

如果每次同步代码后这两个工具的版本号被改变了，开发人员可以自己手动改回来，并且不要把改动工具版本号的代码提交到线上环境，这样还可以勉强继续开发；但是很多公司都会使用持续集成工具（例如Jenkins）用于持续的软件版本发布，而Android出包是需要 Android SDK Build-tools 和 Gradle Build Tools 配合的，一旦提交到线上的版本跟持续集成工具所依赖的Android环境构建工具版本号不一致就会导致Android打包失败。

为了解决上面问题就必须将Android项目中 build.gradle 中的 buildToolsVersion 和 GradleBuildTools 版本号从线上代码隔离出来，保证线上代码的 buildToolsVersion 和 Gradle Build Tools 版本号不会被人为改变。

具体的实施流程大家可以查看我的这篇博文： [AndroidStudio本地化配置gradle的buildToolsVersion和gradleBuildTools](http://blog.csdn.net/guiying712/article/details/72629948)


上面这些要感谢章华洋的博客,基本是按照他的写法自己写了一遍,体会了一下,里面的只是还是很多了,仅作备份,如果楼主有意见,马上删除!!!