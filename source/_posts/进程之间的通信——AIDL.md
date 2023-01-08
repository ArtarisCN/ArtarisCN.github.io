---
layout: post
title: 进程之间的通信——AIDL
date: 2018-11-19 15:51:54
tags: [Android]
categories:
- Android
---

# 进程之间的通信——AIDL（Android Interface Definition Language)

AIDL 是 Android 提供的一种进程间通信 (IPC—— Inter-Process Communication) 机制。
在 Android 上，一个进程通常无法访问另一个进程的内存。 每一个进程都有自己的Dalvik VM实例，都有自己的一块独立的内存，都在自己的内存上存储自己的数据，执行着自己的操作，都在自己的那片狭小的空间里过完自己的一生。每个进程之间都你不知我，我不知你。尽管如此，进程需要将其对象分解成操作系统能够识别的原语，并将对象编组成跨越边界的对象。
AIDL 就是 Android 为我们提供的一种跨进程的通讯方式。通过这种机制，我们只需要写好 aidl 接口文件，编译时系统会帮我们生成 Binder 接口。

<!-- more -->

### AIDL 支持的数据类型

- Java 的基本数据类型
- String 类型，CharSequence类型。
- List 和 Map
  - 元素必须是 AIDL 支持的数据类型
  - Server 端具体的类里则必须是 ArrayList 或者 HashMap
- 其他 AIDL 生成的接口
- 实现 Parcelable 的实体

### AIDL 文件的分类

AIDL 文件分为两种，**一类**是用来定义parcelable对象，以供其他AIDL文件使用AIDL中非默认支持的数据类型的。**一类**是用来定义方法接口，以供系统使用来完成跨进程通信的。可以看到，两类文件都是在“定义”些什么，而不涉及具体的实现，这就是为什么它叫做“Android接口定义语言”。

### AIDL 如何编写

AIDL 的编写主要为以下三部分：

1. 创建 AIDL
   1. 创建要操作的实体类，实现 Parcelable 接口，以便序列化/反序列化；
   2. 新建 aidl 文件夹，在其中创建接口 aidl 文件以及实体类的映射 aidl 文件；
   3. Make project ，生成 Binder 的 Java 文件；
2. 服务端
   1. 新建服务端工程，将在客户端定义的 aidl 接口文件拷贝至服务端并 Make project ，生成相同的 Stub 类；
   2. 创建 Service，在其中创建上面生成的 Binder 对象实例，继承刚才生成的 Stub ，以实现接口定义的方法；
   3. 在 onBind() 中返回刚才定时的 Binder；
   4. 在 AndroidManifest.xml 中声名创造的 Service，提供外部 Action 供客户端调用；
3. 客户端
   1. 实现 ServiceConnection 接口，在其中拿到 AIDL 类；
   2. bindService()，指定 Action 与 PackageName，启动相应的 Service;
   3. 获取通过服务端返回的 Binder ，调用 AIDL 类中定义好的操作请求，进行数据传输；

### AIDL 的代码实例

以下是关键代码演示：
**客户端**

1. 先定义要传输的实体数据类型，实现 Parcelable 接口

```
public class Person implements Parcelable {
    private String mName;

    public String getName() {
        return mName;
    }

    public Person(String name) {
        mName = name;
    }

    protected Person(Parcel in) {
        mName = in.readString();
    }

    public static final Creator<Person> CREATOR = new Creator<Person>() {
        @Override
        public Person createFromParcel(Parcel in) {
            return new Person(in);
        }

        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeString(mName);
    }
}
```

1. 新建 aidl 文件夹，在其中创建接口 aidl 文件以及实体类的映射 aidl 文件

```
// PersonTransmissionInterface.aidl
package cn.artaris.aidldemo;

// Declare any non-default types here with import statements
// 非基本类型的数据需要导入，需要导入它的全路径。
import cn.artaris.aidldemo.Person;
import java.util.List;

interface PersonTransmissionInterface {
    //方法参数中，除了基本数据类型，其他类型的参数都需要标上方向类型
    //in(输入), out(输出), inout(输入输出)
    void addPerson(in Person person);

    List<Person> getPersonList();
}

```

```
// Person.aidl
package cn.artaris.aidldemo;
// Declare any non-default types here with import statements
//通过 Person.aidl 接口找到真正的 实体类 Person.java
parcelable Person;
```

1. Make project ，生成 Binder 的 Java 文件；

```
public interface PersonTransmissionInterface extends android.os.IInterface {
    /**
     * Local-side IPC implementation stub class.
     */
    public static abstract class Stub extends android.os.Binder implements cn.artaris.aidldemo.PersonTransmissionInterface {
        private static final java.lang.String DESCRIPTOR = "cn.artaris.aidldemo.PersonTransmissionInterface";

        /**
         * Construct the stub at attach it to the interface.
         */
        public Stub() {
            this.attachInterface(this, DESCRIPTOR);
        }

        /**
         * Cast an IBinder object into an cn.artaris.aidldemo.PersonTransmissionInterface interface,
         * generating a proxy if needed.
         */
         //转换 Binder 对象，判断是本地还是远程对象
        public static cn.artaris.aidldemo.PersonTransmissionInterface asInterface(android.os.IBinder obj) {
            if ((obj == null)) {
                return null;
            }
            android.os.IInterface iin = obj.queryLocalInterface(DESCRIPTOR);
            if (((iin != null) && (iin instanceof cn.artaris.aidldemo.PersonTransmissionInterface))) {
                return ((cn.artaris.aidldemo.PersonTransmissionInterface) iin);
            }
            return new cn.artaris.aidldemo.PersonTransmissionInterface.Stub.Proxy(obj);
        }

        @Override
        public android.os.IBinder asBinder() {
            return this;
        }

        //服务器接收到数据时调用
        @Override
        public boolean onTransact(int code, android.os.Parcel data, android.os.Parcel reply, int flags) throws android.os.RemoteException {
            java.lang.String descriptor = DESCRIPTOR;
            switch (code) {
                case INTERFACE_TRANSACTION: {
                    reply.writeString(descriptor);
                    return true;
                }
                case TRANSACTION_addPerson: {
                    data.enforceInterface(descriptor);
                    cn.artaris.aidldemo.Person _arg0;
                    if ((0 != data.readInt())) {
                        _arg0 = cn.artaris.aidldemo.Person.CREATOR.createFromParcel(data);
                    } else {
                        _arg0 = null;
                    }
                    this.addPerson(_arg0);
                    reply.writeNoException();
                    return true;
                }
                case TRANSACTION_getPersonList: {
                    data.enforceInterface(descriptor);
                    java.util.List<cn.artaris.aidldemo.Person> _result = this.getPersonList();
                    reply.writeNoException();
                    reply.writeTypedList(_result);
                    return true;
                }
                default: {
                    return super.onTransact(code, data, reply, flags);
                }
            }
        }

        //Binder 的包装内部类，供远程传输使用
        private static class Proxy implements cn.artaris.aidldemo.PersonTransmissionInterface {
            private android.os.IBinder mRemote;

            Proxy(android.os.IBinder remote) {
                mRemote = remote;
            }

            @Override
            public android.os.IBinder asBinder() {
                return mRemote;
            }

            public java.lang.String getInterfaceDescriptor() {
                return DESCRIPTOR;
            }

            @Override
            public void addPerson(cn.artaris.aidldemo.Person person) throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    if ((person != null)) {
                        _data.writeInt(1);
                        person.writeToParcel(_data, 0);
                    } else {
                        _data.writeInt(0);
                    }
                    mRemote.transact(Stub.TRANSACTION_addPerson, _data, _reply, 0);
                    _reply.readException();
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
            }

            @Override
            public java.util.List<cn.artaris.aidldemo.Person> getPersonList() throws android.os.RemoteException {
                android.os.Parcel _data = android.os.Parcel.obtain();
                android.os.Parcel _reply = android.os.Parcel.obtain();
                java.util.List<cn.artaris.aidldemo.Person> _result;
                try {
                    _data.writeInterfaceToken(DESCRIPTOR);
                    mRemote.transact(Stub.TRANSACTION_getPersonList, _data, _reply, 0);
                    _reply.readException();
                    _result = _reply.createTypedArrayList(cn.artaris.aidldemo.Person.CREATOR);
                } finally {
                    _reply.recycle();
                    _data.recycle();
                }
                return _result;
            }
        }

        static final int TRANSACTION_addPerson = (android.os.IBinder.FIRST_CALL_TRANSACTION + 0);
        static final int TRANSACTION_getPersonList = (android.os.IBinder.FIRST_CALL_TRANSACTION + 1);
    }

    //实现 PersonTransmissionInterface 定义的两个接口
    public void addPerson(cn.artaris.aidldemo.Person person) throws android.os.RemoteException;

    public java.util.List<cn.artaris.aidldemo.Person> getPersonList() throws android.os.RemoteException;
}

```

**服务器**

1. 将在客户端定义的 aidl 接口文件拷贝至服务端并 Make project ，生成相同的 Stub 类
   *略*
2. 创建 Service，在其中创建上面生成的 Binder 对象实例，继承刚才生成的 Stub ，以实现接口定义的方法，在 onBind() 中返回刚才定时的 Binder；

```
public class PersonTransmissionService extends Service {


    PersonTransmissionImpl mBinder = new PersonTransmissionImpl();

    private List<Person> mPeople = new ArrayList<>();

    @Override
    public void onCreate() {
        super.onCreate();
        Log.d("PersonService", "### Transmission service created");
    }

    @Nullable
    @Override
    public IBinder onBind(Intent intent) {
        return mBinder;
    }
    // 定义实现 aidl 接口的实现句柄类。由于 Stub 继承了 IBinder，将在onBind(Intent intent)返回 该句柄
    class PersonTransmissionImpl extends PersonTransmissionInterface.Stub {
        @Override
        public void addPerson(Person person) throws RemoteException {
            mPeople.add(person);
            Log.d("PersonService", person.getName());
        }

        @Override
        public List<Person> getPersonList() throws RemoteException {
            return mPeople;
        }
    }
}
```

1. 在 AndroidManifest.xml 中声名创造的 Service，提供外部 Action 供客户端调用；

```
<service android:name=".PersonTransmissionService"
    android:exported="true"
    //定义线程
    android:process=":remote"
    android:label="@string/app_name">
    <intent-filter>
        //提供 Action 供外部调用
        <action android:name="cn.artaris.aidi_server.PersonTransmissionService"/>
        </intent-filter>
</service>
```

**客户端**

1. 实现 ServiceConnection 接口，在其中拿到 AIDL 类，通过 bindService()，指定 Action 与 PackageName，启动相应的 Service，获取通过服务端返回的 Binder ，调用 AIDL 类中定义好的操作请求，进行数据传输；

```
public class ScrollingActivity extends AppCompatActivity {

    PersonTransmissionInterface mBinder;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_scrolling);

        Button add = findViewById(R.id.add_person);
        add.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if(mBinder == null){
                    bindSsoAuthServices();
                } else {
                    addPersion();
                }
            }
        });


        Button get = findViewById(R.id.get_person);
        get.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                if(mBinder == null)
                    return;

                getPerson();
            }
        });
    }


    private void bindSsoAuthServices(){
        //指定服务端的包名、Action名以找到对应的 Service
        Intent intent = new Intent("cn.artaris.aidi_server.PersonTransmissionService");
        intent.setPackage("cn.artaris.aidi_server");
        bindService(intent,mServiceConnection,Context.BIND_AUTO_CREATE);

    }

    private ServiceConnection mServiceConnection = new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            //关键代码，通过
            mBinder = PersonTransmissionInterface.Stub.asInterface(service);
            addPersion();
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {

        }
    };

    String[] name = new String[]{"李一","李二","李三","李四","李五","李六","李七","李九","李十"};
    int count = 0;
    private void addPersion(){
        Log.d("ScrollingActivity","### person client addPersion");
        try {
            mBinder.addPerson(new Person(name[count++]));
        } catch (RemoteException e) {
            e.printStackTrace();
        }

    }

    private void getPerson(){
        try {
            Log.d("ScrollingActivity",String.valueOf(mBinder.getPersonList().size()));
        } catch (RemoteException e) {
            e.printStackTrace();
        }
    }


    @Override
    protected void onDestroy() {
        super.onDestroy();
        unbindService(mServiceConnection);
    }
}
```

### AIDL 原理分析

在 `PersonTransmissionInterface.java` 中自动生成了 `PersonTransmissionInterface` 接口，接口中包含了我们在`PersonTransmissionInterface.aidl` 定义的两个接口。
而最重要的生成了 `Stub` 类，该类继承了 Binder 类并实现了 `PersonTransmissionInterface` 接口。
Stub最重要的就是 `asInterface()` 这个函数，在这个函数中会自动判断参数 obj 的类型，如果 obj 是本地的接口类型，则 AIDL 会认为不是进程之间的通信，会直接返回一个本地的 `PersonTransmissionInterface` 对象；否则会转换成 `Stub` 类的一个内部类 Proxy 来包装 obj ，将其赋值给内部的 `mRemote` 字段，`Proxy` 也实现了 `PersonTransmissionInterface` 接口，不同的是他是通过 `Binder` 机制来与远程进行交互的，例如`mRemote.transact(Stub.TRANSACTION_addPerson, _data, _reply, 0);`，它请求的类型为 `Stub.TRANSACTION_addPerson`，参数为 `Parcel` 格式的 data。
对于服务端的代码来说，它是拥有同一份的 `PersonTransmissionInterface.aidl` 与 `PersonTransmissionInterface.java` 但不同的是服务端是指令的接口端，客户端的调用会通过 Binder 机制传递到服务端，最后到服务端调用的 `Stub` 的 `boolean onTransact()` 函数，通过 code 类型判断具体调用的函数，此时两端的传递就对应上了。
客户端调用了 `bindService(intent,mServiceConnection,Context.BIND_AUTO_CREATE);`之后，如果绑定成功则会调用 `void onServiceConnected(ComponentName name, IBinder service)`，这里的 `Service` 是 `BinderProxy` 经过 `asInterface()`转换以后，被包装成了 `Proxy` 对象，但是执行的时候，调用的是服务端的对应方法。即：`PersonTransmissionInterface` 的实例 `mBinder` 被服务端包装成 `BinderProxy` 类型，再经过客户端的 `Proxy` 进行包装，通过 `Binder` 机制进行包装，实现进程之间的调用。
