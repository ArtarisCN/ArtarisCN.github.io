---
layout: post
title: OMAndroid
date: 2018-06-13 12:21:21
tags: [OMAndroid]
categroies:
- OMAndroid
---

# OMAndroid
## 概述
OMAndroid 是一个整合了如下主流开源项目的 Android 快速开发框架，其中包括：
- Dagger
- Retrofit  + OkHttp
- RxJava
- ButterKnife
- SqlBrite
- EventBus

<!-- more -->

采用本框架即可使用方便易用的 MVP & Dagger 2 & Retrofit & RxJava 2 的
快速开发框架，只需设置简单的参数即可使用。
## 特点
- 包含全局的 Base 基类(`BaseApplication`、`BaseActvity`、`BaseFragment`)；
- 使用 MVP 架构，使用伴随 Application 全局生命周期的 DataManager 来充当 Model 层，可在不同页面共享数据；
- 使用注入配置，在不修改框架源码的情况下， 注入 Retrofit,DataBase 等 Config，并且易于扩展；
- 底层使用 Dagger 连接；
- 使用多个 Delegate 管理 Application、Activity、Fragment 的生命周期；
- 使用 ActivityManager 管理所有 Activity，提供方法进行跳转，关闭、存活判断；
- 还在不断迭代中；
## 框架结构
- 结构示意图
//todo
- Module 依赖
	- OMAndroid
		- 示意图
		//todo
		- 包结构
		//todo
	- OMView
		- 控件介绍
	- OMDataBase
## 框架使用
### 导入框架

[image:36B64C07-1B86-42CD-93D5-07F12C780B04-5012-0000083442BB302E/0C8288FF-B8E8-40B9-9638-D33605E0D5C9.png]
在 Android Studio 中导入框架 `library`及 ` OMView`
本框架使用了 Lambda 表达式及 Library 中的 ButterKnife 支持，在 Project 的`build.gradle`中添加如下
```
dependencies {
    classpath 'com.android.tools.build:gradle:3.1.4'
    classpath 'me.tatarka:gradle-retrolambda:3.7.0'
    classpath 'com.jakewharton:butterknife-gradle-plugin:8.4.0'
}
```
依赖库设置为：
```
repositories {
    google()
    mavenCentral()
    jcenter()
    maven { url "https://jitpack.io" }
}
```
如果你也想使用 Lambda 表达式，在 `app` Module 中加入如下代码即可
```
android {
    ...
    ...
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
	  ...
    ...
}

```
最后在 `app` Module 中加入对`library`的依赖即可
```
android {
    ...
    ...
    implementation project(':omview')
	  implementation project(':library')
	  ...
    ...
}

```
### 配置`Build.gradle`
本框架使用 Dagger 2 依赖，必须配置Dagger 2方可使用
```
dependencies {
    ...
    /**
     * dagger
     */
    implementation 'com.google.dagger:dagger:2.15'
    annotationProcessor 'com.google.dagger:dagger-compiler:
	  ...
}

```
### 开始使用
1. 配置 Application
使用本框架需要继承 OMBaseApplication 并在其 `provideApplicationConfig()`方法中配置使用中的 Retrofit、数据库等配置。
如例子中所示
```
@Override
public IOMApplicationConfig provideApplicationConfig() {
    return new ApplicationConfiguration();
}
```
编写一个配置文件继承`IOMApplicationConfig `来配置框架用到的参数，在
```
void appleOptions(Context context, ConfigModule.Builder builder)
```
中设置网络、数据库使用到的参数。可设置的参数详情在`library/di/module/ConfigModule`中查看。
2.  使用 OMApplicationComponent
OMApplicationComponent 中保存了框架中使用的一些单例，可使用`OMBaseApplication.getInstanse().getApplicationComponent()`获取并使用。
```
@Singleton
@Component(modules = {ApplicationModule.class,ConfigModule.class, NetworkModule.class, DataBaseModule.class})
public interface OMApplicationComponent {

    Application application();

    ActivityManager getActivityManager();

    IRepositoryManager repositoryManager();

    PreferencesHelper preferencesHelper();

    OMActivityLifecycle OMActivityLifecycle();

    void inject(OMApplicationDelegate delegate);

    @Component.Builder
    interface Builder {
        @BindsInstance
        Builder application(Application application);
        Builder configModule(ConfigModule configModule);
        OMApplicationComponent build();
    }
}
```
3. 使用 RepositoryManager
IRepositoryManager 中提供了获取 Retrofit 的方法，在经过之前`IOMApplicationConfig`配置过之后，传入 Retrofit 的接口可直接获取到网络请求工具（Retrofit 的具体用法可查阅 [Retrofit](https://github.com/square/retrofit)）
```
@Inject
public AccountManager(IRepositoryManager depositoryManager) {
    mDepositoryManager = depositoryManager;
    mAccountApi = mDepositoryManager.obtainRetrofitService(AccountApi.class);
}
```
4. 使用 MVP 框架
本框架使用伴随 Application 全局生命周期的 DataManager 来充当 Model 层，可在不同页面共享数据，通过 Presenter 直接调用 DataManager 的方法，通过定义 View 与 Presenter 接口实现复用的目的
- 使用 Contract
Contract 中定义了 View 与 Presenter 之间的协议。
```
public interface AccountContract {

    interface View extends IView {
        void onLoginSuccess(Account account);
    }

    interface Presenter extends IPresenter {

        String getPhone();
        void login(String account, String pwd);
    }
}
```
- 使用 View
使用 Activity 或者 Fragment 来继承 Contract 中定义的 View ，在`void initInject()`方法中可注入对应定义的 Presenter，并在`ActivityComponent`中注入该 Activity 即可，如下所示
```
void inject(LoginInActivity activity);
```
（Fragment 同理在`FragmentComponent` 中注入）；如果页面简单，不实现`void initInject()`方法即可。
```
public class LoginInActivity extends BaseActivity<AccountContract.Presenter> implements AccountContract.View {

    @Override
    protected void initData(Bundle savedInstanceState) {
    }

    @Override
    protected void initInject() {
        DaggerActivityComponent.builder()
                .managerComponent(DemoApplication.getManagerComponent())
                .activityModule(new ActivityModule(this))
                .build()
                .inject(this);
    }

    @Override
    public int initView(@Nullable Bundle savedInstanceState) {
        return R.layout.fragment_account_login_layout;
    }
}
```
- 使用 Presenter
Presenter 层提供视图与数据层的交互，从下面要介绍的 DataManager 中获取数据进过处理传给 View 层。
```
public class AccountPresenter extends BasePresenter<AccountContract.View> {

    @Inject
    AccountManager mAccountManager;

    @Inject
    public AccountPresenter() {

    }

    @Override
    protected void onAttachView() {

    }

    @Override
    protected void onDetachView() {

    }
}
```
- 使用 DataManager
在工程中的相同模块的逻辑可以使用相同的 DataManager 来实现数据共享，在工程中，提前生成相应的 DataManager ，在 Presenter 中注入一个或者多个全局的 DataManager 即可使用
	- ManagerComponent
```
@ManagerScope
@Component(dependencies = OMApplicationComponent.class,modules = ManagerModule.class)
public interface ManagerComponent {
    AccountManager accountManager();
}
```
	- ManagerModule
```
@Module
public class ManagerModule {

    Context mContext;

    public ManagerModule(Context context) {
        mContext = context;
    }

    @Provides
    @ManagerScope
    AccountManager provideAccountManager(IRepositoryManager repositoryManager,PreferencesHelper preferencesHelper){
        return new AccountManager(repositoryManager,preferencesHelper);
    }
}
```

### 其他功能介绍
1. OMLog
在程序中可使用 `Logx.v/i/d/w/e`来打印 Log，支持 String、@StrRes、Json，当需要自定义 tag 时，可以使用 `Logx.tag(TAG).v/i/d/w/e`来使用
```
Logx.tag(TAG).w("mActivityList == null when killActivity(Class)");
Logx.d("build database ");
```

2. Toast
使用 Toastx 来显示 Toast 消息，Toast 内部已做好视图复用，当多次调用 Toast 消息时不会连续弹消息，只会保留最后一次的消息，提升用户体验。
Toastx 还支持两种视图布局：默认的 Toast 与显示在屏幕中间黑色背景的 Toast。

```
Toastx.show("已复制到剪贴板");
```
