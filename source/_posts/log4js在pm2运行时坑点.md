---
title: log4js在pm2运行时坑点
date: 2018-07-04 17:37:57
tags:
---

### 起因
  近期对logger组件进行升级，使用log4js作为底层日志输出。在开发环境下通过node命令行启动程序，log能正常创建打印。但是在生产环境下，通过pm2启动程序后，发现日志文件无法正常创建。经过散弹枪式编程法跟踪后，发现当实例数量大于1时，日志文件无法正常创建。阅读源码发现是log4js与pm2配合的问题，而log4js的文档中并未提及该问题。

### pm2以cluster模式启动程序时到底干了啥
  既然上文说到与pm2有关，那么pm2是如何启动进程的呢？以下只讨论cluster模式。
  我们都知道cluster模式启动进程时，将会产生一个master进程和N个worker进程，那么pm2在启动应用程序时谁是master，谁是worker？
  实际上pm2启动进程时，master是pm2的内部进程，叫做God Daemon进程。所有的用户代码都是以worker进程的方式启动的。
  在pm2的lib/God.js里有如一段代码：
  ```js
  cluster.setupMaster({
    exec : path.resolve(path.dirname(module.filename), 'ProcessContainer.js')
  });
  ```
  这段代码保证了在调用cluster.fork()方法时，pm2去加载用户代码，并在worker进程中执行。
  对应pm2运行原理的分析，这里先告一个段落，接下来我们看看log4js的源码是如何处理多进程写日志以及如何创建日志文件的。

### 多进程下，该如何写同一个日志文件？
  多个进程同时写一个文件时会发生什么呢？参考：[多进程同时写一个文件会怎样？](https://blog.csdn.net/yangbodong22011/article/details/63064166)
  为了避免多进程同时写一个日志文件，log4js的实现方式是：通过IPC传递消息，由一个进程去写。

  log4js.configure()方法执行的内部有如下的一段代码：
  ```js
  if (config.disableClustering) {
    debug('Not listening for cluster messages, because clustering disabled.');
  } else if (isPM2Master()) {
    // PM2 cluster support
    // PM2 runs everything as workers - install pm2-intercom for this to work.
    // we only want one of the app instances to write logs
    debug(`listening for PM2 broadcast messages, ${process.pid}`);
    process.on('message', receiver);
  } else if (cluster.isMaster) {
    debug(`listening for cluster messages, ${process.pid}`);
    cluster.on('message', receiver);
  } else {
    debug('not listening for messages, because we are not a master process');
  }
  ```
  由上面的源码可以看到，在isPM2Master和cluster.isMaster为true时会注册两个监听函数，当worker进程通过IPC发送消息时，将通过receiver接受消息，并在master中进行写入log的操作。上一节说到，pm2的master是God Daemon进程，进程里肯定不包含任何log4js的调用。那么在使用pm2时，由谁来写日志呢？关键就在于isPM2Master函数。

### isPM2Master是如何工作的
  那么接下来我们将看看isPM2Master()这个方法的具体实现：
  ```js
  function isPM2Master() {
    return config.pm2 && process.env[config.pm2InstanceVar] === '0';
  }
  ```
  我们看到有两个判断条件:
  * 条件一: config.pm2可以通过log4js的配置文件指定，可设置为"pm2=true"
  * 条件二: 从env里取环境变量config.pm2InstanceVar所代表的值，经运行发现默认情况下 pm2InstanceVar = NODE_APP_INSTANCE。

  关于NODE_APP_INSTANCE环境变量，在pm2源码的CHANGE_LOG里有这样一段描述:
  ```text
    If we start let's say 4 instances of an app (cluster_mode),
    Each app will have a value in process.env.NODE_APP_INSTANCE which will be 0 for the first one,
    1, 2 and 3 for the next ones.
  ```

  意即，当pm2启动4个用户进程的实例时，这四个进程的process.env.NODE_APP_INSTANCE环境变量值分别为0,1,2,3

  ** 由此可见，当我们在log4js的配置文件设置pm2=true时，log4js将取NODE_APP_INSTANCE=0的进程作为写日志的主进程，这就避免了多进程写**

### 为什么pm2以多进程方式启动时，无法创建日志文件
  最终问题回到了文章开头提到的现象，究竟是为什么导致了创建日志文件失败呢，我们看看源码。在configuration.js的createAppender方法里有这么一段：
  ```js
  if (this.disableClustering || cluster.isMaster || (this.pm2 && process.env[this.pm2InstanceVar] === '0')) {
      // ...
      const appender = appenderModule.configure(
        config,
        layouts,
        this.configuredAppenders.get.bind(this.configuredAppenders),
        this.configuredLevels
      );
  }
  ```
  可见(this.pm2 && process.env[this.pm2InstanceVar] === '0')就是我们上文提到的isPm2Master方法

### 问题的解决
  由此发现，解决这个问题的方式就是在log4js配置文件里加上pm2=true的选项，具体如下：
  ```js
  log4js.configure({
    appenders: {
      df: {
        type: 'dateFile',
        filename: '/Users/didi/Github/log4js-node/zhlin-logs/xxx.log',
      },
    },
    categories: {
      default: { appenders: ['df'], level: 'DEBUG' }
    },
    pm2: true,
  });
  ```