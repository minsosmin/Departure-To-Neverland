# [官方文档](https://jenkins.io/zh/doc/#doc/developer#)

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
  
  
