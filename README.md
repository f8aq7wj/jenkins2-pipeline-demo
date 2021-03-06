#Jenkins 2.0 Cloud Foundry Pipeline Demo

Demonstrates a Jenkins 2.0 Pipeline to build a Java project and deploy the project to a running instance of Pivotal Cloud Foundry.

## To Run this Demo

Simply clone this project:
```
git clone https://github.com/pivotalservices/jenkins2-pipeline-demo
cd jenkins2-pipeline-demo
```

Assumes a running Docker Machine at IP 192.168.99.100, and that you have run through the initial Jenkins 2.0 setup.

To start the Docker Machine and to pull the jenkinsci/jenkins Docker image, first run these commands:
```
## Your container will mount the jenkins home volume using this directory
docker-machine create jenkins2-pipeline-demo --driver virtualbox --virtualbox-memory "11000" --virtualbox-disk-size "100000"
eval "$(docker-machine env jenkins2-pipeline-demo)"
docker pull dmalone/jenkins2-cf-pipeline-demo or docker build -t dmalone/jenkins2-cf-pipeline-demo - < Dockerfile
docker run -i -t -p 8080:8080 -p 50000:50000 --name=jenkins-pipeline-demo -v /var/jenkins_home dmalone/jenkins2-cf-pipeline-demo
```

The default admin password will appear in the console as logs from Jenkins. This password is required to run through the initial setup of Jenkins. You can also obtain the Jenkins Admin password by executing the following: `docker exec -it $(docker ps -l -q) cat /var/jenkins_home/secrets/initialAdminPassword` where ``docker ps -l -q`` will retrieve the last container ID.

After this is complete, then you can create the Demo Pipeline Job using the CLI:

```
curl -O http://192.168.99.100:8080/jnlpJars/jenkins-cli.jar
export JENKINS_URL=http://192.168.99.100:8080/
java -jar jenkins-cli.jar create-job sample-spring-cloud-svc-ci < config.xml --username admin --password <REPLACE-WITH-PASSWORD-ABOVE>
```

Navigate to the IP of the Docker VM and port 8080 to access Jenkins. The pipeline created using the above script should be located at http://192.168.99.100:8080/job/sample-spring-cloud-svc-ci


## To Build this project

This project builds a custom Docker container with Jenkins plugins automatically installed, and the CF CLI automatically installed and configured so it can be used by Pipelines.

`build -t dmalone/jenkins2-cf-pipeline-demo .`

## To Build Cloudbees version of this project

`build -f Dockerfile_cloudbees -t dmalone/cb-cf-pipeline-demo .`

### Initilization scripts (Groovy init scripts that run on Jenkins startup)
- `init_02_pull_remote_pipeline_global_libs.groovy`: Pulls Pipeline Global Library from a GitHub repo into the Jenkins Pipeline repo, making the shared Pipeline Libraries available immediately after initial Jenkins startup
- `init_03_add_sa_credentials.groovy`: Sets up global credentials, retrieving sensitive data from environmental variables; includes credential to connect to GitHub Enterprise
- `init_04_add_ghe_server.groovy`: Configures GitHub Enterprise API endpoint to be used with the GitHub Organization Folder plugin
- `init_05_mail_server_config.groovy`: Configures mail server to allow sending email from Jenkins
- `init_08_audit_trail_config.groovy`: Configures Audit Trail plugin to send data to Logstash via syslog; and on to Elasticsearch to be displayed in custom CJOC analytics dashboard
- `init_20_top-level-folders.groovy`: Creates a folder with a custom icon from the custom [Simple Build plugin](https://github.com/kmadel/simple-build-for-pipeline-plugin), used to hold example job(s)
- `init_21_simple_build_pipeline.groovy`: Creates an example Pipeline job that uses DSL from the [Simple Build plugin](https://github.com/kmadel/simple-build-for-pipeline-plugin)
- `init_99_save.groovy`: Ensures any previous configuration changes are saved  


## References

* Docker Machine Installation Instructions: https://docs.docker.com/machine/install-machine/
* Jenkins Docker Image README:  https://github.com/jenkinsci/docker/blob/master/README.md
