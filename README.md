# ansible-windows-talk
Talk materials, links, snippets and a simple ansible playbook as showcase

Link to the Slides: [Running Spring Boot Apps on Docker Windows Containers with Ansible.pdf](https://github.com/jonashackt/ansible-windows-talk/blob/master/Running%20Spring%20Boot%20Apps%20on%20Docker%20Windows%20Containers%20with%20Ansible.pdf)

### Repositories needed throughout the talk

https://github.com/jonashackt/ansible-windows-docker-springboot

https://github.com/jonashackt/cxf-spring-cloud-netflix-docker

### Links to dive in deeper

https://blog.codecentric.de/en/2017/01/ansible-windows-spring-boot/

https://blog.codecentric.de/en/2017/04/ansible-docker-windows-containers-spring-boot/

https://blog.codecentric.de/en/2017/05/ansible-docker-windows-containers-scaling-spring-cloud-netflix-docker-compose/

https://www.heise.de/developer/artikel/Docker-Windows-Container-mit-Ansible-managen-1-2-3824736.html

https://www.heise.de/developer/artikel/Docker-Windows-Container-mit-Ansible-managen-2-2-3838642.html

https://docs.microsoft.com/en-us/virtualization/windowscontainers/index

https://blog.codecentric.de/en/2017/09/taming-hybrid-swarm-init-mixed-os-docker-swarm-vagrant-ansible/

# Talk snippets

## Prerequisites (1. Windows box)

> This is a step that requires to download ~10GB and takes quite long to complete - if you want to follow the demo live @ the talk, you should prepare the hole step at home.

#### a) install Virtual Box, Vagrant and Packer

###### if you´re on a Mac, this can easily be accomplished via [brew](https://brew.sh/index_de.html)
* `brew cask install virtualbox` 
* `brew cask install vagrant`
* `brew install packer`
* (Java part only) `brew install maven`

###### on Windows, take [chocolatey](https://chocolatey.org/)
* `choco install virtualbox`
* `choco install vagrant`
* `choco install packer`
* (Java part only) `choco install maven` 

###### on Linux - just use your favourite package manager

#### b) Download ISO

https://www.microsoft.com/de-de/evalcenter/evaluate-windows-server-2016 (registration needed)

#### c) Build your Vagrant Box with Packer

Clone this GitHub repo [ansible-windows-docker-springboot](https://github.com/jonashackt/ansible-windows-docker-springboot), cd into it and the subfolder `step0-packer-windows-vagrantbox`. Then run:

```
packer build -var iso_url=14393.0.161119-1705.RS1_REFRESH_SERVER_EVAL_X64FRE_EN-US.ISO -var iso_checksum=70721288bbcdfe3239d8f8c0fae55f1f windows_server_2016_docker.json
```

#### d) Add the Vagrant box and run it
```
vagrant init windows_2016_docker_virtualbox.box 
```

Now fire up your Windows Server 2016 box:
```
vagrant up
```

> You can check if everything is ok as a last step if you cd into [ansible-windows-simple](https://github.com/jonashackt/ansible-windows-docker-springboot/tree/master/step0-packer-windows-vagrantbox/ansible-windows-simple) and run a `ansible windows-dev -i hostsfile -m win_ping` - which should give an `SUCCESS` 

Find more info here: https://github.com/jonashackt/ansible-windows-docker-springboot#build-your-windows-server-2016-vagrant-box


## 2. Ansible provisions Windows

cd into [ansible-windows-simple](https://github.com/jonashackt/ansible-windows-docker-springboot/tree/master/step0-packer-windows-vagrantbox/ansible-windows-simple) and test the connection first:

```
ansible windows-dev -i hostsfile -m win_ping
```

Then run the playbook:

```
ansible-playbook -i hostsfile windows-playbook.yml --extra-vars "host=windows-dev"
```


## 3. Prepare Docker on Windows

cd into [step1-prepare-docker-windows](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step1-prepare-docker-windows/)

```
ansible-playbook -i hostsfile prepare-docker-windows.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

```
docker run --name dotnetbot microsoft/dotnet-samples:dotnetapp-nanoserver
```

## 4. Run Spring Boot App on Docker Windows Container

Clone example application´s repository [cxf-spring-cloud-netflix-docker](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker), cd into [weatherbackend](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/tree/master/weatherbackend) & do a:
```
mvn clean package
```

Then cd into weatherbackend/target

```
java -jar weatherbackend-0.0.1-SNAPSHOT.jar
```

Go to [localhost:8090/swagger-ui.html](http://localhost:8090/swagger-ui.html) and do a __GET__ onto __weather-backend-controller__ `/weather/{name}`

cd into [step2-single-spring-boot-app](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step2-single-spring-boot-app/) and run the playbook:

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev app_name=weatherbackend jar_input_path=../../cxf-spring-cloud-netflix-docker/weatherbackend/target/weatherbackend-0.0.1-SNAPSHOT.jar"
```

#### Inside the Vagrant Windows box:

###### Check running webserver inside container

Show running Docker containers:

```
docker ps -a 
```

Show logs of running container:

```
docker logs simpleapp_weatherbackend
```

Connect into running container with Powershell:

```
docker exec -it simpleapp_weatherbackend powershell
```

Show some environment variables:

```
Get-ChildItem Env:
```

Call webserver:
```
iwr http://localhost:8090/swagger-ui.html -UseBasicParsing
```

###### Call webserver from Windows Docker Host:

Find container´s IP:

```
docker network inspect nat
```

Go to [containerIP:8088/swagger-ui.html](http://containerIP:8088/swagger-ui.html) and try it out again!



## 5. Scale Spring Boot Apps

Example project [cxf-spring-cloud-netflix-docker](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker)

More info on this blog post https://blog.codecentric.de/en/2017/05/ansible-docker-windows-containers-scaling-spring-cloud-netflix-docker-compose/

#### Run playbook

cd into [step3-multiple-spring-boot-apps-docker-compose](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step3-multiple-spring-boot-apps-docker-compose/)

```
ansible-playbook -i hostsfile ansible-windows-docker-springboot.yml --extra-vars "host=ansible-windows-docker-springboot-dev"
```

Show running Docker containers:

```
docker ps -a 
```

#### Eureka - Service Registry

Look for eureka-serviceregistry Container´s IP

```
docker network inspect nat
```

and go to [eurekaIP:8761](http://eurekaIP:8761)


#### Zuul - Dynamic Proxy & Routing

Go to http://localhost:48080/routes on your VirtualBox Host machine


#### Scaling

Inside the Vagrant Box, on Powershell cd into `c:\springboot` and run:

```
docker-compose scale weatherbackend=3
```


#### Testing the complete route - with the weatherclient app

cd into [weatherclient](https://github.com/jonashackt/cxf-spring-cloud-netflix-docker/tree/master/weatherclient) and run

```
java -jar target/weatherclient-0.0.1-SNAPSHOT.jar
```

Inside the Vagrant Windows Box: Open 2 Powershells and connect to the logs:

```
docker logs springboot_weatherservice_1 --follow
docker logs springboot_weatherbackend_1 --follow
```

On your Computer:

Go to [localhost:8087/swagger-ui.html](http://localhost:8087/swagger-ui.html) and do a __GET__ on `/forecast/{zip}`


## X. One more thing...

#### Prepare:

cd into [step4-windows-linux-multimachine-vagrant-docker-swarm-setup](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step4-windows-linux-multimachine-vagrant-docker-swarm-setup/) and run

```
ansible-playbook -i hostsfile prepare-docker-nodes.yml

ansible-playbook -i hostsfile initialize-docker-swarm.yml
```

Have a look into Portainer UI on your Vagrant host:

http://localhost:49000/


#### Deploy Apps:

cd into [step5-deploy-multiple-spring-boot-apps-to-mixed-os-docker-swarm](https://github.com/jonashackt/ansible-windows-docker-springboot/blob/master/step5-deploy-multiple-spring-boot-apps-to-mixed-os-docker-swarm/) and run

```
ansible-playbook -i hostsfile build-and-deploy-apps-2-swarm.yml
```

Now have a look into the Traefik UI:

http://localhost:48080/dashboard/

All the services should be available through Docker Swarm routing mesh / ingress networking - on your Vagrant host:

__weatherbackend__: http://localhost:8090/swagger-ui.html

__weatherservice__: http://localhost:8095/soap

__Eureka__: http://localhost:8761/

__Spring Boot Admin__: http://localhost:8092/



