
## SonarQube Integration with Jenkins

Integration SonarQube server with Jenkins is necessary to store your reports. Follow below steps to enable that.

### Prerequisites
1. SonarQube Server **[Get Help here](https://github.com/coachzee/sonarqube/blob/main/sonarqube.MD)**
1. Jenkins Server 

### Implementation

Login to Jenkins server and install sonarqube scanner. 

- SonarQube scanner URL : https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner
- Package : https://docs.sonarqube.org/latest/analysis/scan/sonarscanner-for-jenkins/

Go to SonarQube server
```sh
# cd /opt
# wget https://binaries.sonarsource.com/Distribution/sonar-scanner-cli/sonar-scanner-cli-4.6.0.2311-linux.zip
# unzip sonar-scanner-cli-4.6.0.2311-linux.zip
# mv sonar-scanner-4.6.0.2311-linux sonar-scanner 
```

Set SonarQube server details in sonar-scanner property file 

 - Sonar properties file: /opt/sonar_scanner/conf/sonar-scanner.properties
 
 ```sh
# vim /opt/sonar_scanner/conf/sonar-scanner.properties

Note: uncomment Sonar.host line and add IP address

sonar.host.url=http://`<SONAR_SERVER_IP>`:9000 # E.g sonar.host.url=http://10.10.10.2:9000
```

Login to Jenkins GUI console and install " SonarQube scanner" plugin

 - `Manage Jenkins` > `Manage Plugins` > `Avalable` > `SonarQube scanner` 

Configure SonarQube scanner home path

- `Manage Jenkins` > `Global Tool Configuration` > `SonarQube servers` > `Add SonarQube` 
   - Name  : `sonar-scanner`
   - SONAR_RUNNER_HOME : `/opt/sonar-scanner`

Configure SonarQube server name and authentication token 
- `Manage Jenkins` > `Configure Systems` > `SonarQube Servers`
    - Name : `SonarQube`
	- ServerURL : `http://<Sonarqube_server>:9000/sonar`
	- Server `authentication token`
To Get Authentication code follow below steps.
	Login to SonarQube server as a admin  `My Account` > `Security` > `Generate Token`

Create a job to test SonarQube. Provide below sonar properties details in the job under build 
- Build:
  - `Execute SonarQube Scanner` > `Analysis properties`  (it is mandatary).  

```sh 
# Required parameters
sonar.projectKey=Coachy
sonar.projectName=CoachyDemo
sonar.projectVersion=1.0

# Artifactory directory
sonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/SimpleCustomerApp/src

-
Note: sonar.sources=/var/lib/jenkins/workspace/$JOB_NAME/<PROJECT_NAME>/src  
```

Execute job to get analysis report. 
