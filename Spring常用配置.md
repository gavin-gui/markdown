### 1.Bean的Scope

---

Spring中的Scope注解主要是为了解决Bean的实例问题，就是Bean在不同的场合下到底有几个实例，是单例模式还是其它模式？一般来说，Spring的Scope有如下几种：

- Singleton：表示该Bean是单例模式，在Spring容器中共享一个Bean实例
- Prototype：每次调用都会新建一个Bean的实例
- Request：这个是在使用Web中，给每一个http request新建一个Bean实例
- Session：这个同样是使用在Web中，表示给每一个http session新建一个Bean实例



编写一个简单的案例来看看@Scope注解的使用：

```java
@Component
@Scope("singleton")
public class ScopeTest{
    
}
```

这里使用Scope("singleton")注解，表示一个单例模式，如果要使用Prototype模式，可以将singleton改为prototype。





### 2.Spring EL和资源调用

---

Spring中的EL表达式类似于JSP中的EL表达式，它支持在XML和注解中使用表达式。

@Value的值有两类：

① ${ property **:** default_value }

② #{ obj.property**? :** default_value }

就是说，第一个注入的是外部参数对应的property，第二个则是SpEL表达式对应的内容。

那个 default_value，就是前面的值为空时的默认值。

示例：

```java
@Configuration
@PropertySource(value = "t.properties", encoding = "UTF-8")
public class ELConfig {
    //注入字符串
    @Value("I Love You!")
    private String normal;
    //注入系统属性
    @Value("#{systemProperties['os.name']}")
    private String osName;
    @Value("#{systemEnvironment['os.arch']}")
    private String osArch;
    //注入方法计算结果
    @Value("#{T(java.lang.Math).random()*100}")
    private double randomNumber;
    //注入对象属性
    @Value("#{userService.name}")
    private String author;
    //注入资源对象
    @Value("t.txt")
    private Resource testFile;

    @Value("http://www.baidu.com")
    private Resource testUrl;
    @Value("${user.username}")
    private String su;
    @Value("${user.password}")
    private String sp;
    @Value("${user.nickname}")
    private String sn;
    @Autowired
    private Environment environment;

    public void output() {
        try {
            System.out.println(normal);
            System.out.println(osName);
            System.out.println(osArch);
            System.out.println(randomNumber);
            System.out.println(author);
            System.out.println(IOUtils.toString(testFile.getInputStream(),"UTF-8"));
            //访问网址
            System.out.println(IOUtils.toString(testUrl.getInputStream(),"UTF-8"));
            //获取网址
            System.out.println("testUrl.getURL():"+testUrl.getURL());
            System.out.println(su);
            System.out.println(sp);
            System.out.println(sn);
            System.out.println(environment.getProperty("user.nickname"));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```





### 3.Bean的初始化与销毁

---

有两种方式：

- 使用@Bean的注解中的initMethod和destroyMetho。

  例如：@Bean(initMethod = "init",destroyMethod = "destroy")，init和destroy对于类中的方法

- 使用JSR-250中的注解@PostConstruct和@PreDestroy。

  @PostConstruct //构造方法执行之后执行

  @PreDestroy //销毁之前执行



### 4.Profile

Profile可以理解为我们在Spring容器中所定义的Bean的**逻辑组名称**，只有当这些Profile被激活的时候，才会将Profile中所对应的Bean注册到Spring容器中。

示例：

```java
//示例Bean
public class DemoBean {

    private String content;

    public DemoBean(String content) {
        super();
        this.content = content;
    }

    public String getContent() {
        return content;
    }

    public void setContent(String content) {
        this.content = content;
    }
}

//配置类
@Configuration
public class ProfileConfig {
    @Bean
    @Profile("dev")
    public DemoBean devDemoBean() {
        return new DemoBean("dev");
    }

    @Bean
    @Profile("prod")
    public DemoBean prodDemoBean() {
        return new DemoBean("prod");
    }
}

//测试类
public class Main {
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.getEnvironment().setActiveProfiles("prod");
        context.register(ProfileConfig.class);
        context.refresh();

        DemoBean bean = context.getBean(DemoBean.class);
        System.out.println(bean.getContent());

        context.close();
    }
}
//输出prod
```

当Profile为dev时使用devDemoBean来实例化DemoBean，当Profile为prod时，使用prodDemoBean来实例化DemoBean。

 这里先获取Spring容器，不过在获取容器时先不传入配置文件，待我们先将活动的Profile置为prod之后，再设置配置文件，设置成功之后，一定要刷新容器。



### 5.Spring中的事件传递

---

Spring中的事件使用了观察者模式，主要包含三个部分：

- 事件载体

  继承自ApplicationEvent，创建一个事件类

- 事件监听器

  实现ApplicationListener接口，创建事件监听器

- 事件发布

  调用ApplicationContext的publishEvent(event)方法，发布事件



示例：

```java
//定义事件载体
public class DemoEvent extends ApplicationEvent{
    private String msg;

    public DemoEvent(Object source, String msg) {
        super(source);
        this.msg = msg;
    }

    public String getMsg() {
        return msg;
    }

    public void setMsg(String msg) {
        this.msg = msg;
    }
}

//定义事件监听器
@Component
public class DemoListener implements ApplicationListener<DemoEvent> {
    public void onApplicationEvent(DemoEvent demoEvent) {
        System.out.println("我收到DemoEvent的事件了:"+demoEvent.getMsg());
    }
}

//事件发布
@Component
public class DemoPublish{
    @Autowired
    ApplicationContext applicationContext;

    public void publish(String msg) {
        applicationContext.publishEvent(new DemoEvent(this,msg));
    }
}

```



### 6.Spring多线程

---

在使用线程的时候我们通常会使用线程池，Spring对线程池提供了很好的支持。下面来看下Spring是如何使用线程池的。

- 创建线程池配置类

  ```java
  @Configuration
  @ComponentScan("com.gui")
  @EnableAsync//开启异步任务支持
  public class TaskExecutorConfig implements AsyncConfigurer {
      //返回一个线程池
      public Executor getAsyncExecutor() {
          //创建并配置线程池
          ThreadPoolTaskExecutor taskExecutor = new ThreadPoolTaskExecutor();
          taskExecutor.setCorePoolSize(2);
          taskExecutor.setMaxPoolSize(5);
          taskExecutor.setQueueCapacity(25);
          taskExecutor.initialize();
          return taskExecutor;
      }

      public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
          return null;
      }
  }
  //这里的配置类实现了AsyncConfigurer接口，并重写了getAsyncExecutor()方法，该方法返回一个TreadPoolTaskExecutor类，该类中包含了线程池类。
  ```

- 创建任务执行类

  ```java
  @Service
  public class AsyncTaskService {
      @Async
      public void executeAsyncTask(int i) {
          System.out.println("异步任务1：" + i+"；Thread.currentThread().getName():"+Thread.currentThread().getName());
      }

      @Async
      public void executeAsyncTask2(int i) {
          System.out.println("异步任务2：" + i+"；Thread.currentThread().getName():"+Thread.currentThread().getName());
      }
  }
  //我们使用@Async注解表明该方法是一个异步方法，如果这个注解添加在类上，则表明这个类的所有方法都是异步方法。
  ```

- 执行

  ```java
  public class Main {
      public static void main(String[] args) {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(TaskExecutorConfig.class);
          AsyncTaskService bean = context.getBean(AsyncTaskService.class);
          System.out.println(Thread.currentThread().getName());
          for (int i = 0; i < 10; i++) {
              bean.executeAsyncTask(i);
              bean.executeAsyncTask2(i);
          }
          context.close();
      }
  }
  ```

  ​

### 7.Spring计划任务

---

Spring对计划任务的支持也非常好，使用起来非常方便。

- 计划任务配置

  ```java
  @Configuration
  @ComponentScan("com.gui")
  @EnableScheduling//开启对计划任务的支持
  public class MyConfig {
  }
  ```

  ​

- 计划任务执行类

  ```java
  @Service
  public class ScheduledTaskService {
      private static final SimpleDateFormat FORMAT = new SimpleDateFormat("HH:mm:ss");

      @Scheduled(fixedRate = 5000)
      public void reportCurrentTime() {
          System.out.println("每隔5秒执行一次:" + FORMAT.format(new Date()));
      }

      @Scheduled(fixedDelay = 10000)
      public void delayExecuteTask() {
          System.out.println("延迟10s之后，每隔10s执行一次");
      }

      /**
       * CronTrigger配置格式:
       格式: [秒] [分] [小时] [日] [月] [周] [年]
       序号 说明 是否必填 允许填写的值 允许的通配符
       1   秒    是      0-59 ,         - * /
       2    分    是      0-59 ,        - * /
       3    小时  是      0-23 ,       - * /
       4    日    是      1-31 ,      - * ? / L W
       5    月    是    1-12 or JAN-DEC , - * /
       6    周     是     1-7 or SUN-SAT , - * ? / L #
       7    年     否     empty 或 1970-2099 , - * /

       通配符说明:
       * 表示所有值. 例如:在分的字段上设置 "*",表示每一分钟都会触发。
       ? 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为"?" 具体设置为 0 0 0 10 * ?
       - 表示区间。例如在小时上设置 "10-12",表示 10,11,12点都会触发。
       , 表示指定多个值，例如在周字段上设置 "MON,WED,FRI" 表示周一，周三和周五触发
       / 用于递增触发。如在秒上面设置"5/15" 表示从5秒开始，每增15秒触发(5,20,35,50)。 在月字段上设置'1/3'所示每月1号开始，每隔三天触发一次。
       L 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于"7"或"SAT"。如果在"L"前加上数字，则表示该数据的最后一个。例如在周字段上设置"6L"这样的格式,则表示“本 月最后一个星期五"
       W 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上设置"15W"，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 "1W",它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，"W"前只能设置具体的数字,不允许区间"-").
       小提示
       'L'和 'W'可以一组合使用。如果在日字段上设置"LW",则表示在本月的最后一个工作日触发(一般指发工资 )
       # 序号(表示每月的第几个周几)，例如在周字段上设置"6#3"表示在每月的第三个周六.注意如果指定"#5",正好第五周没有周六，则不会触发该配置(用 在母亲节和父亲节再合适不过了)
       小提示
       周字段的设置，若使用英文字母是不区分大小写的 MON 与mon相同.

       常用示例:
       0 0 12 * * ? 每天12点触发
       0 15 10 ? * * 每天10点15分触发
       0 15 10 * * ? 每天10点15分触发
       0 15 10 * * ? * 每天10点15分触发
       0 15 10 * * ? 2005 2005年每天10点15分触发
       0 * 14 * * ? 每天下午的 2点到2点59分每分触发
       0 0/5 14 * * ? 每天下午的 2点到2点59分(整点开始，每隔5分触发)
       0 0/5 14,18 * * ? 每天下午的 2点到2点59分(整点开始，每隔5分触发) 每天下午的 18点到18点59分(整点开始，每隔5分触发)
       0 0-5 14 * * ? 每天下午的 2点到2点05分每分触发
       0 10,44 14 ? 3 WED 3月分每周三下午的 2点10分和2点44分触发
       0 15 10 ? * MON-FRI 从周一到周五每天上午的10点15分触发
       0 15 10 15 * ? 每月15号上午10点15分触发
       0 15 10 L * ? 每月最后一天的10点15分触发
       0 15 10 ? * 6L 每月最后一周的星期五的10点15分触发
       0 15 10 ? * 6L 2002-2005 从2002年到2005年每月最后一周的星期五的10点15分触发
       0 15 10 ? * 6#3 每月的第三周的星期五开始触发
       0 0 12 1/5 * ? 每月的第一个中午开始每隔5天触发一次
       0 11 11 11 11 ? 每年的11月11号 11点11分触发

       参考网址：http://blog.csdn.net/irencewh/article/details/45332295（出处没找到）
       */
      @Scheduled(cron = "0 51 20 * * ?")
      public void fixTimeExecution() {
          System.out.println("在指定时间:"+FORMAT.format(new Date())+"执行");
      }
  }
  ```

  ​


- 执行

  ```java
  public class Main {
      private static final SimpleDateFormat FORMAT = new SimpleDateFormat("HH:mm:ss");

      public static void main(String[] args) {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(MyConfig.class);
          ScheduledTaskService bean = context.getBean(ScheduledTaskService.class);
          System.out.println("当前时间:"+FORMAT.format(new Date()));
      }
  }
  ```

