## 前言
本文记录在springboot中使用 @Scheduled 创建定时任务

## 使用步骤
1. 导包，springboot已经为我们内置了定时任务，所以无需额外的导包只需要在启动器上配置一个注解开启定时器。
```java
@SpringBootApplication
@EnableScheduling
public class ScheduledApplication {

	public static void main(String[] args) {
		SpringApplication.run(ScheduledApplication.class, args);
	}
}
```
2. 在定时任务的方法上加上 @Scheduled 注解进行定时任务的配置，如执行时间，间隔时间，延迟时间等等。
```java
// 每三十分钟执行一次
@Scheduled(cron = "0 */30 * * * ? ")
	public void numberAndPhoneBinding() {
		dothing();
	}
```
可以将定时任务的开启与执行写入配置文件中，如：
```java
## 定时任务基础配置
scheduling:
    enabled: true   #定时任务开关
    isExcuted: true  #是否执行统计
    thread:         #定时任务线程池配置
        threadName: ${spring.application.name}-thread
        maxPoolSize: 20
        corePoolSize: 10
        queueCapacity: 200
```
在需要使用到定时任务的类中注入：
```java
@Component
@ConditionalOnProperty(prefix = "scheduling", name = "enabled", havingValue = "true")
public class TaskAx2Client {
	@Value("${scheduling.isExcuted}")
	private Boolean isExcuted;
	@Autowired
	private TaskAx2ClientService taskAx2ClientService;
	
	/**
	 * 
	 * @Title:        daySynthesizeReportThred 
	 * @Description:  用户号码与排班任务号码绑定 每30分钟执行一次 
	 * @param:            
	 * @return:       void    
	 * @author        dave
	 * @date          2020年7月6日 上午11:07:11
	 */
	//@Scheduled(cron = "0 */30 * * * ? ")
	@Scheduled(fixedDelay = 60*1000)
	public void numberAndPhoneBinding() {
		if(isExcuted) {
			dothing();
		}
	}
	
}
```
## @Scheduled注解参数
### 1. cron
该参数接收一个cron表达式，cron表达式是一个字符串，字符串以5或6个空格隔开，分开共6或7个域，每一个域代表一个含义。cron表达式语法如下
```
[秒] [分] [小时] [日] [月] [周] [年]
```
注：[年]不是必须的域，可以省略[年]，则一共6个域
| 序号 | 说明 | 必填 | 允许填写的值 | 允许的通配符 |
| :---:  |  :---:  |  :---:  |  :---:  |  :---:  |
|1|秒|是|0-59|, - * /|
|2|分|是	|0-59|, - * /|
|3|	时|	是|	0-23|	, - * /|
|4|	日|	是|	1-31|	, - * ? / L W|
|5|	月|	是|	1-12 / JAN-DEC|	, - * /|
|6|	周|	是|	1-7 or SUN-SAT|	, - * ? / L #|
|7|	年|	否|	1970-2099|	, - * /|


- <font color='red'> \* </font> 表示所有值。 例如:在分的字段上设置 *,表示每一分钟都会触发。
- <font color='red'> ? </font> 表示不指定值。使用的场景为不需要关心当前设置这个字段的值。例如:要在每月的10号触发一个操作，但不关心是周几，所以需要周位置的那个字段设置为”?” 具体设置为 0 0 0 10 * ?
- <font color='red'> \- </font> 表示区间。例如 在小时上设置 “10-12”,表示 10,11,12点都会触发。
- <font color='red'> , </font> 表示指定多个值，例如在周字段上设置 “MON,WED,FRI” 表示周一，周三和周五触发
- <font color='red'> / </font> 用于递增触发。如在秒上面设置”5/15” 表示从5秒开始，每增15秒触发(5,20,35,50)。 在日字段上设置’1/3’所示每月1号开始，每隔三天触发一次。
- <font color='red'> L </font> 表示最后的意思。在日字段设置上，表示当月的最后一天(依据当前月份，如果是二月还会依据是否是润年[leap]), 在周字段上表示星期六，相当于”7”或”SAT”。如果在”L”前加上数字，则表示该数据的最后一个。例如在周字段上设置”6L”这样的格式,则表示“本月最后一个星期五”
- <font color='red'> W </font> 表示离指定日期的最近那个工作日(周一至周五). 例如在日字段上置”15W”，表示离每月15号最近的那个工作日触发。如果15号正好是周六，则找最近的周五(14号)触发, 如果15号是周未，则找最近的下周一(16号)触发.如果15号正好在工作日(周一至周五)，则就在该天触发。如果指定格式为 “1W”,它则表示每月1号往后最近的工作日触发。如果1号正是周六，则将在3号下周一触发。(注，”W”前只能设置具体的数字,不允许区间”-“)。
- <font color='red'> \# </font> 序号(表示每月的第几个周几)，例如在周字段上设置”6#3”表示在每月的第三个周六.注意如果指定”#5”,正好第五周没有周六，则不会触发该配置(用在母亲节和父亲节再合适不过了) ；小提示：’L’和 ‘W’可以一组合使用。如果在日字段上设置”LW”,则表示在本月的最后一个工作日触发；周字段的设置，若使用英文字母是不区分大小写的，即MON与mon相同。

**示例：**
1. 每隔5秒执行一次：*/5 * * * * ?

2. 每隔1分钟执行一次：0 */1 * * * ?

3. 每天23点执行一次：0 0 23 * * ?

4. 每天凌晨1点执行一次：0 0 1 * * ?

5. 每月1号凌晨1点执行一次：0 0 1 1 * ?

6. 每月最后一天23点执行一次：0 0 23 L * ?

7. 每周星期六凌晨1点实行一次：0 0 1 ? * L

8. 在26分、29分、33分执行一次：0 26,29,33 * * * ?

9. 每天的0点、13点、18点、21点都执行一次：0 0 0,13,18,21 * * ?

**cron表达式使用占位符**
另外，cron 属性接收的 cron 表达式支持占位符。eg：
配置文件：
```yml
time:
  cron: */5 * * * * *
  interval: 5
```
每5秒执行一次：
```java
    @Scheduled(cron="${time.cron}")
    void testPlaceholder1() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }

    @Scheduled(cron="*/${time.interval} * * * * *")
    void testPlaceholder2() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }
```
### 2. zone
时区，接收一个java.util.TimeZone#ID。cron表达式会基于该时区解析。默认是一个空字符串，即取服务器所在地的时区。比如我们一般使用的时区Asia/Shanghai。该字段我们一般留空。

### 3. fixedDelay
上一次执行完毕时间点之后多长时间再执行。如：
```java
@Scheduled(fixedDelay = 5 * 1000) //上一次执行完毕时间点之后5秒再执行
```
### 4. fixedDelayString
与 3. fixedDelay 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。如：
```java
@Scheduled(fixedDelayString = "5000") //上一次执行完毕时间点之后5秒再执行
```
占位符的使用(配置文件中有配置：time.fixedDelay=5000)：
```java
    @Scheduled(fixedDelayString = "${time.fixedDelay}")
    void testFixedDelayString() {
        System.out.println("Execute at " + System.currentTimeMillis());
    }
```
### 5. fixedRate
上一次开始执行时间点之后多长时间再执行。如：
```java
@Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行
```
### 6. fixedRateString
与 5. fixedRate 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。
### 7. initialDelay
第一次延迟多长时间后再执行。如：
```java
@Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
```
### 8. initialDelayString
与 7. initialDelay 意思相同，只是使用字符串的形式。唯一不同的是支持占位符。