# Туториал по установке и настройке ROS-Android

"First get some food or something, cause this is gonna take some time" (c)

## Начало

Ubuntu 18.04, ROS melodic, Android-Studio 3.2

Запустите Ubuntu.

### Java

Версия: Java 8 Oracle (не 10), если у вас уже стоит 10, то установите 8 и укажите env-переменную $JAVA_HOME - путь к папке с Java 8 Oracle (в конце java-части описано как это сделать).

```
sudo add-apt-repository ppa:webupd8team/java
sudo apt-get update
sudo apt-get install oracle-java8-installer
```

Проверить версию Java можно:

```
java -version
```

С Java-home (настройка версии java по умолчанию):

Откройте файл etc/environment в текстовом редакторе (gedit) и добавьте (свой) путь к версии java:

```
JAVA_HOME="/usr/lib/jvm/java-8-oracle"
```

Затем либо:

```
source /etc/environment
```

Либо перезагрузите терминал.

### Android-Studio

Скачайте и установите https://developer.android.com/studio/

Через SDK-менеджер установите все, что там есть.

Проверьте, чтобы в папке build-tools были версии 21.1.2, 25.0.2, 26.0.2, 28.0.2

Android-ndk установите 17й версии (https://developer.android.com/ndk/downloads/older_releases).

###  Rosjava

```
sudo apt-get install ros-melodic-catkin ros-melodic-rospack python-wstool
```
```
mkdir -p ~/rosjava/src
```
```
wstool init -j4 ~/rosjava/src https://raw.githubusercontent.com/rosjava/rosjava/kinetic/rosjava.rosinstall
```
```
source /opt/ros/melodic/setup.bash
```
```
cd ~/rosjava
```

Убедимся, что имеются все Rosdep и msg пакеты:

```
rosdep update
```
```
rosdep install --from-paths src -i -y
```
```
catkin_make
```

Не страшно, если rosdep install завершится ошибкой.
Catkin_make займет довольно много времени.


### Android-core

```
mkdir -p ~/android_core
```
```
wstool init -j4 ~/android_core/src https://raw.github.com/rosjava/rosjava/kinetic/android_core.rosinstall
```
```
source /opt/ros/melodic/setup.bash 
```
```
source ~/rosjava/devel/setup.bash
```
```
cd ~/android_core
```

Теперь создайте файл local.properties и поместите его в android_core/src/android_core и в android_core/src/android_extras

В файле укажите путь к Android SDK и NDK:

```
sdk.dir=/home/имя_пользователя/Android/Sdk
ndk.dir=/home/имя_пользователя/Android/Sdk/ndk-bundle
```

(если ваша Android NDK 17 находится там)

```
catkin_make
```

### Создаем  Android-ROS-проекты и Android-ROS-пакеты (packages)

Создадим пустой workspace и наполняем его всем необходимым:

```
mkdir -p ~/myandroid/src
```
```
cd ~/myandroid/src
```

Создаем package с названием android_foo, который будет ссылаться на android_core и rosjava, которые мы создали раньше (названия не меняйте пока, catkin_make будет ругаться).

```
catkin_create_android_pkg android_foo android_core rosjava_core std_msgs
```
```
cd android_foo
```
```
catkin_create_android_project -t 10 -p com.github.ros_java.android_foo.bar bar
```
```
catkin_create_android_library_project -t 13 -p com.github.ros_java.android_foo.barlib barlib
```
```
cd ../..
```
```
catkin_make
```

Займет много времени и будет много ругаться

## Устраняем ошибки

Не забудьте добавить env переменную PATH в bashrc:

```
PATH=/home/makary/Android/Sdk/build-tools/21.1.2:$PATH
```

Также скопируйте созданный ранее файл local.properties и поместите его в папку пакета android_foo и в папку проекта bar.

Запустите Android-studio и откройте проект bar, который создали в предыдущем пункте.

При запуске предложат установить правильный gradle-wrapper, соглашаемся.

Теперь заходим в файл build.gradle, который в папке bar, и заменяем все, что там есть, на вот это:

```
dependencies {
    //compile 'org.ros.rosjava_core:rosjava_tutorial_pubsub:[0.3,0.4)'
    compile project(':android_10')
}

apply plugin: 'com.android.application'

android {
    compileSdkVersion 26
    defaultConfig {
        minSdkVersion 24
        targetSdkVersion 24
        versionCode 1
        versionName "1.0"
    }
    buildToolsVersion '26.0.2'
}
```

Помещаем в Android-manifest следующее:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    package="com.github.ros_java.android_foo.bar"
    android:versionCode="1"
    android:versionName="1.0">

    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.SYSTEM_ALERT_WINDOW" />

    <application android:icon="@drawable/ic_launcher" android:label="bar" android:theme="@android:style/Theme.Holo.NoActionBar.Fullscreen" tools:replace="android:icon, android:label" >
        <activity android:name="bar"
            android:label="bar">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <activity android:name="org.ros.android.MasterChooser" />
        <service android:name="org.ros.android.NodeMainExecutorService" >
            <intent-filter>
                <action android:name="org.ros.android.NodeMainExecutorService" />
            </intent-filter>
        </service>

    </application>
</manifest>
```

Позже можете все переименовать (как название приложения, так и темы, и иконку (здесь ее имя ic_launcher и находится она в папке drawable).

В папке android_foo в файле build-gradle заменяете все на 

```
task wrapper(type: Wrapper) {
    gradleVersion = "2.14.1"
}

buildscript {
    apply from: "https://github.com/rosjava/rosjava_bootstrap/raw/kinetic/buildscript.gradle"
    dependencies {
        classpath 'com.android.tools.build:gradle:3.0.1'
    }
    repositories {
        google()
    }
}

configure(subprojects.findAll { it.name.startsWith("android_") }) {
    apply plugin: "ros-android"
        afterEvaluate { project ->
            // Change the layout of Android projects to be compatible with Eclipse.
            android {
                sourceSets {
                    //noinspection GroovyAssignabilityCheck
                    main {
                        manifest.srcFile "AndroidManifest.xml"
                        res.srcDirs "res"
                        assets.srcDirs "assets"
                        java.srcDirs "src"
                    }
                }
                packagingOptions {
                /* https://github.com/rosjava/android_core/issues/194 */
                exclude "META-INF/LICENSE.txt"
                exclude "META-INF/NOTICE.txt"
            }

            lintOptions {
                abortOnError false
            }
        }
    }
}

defaultTasks 'assembleRelease', 'uploadArchives'

repositories {
    google()
}
    
```

Синхронизируем в Android-studio проект с gradle-файлом.

Собираем все через Android-studio (значок молотка).

Теперь ошибок быть не должно (разве что предупреждения).

### Стандартные publisher и subscriber

В папке main/java/ создаем папку org, в ней ros, в ней rosjava_pubsub (например).

В папке rosjava_pubsub создаем два файла:

MessagePub.java

```
package org.ros.rosjava_pubsub;

import android.annotation.SuppressLint;
import android.content.Context;

import org.ros.namespace.GraphName;
import org.ros.node.ConnectedNode;
import org.ros.node.Node;
import org.ros.node.NodeMain;
import org.ros.node.topic.Publisher;

public class MessagePub<T> implements NodeMain {

    private String topicName;
    private String messageType;
    private Publisher<T> publisher;
    private ConnectedNode mConnectedNode;
    public T message ;

    public MessagePub(Context context) {

    }

    public void setTopicName(String topicName) {
        this.topicName = topicName;
    }

    public void changeTopicName(String changetopicName) {
        if(!changetopicName.equals(topicName)) {
            publisher.shutdown();
            publisher = mConnectedNode.newPublisher(changetopicName, messageType);
            message = publisher.newMessage();
        }
    }

    public void setMessageType(String messageType) {
        this.messageType = messageType;
    }

    public void publish() {
        if ((message != null) && (publisher != null)) {
            publisher.publish(message);
        }
    }

    @Override
    public GraphName getDefaultNodeName() {
        return GraphName.of("android/ros_publish");
    }

    @Override
    public void onStart(ConnectedNode connectedNode) {
        mConnectedNode = connectedNode;
        publisher = connectedNode.newPublisher(topicName, messageType);
        message = publisher.newMessage();
    }

    @Override
    public void onShutdown(Node node) {

    }

    @Override
    public void onShutdownComplete(Node node) {
    }

    @Override
    public void onError(Node node, Throwable throwable) {
    }

}
```

И файл MessageSub.java

```
package org.ros.rosjava_pubsub;

import android.annotation.SuppressLint;
import android.content.Context;
import android.util.Log;

import org.ros.message.MessageListener;
import org.ros.namespace.GraphName;
import org.ros.node.ConnectedNode;
import org.ros.node.Node;
import org.ros.node.NodeMain;
import org.ros.node.topic.Subscriber;

public class MessageSub<T> implements NodeMain {

    private String topicName;
    private String messageType;
    private MessageCallable<String, T> callable;
    private ConnectedNode mConnectedNode;
    private Subscriber<T> subscriber;

    public MessageSub(Context context) { }

    public void setTopicName(String topicName) {
        this.topicName = topicName;
    }

    public void changeTopicName(String changetopicName) {
        if(!changetopicName.equals(this.topicName)) {
            this.topicName = changetopicName;
            subscriber.shutdown();
            subscriber = mConnectedNode.newSubscriber(topicName, messageType);
            subscriber.addMessageListener(new MessageListener<T>() {
                @Override
                public void onNewMessage(final T message) {

                    if (callable != null) {
                        callable.call(message);
                    }
                }
            });
        }
    }

    public void setMessageType(String messageType) {
        this.messageType = messageType;
    }

    public void setMessageCallable(MessageCallable<String, T> callable) {
        this.callable = callable;
    }

    @Override
    public GraphName getDefaultNodeName() {
        return GraphName.of("android/ros_subscrib");
    }

    @Override
    public void onStart(ConnectedNode connectedNode) {
        mConnectedNode = connectedNode;
        subscriber = connectedNode.newSubscriber(topicName, messageType);
        subscriber.addMessageListener(new MessageListener<T>() {
            @Override
            public void onNewMessage(final T message) {

                if (callable != null) {
                    callable.call(message);
                }
            }
        });
    }

    @Override
    public void onShutdown(Node node) {
    }

    @Override
    public void onShutdownComplete(Node node) {
    }

    @Override
    public void onError(Node node, Throwable throwable) {
    }


}
```

Теперь основной файл bar/src/main/java/com/github/ros_java/android_foo/bar/bar.java

```
package com.github.ros_java.android_foo.bar;

import android.content.Context;
import android.os.Bundle;
import android.os.Handler;
import android.os.Message;
import android.view.View;
import android.widget.Button;
import android.widget.EditText;
import android.widget.TextView;

import org.ros.android.MessageCallable;
import org.ros.android.RosActivity;
import org.ros.node.NodeConfiguration;
import org.ros.node.NodeMainExecutor;

import org.ros.rosjava_pubsub.MessageSub;
import org.ros.rosjava_pubsub.MessagePub;

public class bar extends RosActivity {

  private MessagePub<std_msgs.String> pubString;
  private MessageSub<std_msgs.String> subString;
  private String subStringMessage;
  private Button publishButton;
  private EditText textPublish;
  private TextView textSubscrib;
  private Context mContext;

  public bar() {
    // The RosActivity constructor configures the notification title and ticker
    // messages.
    super("bar", "bar");
  }

  @SuppressWarnings("unchecked")
  @Override
  public void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.main);
    mContext = getApplicationContext();
    publishButton = (Button) findViewById(R.id.publishButton);
    textPublish = (EditText) findViewById(R.id.stringPublish);
    textSubscrib = (TextView) findViewById(R.id.stringSubscrib);

    publishButton.setOnClickListener(new View.OnClickListener() {
      @Override
      public void onClick(View view) {
        String publishtext = textPublish.getText().toString();
        pubString.message.setData(publishtext);
        pubString.publish();
      }
    });

  }

  @Override
  protected void init(NodeMainExecutor nodeMainExecutor) {

    NodeConfiguration nodeConfiguration = NodeConfiguration.newPublic(getRosHostname());
    nodeConfiguration.setMasterUri(getMasterUri());

    ////////////////publish string_test //
    pubString = new MessagePub<std_msgs.String>(mContext);
    pubString.setTopicName("robot_command");
    pubString.setMessageType(std_msgs.String._TYPE);
    nodeMainExecutor.execute(pubString,
            nodeConfiguration.setNodeName("PUB"));

    ///////////////subscribe string_test//
    subString =  new MessageSub<std_msgs.String>(mContext) ;
    subString.setTopicName("robot_command");
    subString.setMessageType(std_msgs.String._TYPE);
    subString.setMessageCallable(new MessageCallable< String, std_msgs.String>() {
      @Override
      public String call(std_msgs.String message) {
        subStringMessage = message.getData();
        new Thread()
        {
          public void run()
          {
            Message messageImage = handler.obtainMessage();
            handler.sendMessage(messageImage);
          }
        }.start();
        return null ;
      }
    });
    nodeMainExecutor.execute(subString,
            nodeConfiguration.setNodeName("SUB"));

  }

  final Handler handler = new Handler()
  {
    public void handleMessage(Message msg)
    {
      textSubscrib.setText("Subscribe : " + subStringMessage);
    }
  };

}
```

Теперь файл layout/main.xml

```
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:orientation="vertical" >

    <RelativeLayout
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_margin="16dp">


        <Button
            android:id="@+id/publishButton"
            android:layout_width="wrap_content"
            android:layout_height="50dip"
            android:text="Publish" />

        <EditText
            android:id="@+id/stringPublish"
            android:layout_width="150dp"
            android:layout_height="wrap_content"
            android:padding="10dp"
            android:layout_toRightOf="@+id/publishButton"
            android:hint=""/>

        <TextView
            android:id="@+id/stringSubscrib"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:text="Subscribe : "
            android:layout_below="@+id/publishButton"/>

    </RelativeLayout>

</LinearLayout>
```

Собираем проект, настроиваем .bashrc файл, запускаем в терминале roscore и проверяем на девайсе работоспособность приложения.

На основе уже делаем что-либо свое.

## Настройки ROS

Идем домой

```
cd
```

Редактируем .bashrc

```
gedit .bashrc
```

Добавляем в конце:

```
source /opt/ros/melodic/setup.bash
source ~/rosjava/./devel/setup.bash . ~/rosjava/devel/setup.bash
source ~/android_core/./devel/setup.bash . ~/android_core/devel/setup.bash
source ~/myandroid/./devel/setup.bash . ~/myandroid/devel/setup.bash

export ROS_HOSTNAME=192.168.....
export ROS_MASTER_URI=http://192.168.....:11311
export ROS_IP=192.168.....
```

Где многоточие - ваш IP (посмотреть - ifconfig в терминале).

Сохраняем файл, закрываем, перезапускаем терминал.

Далее команда

```
roscore
```

Теперь можно подключаться с девайса к мастеру.

## Предупреждение

Если Вы запустили приложение на эмуляторе, то Вы не сможете полностью проверить его работоспособность. 
После подключения к мастеру Вы сможете посмотреть rostopic list (будет видно, что появился созданный Вами топик), но не сможете просматривать сообщения, которые в него приходят, а также subscriber из Android-приложения не увидит то, что Вы отправляете в топик через rostopic pub.
Все это доступно Вам только при запуске на настоящем Android-устройстве.

## А также

Автор туториала на русском:
* **Makary Boriskin** -  КБПО, ЦНИИ РТК (boriskin.ma@edu.spbstu.ru)

Подробнее:
http://wiki.ros.org/android/Tutorials/kinetic

Может пригодиться, будет интересно:
https://github.com/erkihindo/controller
https://github.com/AuTURBO/ros-app-tb3-voiceorder
