# ansible-windows-talk
Talk materials, links, snippets and a simple ansible playbook as showcase

### Repositories needed throughout the talk

https://github.com/jonashackt/ansible-windows-docker-springboot

https://github.com/jonashackt/cxf-spring-cloud-netflix-docker

### Links to dive in deeper

https://blog.codecentric.de/en/2017/01/ansible-windows-spring-boot/

https://blog.codecentric.de/en/2017/04/ansible-docker-windows-containers-spring-boot/

https://blog.codecentric.de/en/2017/05/ansible-docker-windows-containers-scaling-spring-cloud-netflix-docker-compose/

https://docs.microsoft.com/en-us/virtualization/windowscontainers/index

# Talk snippets

### Prerequisites

install Virtual Box, Vagrant and Packer

__if you´re on a Mac, this can easily be accomplished via brew:__
* `brew cask install virtualbox` 
* `brew cask install vagrant`
* `brew install packer`

__Windows:__
* `choco install virtualbox`
* `choco install vagrant`
* `choco install packer` 


### 1. Windows box
```
(Get-ItemProperty -Path c:\windows\system32\hal.dll).VersionInfo.FileVersion
```

ISO: https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2016

Packer configuration json, Vagrant template: https://github.com/jonashackt/ansible-windows-docker-springboot/tree/master/step0-packer-windows-vagrantbox

```
packer build -var iso_url=14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO -var iso_checksum=70721288bbcdfe3239d8f8c0fae55f1f windows_server_2016_docker.json
```

Add the box and run it:
```
vagrant init windows_2016_docker_virtualbox.box 
```

Now fire up your Windows Server 2016 box:
```
vagrant up
```

https://github.com/jonashackt/ansible-windows-docker-springboot#build-your-windows-server-2016-vagrant-box


### 2. Ansible provisions Windows

cd into [ansible-windows-simple](https://github.com/jonashackt/ansible-windows-talk/tree/master/ansible-windows-simple) and run the playbook:

```
ansible windows-dev -i hostsfile -m win_ping
```

```
ansible-playbook -i hostsfile windows-playbook.yml --extra-vars "host=windows-dev"
```


### 3. Prepare Docker on Windows

All the preparing playbooks can be found here: [step1-prepare-docker-windows](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step1-prepare-docker-windows/)

```
ansible-playbook -i hostsfile prepare-docker-windows.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

```
docker run --name dotnetbot microsoft/dotnet-samples:dotnetapp-nanoserver
```

### 4. Run Spring Boot App on Docker Windows Container

Clone simple Spring Boot app [weatherbackend](https://github.com/jonashackt/spring-cloud-netflix-docker/tree/master/weatherbackend) & do a:
```
mvn clean package
```

Then cd into weatherbackend/target

```
java -jar weatherbackend-0.0.1-SNAPSHOT.jar
```

cd into [step2-single-spring-boot-app](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step2-single-spring-boot-app/) and run the playbook:

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev app_name=weatherbackend jar_input_path=../../cxf-spring-cloud-netflix-docker/weatherbackend/target/weatherbackend-0.0.1-SNAPSHOT.jar"
```

### 5. Scale Spring Boot Apps

Example project [cxf-spring-cloud-netflix-docker](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker)

Spring Cloud docs: http://cloud.spring.io/spring-cloud-static/Dalston.RELEASE/

###### Zuul dynamic routing

zuul-edgeservice [application.yml](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/blob/master/zuul-edgeservice/src/main/resources/application.yml)

###### Run playbook

cd into [step3-multiple-spring-boot-apps-docker-compose](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step3-multiple-spring-boot-apps-docker-compose/)

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

###### Healthcheck

Spring Boot/Cloud [about well written clients](https://stackoverflow.com/a/42352258/4964553) 





