# ansible-windows-simple
Talk materials, links, snippets and a simple ansible playbook as showcase


### Talk snippets

##### 1. Windows box

`(Get-ItemProperty -Path c:\windows\system32\hal.dll).VersionInfo.FileVersion`

https://github.com/jonashackt/ansible-windows-docker-springboot#build-your-windows-server-2016-vagrant-box


##### 2. Ansible provisions Windows

`ansible windows-dev -i hostsfile -m win_ping`

`ansible-playbook -i hostsfile windows-playbook.yml --extra-vars "host=windows-dev"`


##### 3. Prepare Docker on Windows

`docker run --name dotnetbot-container microsoft/dotnet-samples:dotnetapp-nanoserver`