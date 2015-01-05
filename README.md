#### Developer Quickstart on HDP Sandbox using Ambari Stacks
An Ambari Stack service package for VNC Server with the ability to install Eclipse as well to quickly start developing on HDP Hadoop

- Download HDP 2.2 sandbox VM image (Sandbox_HDP_2.2_VMware.ova) from [Hortonworks website](http://hortonworks.com/products/hortonworks-sandbox/)
- Import Sandbox_HDP_2.2_VMware.ova into VMWare and set the VM memory size to 8GB
- Now start the VM
- After it boots up, find the IP address of the VM and add an entry into your machines hosts file e.g.
```
192.168.191.241 sandbox.hortonworks.com sandbox    
```
- Connect to the VM via SSH (password hadoop) and start Ambari server
```
ssh root@sandbox.hortonworks.com
/root/start_ambari.sh
```

- To deploy the VNC stack, run below
```
cd /var/lib/ambari-server/resources/stacks/HDP/2.2/services
git clone https://github.com/abajwa-hw/vnc-stack.git   
sudo service ambari restart
```
- Then you can click on 'Add Service' from the 'Actions' dropdown menu in the bottom left of the Ambari dashboard:

On bottom left -> Actions -> Add service -> check VNC Server -> Next -> Next -> Enter password -> Next -> Deploy
![Image](../master/screenshots/screenshot-vnc-config.png?raw=true)

On successfull deployment you will see the VNC service as part of Ambari stack and will be able to start/stop the service from here:
![Image](../master/screenshots/screenshot-vnc-stack.png?raw=true)

When you've completed the install process, VNC server will be available at your VM's IP on display 1 with the password you setup
![Image](../master/screenshots/screenshot-vnc-clientsetup.png?raw=true)

On logging in you will see the CentOS desktop running on the sandbox
![Image](../master/screenshots/screenshot-vnc-clientlogin.png?raw=true)

Click the eclipse shortcut to start Eclipse
![Image](../master/screenshots/screenshot-vnc-eclipsestarted.png?raw=true)

- To remove the VNC service: 
  - Stop the service via Ambari
  - Delete the service
```
curl -u admin:admin -i -H 'X-Requested-By: ambari' -X DELETE http://sandbox.hortonworks.com:8080/api/v1/clusters/Sandbox/services/VNC
```
  - Remove artifacts 
```
/var/lib/ambari-server/resources/stacks/HDP/2.2/services/vnc-stack/remove.sh
```

- As a next step, try setting up the Twitter storm topology from [here](https://github.com/abajwa-hw/hdp22-hive-streaming#step-4-import-tweets-for-users-into-hive-orc-table-via-storm)
You can get the sample code by running "git clone" from your repo (git already installed on sandbox)
```
cd /root
git clone https://github.com/abajwa-hw/hdp22-hive-streaming.git 
```

- Once you already have your storm code on the VM, just import the dir containing the pom.xml into Eclipse:
File > Import > Existing Maven Projects > navigate to your code (e.g. /root/hdp22-hive-streaming)  > OK

- Check the java compiler is using 1.7:
File > Properties > Java Compiler > uncheck "use compliance from..." > set "Compiler compliance level" to 1.7 > OK

- The eclipse project should build on its own and not show errors (if not, you may need to add jars to the project properties)

- To run maven compile: Run > Run as > Maven Build
  - The first time you do this, it will ask you for the configuration:
    - Under ‘Goals’: clean install
    - Under Maven Runtime, add your existing mvn install on the sandbox (its faster than using the embedded one)
    - Configure > Add > click ‘Directory’ and navigate to the dir where you installed mvn (e.g. /usr/share/maven/latest)
    
- Eclipse should now be able to run a mvn compile and create the uber jar

- Now from terminal, run your topology:
```
cd /root/hdp22-hive-streaming
storm jar ./target/storm-test-1.0-SNAPSHOT.jar test.HiveTopology
```    