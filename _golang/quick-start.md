---
title: "Quick Start of Go-chassis"
lang: en
ref: quick-start
permalink: /golang/quick-start/
excerpt: "Introduce how to run microservice applications using ServiceComb Go-chassis"
last_modified_at: 2017-09-06T00:50:43-55:00
---

{% include toc %}

## Install Golang development environment

* Install Git, details can refer to [Git Installing Guide](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git){:target="_blank"}.

* Install Golang 1.8+, details can refer to [JDK Installing Guide](https://docs.oracle.com/javase/8/docs/technotes/guides/install/install_overview.html){:target="_blank"}.

* Install Docker , details can refer to [Maven Installing Guide](https://maven.apache.org/install.html){:target="_blank"}.

* Install ServiceComb Go-chassis(SDK) by executing the following commands.

   ```bash
   mkdir -p $GOPATH/src/github.com/ServiceComb
   cd $GOPATH/src/github.com/ServiceComb
   git clone https://github.com/ServiceComb/go-chassis.git
   ```
   
## Run Service Center
**Service Center** enables capabilities of service registration and service discovery in **ServiceComb**. It can run inside docker.
```bash
docker pull servicecomb/service-center
docker run -d -p 30100:30100 servicecomb/service-center:latest
```
*Reference to [service center deployment](/users/setup-environment/#运行service-center) to learn deploying Service Center as a local binary.*   


## Create your first microservice application
Let\'s start the journey of microservice with a simple **Body Mass Index(BMI)** application. The [BMI](https://en.wikipedia.org/wiki/Body_mass_index){:target="_blank"} is an attempt to quantify the amount of tissue mass in an individual. This application contains two separate microservices:

* **BMI calculator service**：A microservice provides calculation of BMI.

* **Web service**：A microservice provides both user interface and gateway service.

The workflow of the application is shown as follows:  
![workflow of BMI](/assets/images/quick-start-sample-workflow-en.png){: .align-center}

Note that the dotted lines indicate the process of service registration and discovery.

## Run microservice application


1. Get source code of BMI example and enter BMI codes directory.

   ```bash
   mkdir -p $GOPATH/src/github.com/ServiceComb-samples
   cd $GOPATH/src/github.com/ServiceComb-samples
   git clone https://github.com/ServiceComb-samples/go-bmi.git
   cd go-bmi
   ```

   **Note**: In windows development environment, the docker runs inside the virtual machine. The IP address of **Service Center** needs to be modified as the virtual machine\'s IP address. Modify the 2 configuration files [calculator\|webapp]/src/main/resources/microservice.yaml, changed <a>http://127.0.0.1:30100</a> to <a>http://192.168.99.100:30100</a> , where 192.168.99.100 is the virtual machine\'s IP address.

2. Run microservices.

   ```bash
   cd calculator; go build; ./calculator
   cd web-app; go build; ./web-app
   ```

3. Verify the application. Visit <a>http://localhost:8889</a> in browser. Then input your height and weight to verify.

   ![BMI user interface](/assets/images/bmi-interface.png){: .align-center}

## What's next

* Learn how to [develop microservice application in minutes](/golang/quick-start-bmi/).


