# ansible-windows-talk
Talk materials, links, snippets and a simple ansible playbook as showcase


# Talk snippets

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
```
ansible windows-dev -i hostsfile -m win_ping
```

```
ansible-playbook -i hostsfile windows-playbook.yml --extra-vars "host=windows-dev"
```


### 3. Prepare Docker on Windows
```
docker run --name dotnetbot-container microsoft/dotnet-samples:dotnetapp-nanoserver
```