# [官方文档](https://jenkins.io/zh/doc/#doc/developer#)

 - dependency
 ```xml
  <!-- https://github.com/jenkinsci/java-client-api -->
  <dependency>
    <groupId>com.offbytwo.jenkins</groupId>
    <artifactId>jenkins-client</artifactId>
    <version>0.3.8</version>
  </dependency>
 ```

### 利用 Java 操作 Jenkins API 实现对 Jenkins 的控制详解 [dalao-blob](http://www.mydlq.club/article/23/)
  - Job 相关
  ```java
  Job job = new Job();
  job.build();

  Map<String, Job> jobs = jenkins.getJobs();
  
  // 无参数 Job build
  jenkins.getJob("eos-health-manager").build();
  
  JobWithDetails job = jobs.get("eos-health-manager").details();
  // 即将执行任务的jobId
  int buildNumber = job.getNextBuildNumber();
  // 带参数 Job build
  HashMap<String,String> params = null;
  params.put("GIT_URL","");
  params.put("GIT_BRANCH","");
  params.put("IMAGE_NAME","");
  params.put("IMAGE_VERSION","1.0.2");
  params.put("DOCKER_TYPE","");
  params.put("K8S_TYPE","");
  params.put("REPLICAS","");
  params.put("NODE","");
  params.put("CPU","");
  params.put("MEMORY","");
  job.build(params);
  
  // 停止最后构建的 Job Build
  // 获取最后的 build 信息
  Build build = jenkins.getJob("eos-health-manager").getLastBuild();
  // 停止最后的 build
  build.Stop();

   // 更新 Job
  String jobXml = "";
  jenkins.updateJob("eos-health-manager",jobXml);

  // 删除 Job
   jenkins.deleteJob("eos-health-manager");

   // 禁用 Job
  jenkins.disableJob("eos-health-manager");

  // 启用 Job
  jenkins.enableJob("eos-health-manager");

  // 是否执行结束 jobId
  boolean isBuilding = job.getBuildByNumber(jobId).details().isBuilding();

  // 获取 Maven Job 信息
  MavenJobWithDetails mavenJobWithDetails = jenkins.getMavenJob("eos-health-manager");

  // 根据 View 名称获取 Job 列表
  Map<String,Job> viewJobs = jenkins.getJobs("all");	
  ```
  - Build 相关
  ```java
  JobWithDetails jobWithDetails = jobs.get(jobName).details();
  # 首次编译信息
  Build firstBuild = jobWithDetails.getFirstBuild();
  # 最后编译信息
  Build lastBuild = jobWithDetails.getLastBuild();
  # 根据 Job Build 编号获取编译信息
  Build numberBuild = jobWithDetails.getBuildByNumber(1);
  # 全部 Build 信息
  List<Build> builds = jobWithDetails.getAllBuilds();
  # 一定范围的 Build 列表
  Range range = Range.build().from(1).to(2); // 设定范围
  List<Build> builds = job.getAllBuilds(range);
  for (Build build : builds){
    build.getUrl(); // 构建的 URL 地址
    build.getNumber();// 构建编号         
    build.getTestReport();// 获取测试报告       
    build.getTestResult();// 获取测试结果
  }
  # 上次成功
  Build lastSuccessfulBuild = JobWithDetails.getLastSuccessfulBuild();
  # 上次失败
  Build lastFailedBuild = JobWithDetails.getLastFailedBuild();
  # 上次持续时间
  Build lastBuild = JobWithDetails.getLastBuild().getDuration();

  # 构建的显示名称
  buildWithDetails.getDisplayName();
  # 构建的参数信息
  buildWithDetails.getParameters();
  # 构建编号
  buildWithDetails.getNumber();
  # 构建结果，如果构建未完成则会显示为 null
  buildWithDetails.getResult();
  # 构建的活动信息
  buildWithDetails.getActions();
  # 构建持续多少时间(ms)
  buildWithDetails.getDuration();
  # 构建开始时间戳
  buildWithDetails.getTimestamp();
  # 构建头信息，里面包含构建的用户，上游信息，时间戳等
  List<BuildCause> buildCauses = buildWithDetails.getCauses();
  for (BuildCause bc : buildCauses){
      bc.getUserId();
      bc.getShortDescription();
      bc.getUpstreamBuild();
      bc.getUpstreamProject();
      bc.getUpstreamUrl();
      bc.getUserName();
  }
  # Text格式日志
  buildWithDetails.getConsoleOutputText();
  # Html格式日志
  buildWithDetails.getConsoleOutputHtml();
  # 部分日志,一般用于正在执行构建的任务
  ConsoleLog consoleLog = buildWithDetails.getConsoleOutputText(0);	
  # 当前日志大小
  Integer currentBufferSize = consoleLog.getCurrentBufferSize();		
  # 是否已经构建完成，还有更多日志信息	
  Boolean hasMoreData = consoleLog.getHasMoreData();
  # 当前截取的日志信息
  String consoleLog = consoleLog.getConsoleLog();

  /**
   * 获取正在执行构建任务的日志信息
   */
  public void getBuildActiveLog(){
    try {
      // 这里用最后一次编译来示例
      BuildWithDetails build = jenkinsServer.getJob("test-job").getLastBuild().details();
      // 当前日志
      ConsoleLog currentLog = build.getConsoleOutputText(0);
      // 输出当前获取日志信息
      System.out.println(currentLog.getConsoleLog());
      // 检测是否还有更多日志,如果是则继续循环获取
      while (currentLog.getHasMoreData()){
        // 获取最新日志信息
        ConsoleLog newLog = build.getConsoleOutputText(currentLog.getCurrentBufferSize());
        // 输出最新日志
        System.out.println(newLog.getConsoleLog());
        currentLog = newLog;
        // 睡眠1s
        Thread.sleep(1000);
      }
    }catch (IOException | InterruptedException e) {
      e.printStackTrace();
    }
  }
  ```
  
  - 相关参数
  ```java
    private List<LinkedHashMap<String, List<LinkedHashMap<String, Object>>>> actions; // TODO: Should be improved.
    
    long timestamp = details.getTimestamp();
    Timestamp ts = new Timestamp(timestamp);
    buildDetailsHistoryV.setTimestamp(ts.toString());
    
    long duration = details.getDuration();
    String durationStr = getDurationStr(duration);
    private String getDurationStr(long duration) {

        Date date = new Date(duration);
        Instant instant = date.toInstant();
        //ZoneId zone = ZoneId.systemDefault();
        ZoneId zone = ZoneId.of("GMT");
        LocalDateTime localDateTime = LocalDateTime.ofInstant(instant, zone);
        LocalTime localTime = localDateTime.toLocalTime();
        int hour = localTime.getHour();
        int minute = localTime.getMinute();
        int second = localTime.getSecond();

        if (hour == 0) {
            if (minute == 0) {
                if (second == 0) {
                    return "小于 1 秒";
                }
                return String.format("%d 秒", second);
            } else {
                return String.format("%d 分 %d 秒", minute, second);
            }
        } else {
            if (minute == 0) {
                return String.format("%d 小时 %d 秒", hour, second);
            } else {
                return String.format("%d 小时 %d 分 %d 秒", hour, minute, second);
            }
        }

  }
  ```
  
