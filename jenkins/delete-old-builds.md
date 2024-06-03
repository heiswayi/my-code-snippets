Delete old builds in each project except the latest 10 builds.

Use this script at **Dashboard > Manage Jenkins > Script Console**.

```groovy
import jenkins.model.*
import hudson.model.*
Jenkins.instance.getAllItems(AbstractItem.class).each { 
  def job = jenkins.model.Jenkins.instance.getItem(it.fullName)
  if (job!=null){
    println "job: " + job;  
    int i = 1;
    job.builds.each {
        def build = it;
        if (i<=10){
           println "build: " + build.displayName + " -- NOT DELETED";
        }else{
           println "build: " + build.displayName + " -- DELETED";
           it.delete(); 
        }
        i = ++i
    }
  } 
};
```