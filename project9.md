# CONTINOUS INTEGRATION PIPELINE FOR TOOLING WEBSITE

## STEP 1 - INSTALL AND CONFIGURE JENKINS SERVER
1. Create an AWS EC2 server based on Ubuntu Server 20.04 LTS and name it "Jenkins"
   
   ![](images/ec2-jenkins.png)

2. Install JDK (since Jenkins is a Java-based application)
   ```
   sudo apt update
   sudo apt install default-jdk-headless
   ```

3. Install Jenkins
   ```
   wget -q -O - https://pkg.jenkins.io/debian-stable/jenkins.io.key | sudo apt-key add -
   sudo sh -c 'echo deb https://pkg.jenkins.io/debian-stable binary/ > /etc/apt/sources.list.d/jenkins.list'
   sudo apt update
   sudo apt-get install jenkins
   ```

4. Make sure Jenkins is up and running
   ```
   sudo systemctl status jenkins
   ```
   ![](images/jenkins-running.png)

5. By default Jenkins server uses TCP port 8080 – open it by creating a new Inbound Rule in your EC2 Security Group
   ![](images/open-8080-jenkins-ec2.png)

6. Perform initial Jenkins setup.
   From your browser access http://<Jenkins-Server-Public-IP-Address-or-Public-DNS-Name>:8080
   You will be prompted to provide a default admin password

   ![](images/signin-jenkins.png)

   Retrieve it from your server:
   `sudo cat /var/lib/jenkins/secrets/initialAdminPassword`
   
   Then you will be asked which plugings to install – choose suggested plugins.

   ![](images/suggested-plugin.png)

   Once plugins installation is done – create an admin user and you will get your Jenkins server address.

   ![](images/setting%20plgin.png)

   ![](images/instance-conf.png)

   ![](images/jenkins-ready.png)
   
   The installation is completed!


## Step 2 – Configure Jenkins to retrieve source codes from GitHub using Webhooks
In this part, we will configure a simple Jenkins job/project (these two terms can be used interchangeably). This job will be triggered by GitHub webhooks and will execute a ‘build’ task to retrieve codes from GitHub and store it locally on Jenkins server.
1. Fork this repo `https://github.com/franklinobasy/tooling` and replace it with your username - `https://github.com/<Your-username>/tooling`
2. Open the repository and select the `settings` tab.
   ![](images/select-git-settings.png)
3. Select `weebhooks` from the navigation panel on the left.
   ![](images/select-git-webhook.png)
4. click `Add webhook` on the top right side
   ![](images/click-add-webhook1.png)
5. In the payload URL, paste the `<Jenkins-server-address:8080>/github-webhook`, and  in the content-type, select `application/json`.
   Then click `Add Webhook`
   ![](images/set-webhook.png)

6. Go to Jenkins web console, click "New Item" and create a "Freestyle project"
   ![](images/create%20job%20jenkins.png)

   To connect your GitHub repository, you will need to provide its URL, you can copy from the repository itself

   In configuration of your Jenkins freestyle project choose Git repository, provide there the link to your Tooling GitHub repository and credentials (user/password) so Jenkins could access files in the repository.

   ![](images/config-jenk-git.png)

   Save the configuration and let us try to run the build. For now we can only do it manually.
   Click "Build Now" button, if you have configured everything correctly, the build will be successfull and you will see it under #1

   ![](images/build1.png)

   You can open the build and check in "Console Output" if it has run successfully.

   If so – congratulations! You have just made your very first Jenkins build!

   But this build does not produce anything and it runs only when we trigger it manually. Let us fix it.

7. Click "Configure" your job/project and add these two configurations
   ![](images/configure1.png)

   Configure triggering the job from GitHub webhook:
   ![](images/configure2.png)
   ![](images/configure3.png)

   Now, go ahead and make some change in any file in your GitHub repository (e.g. README.MD file) and push the changes to the master branch.

   You will see that a new build has been launched automatically (by webhook) and you can see its results – artifacts, saved on Jenkins server.
   ![](images/build-auto.png)

   You have now configured an automated Jenkins job that receives files from GitHub by webhook trigger (this method is considered as ‘push’ because the changes are being ‘pushed’ and files transfer is initiated by GitHub). There are also other methods: trigger one job (downstreadm) from another (upstream), poll GitHub periodically and others.

   By default, the artifacts are stored on Jenkins server locally

   `ls /var/lib/jenkins/jobs/tooling_github/builds/<build_number>/archive/`

## Step 3 - Configure Jenkins to copy files to NFS server via SSH

Now we have our artifacts saved locally on Jenkins server, the next step is to copy them to our NFS server to /mnt/apps directory.

Jenkins is a highly extendable application and there are 1400+ plugins available. We will need a plugin that is called "Publish Over SSH".

1. Install "Publish Over SSH" plugin.
   
   On main dashboard select "Manage Jenkins" and choose "Manage Plugins" menu item.

   On "Available" tab search for "Publish Over SSH" plugin and install it
   
   ![](images/pos.png)

2. Configure the job/project to copy artifacts over to NFS server.
   On main dashboard select "Manage Jenkins" and choose "Configure System" menu item.

  Scroll down to Publish over SSH plugin configuration section and configure it to be able to connect to your NFS server:
  - Provide a private key (content of .pem file that you use to connect to NFS server via SSH/Putty)
  - Arbitrary name
  - Hostname – can be private IP address of your NFS server
  - Username – ec2-user (since NFS server is based on EC2 with RHEL 8)
  - Remote directory – /mnt/apps since our Web Servers use it as a mointing point to retrieve files from the NFS server
  
  ![](images/pos2.png)

  Test the configuration and make sure the connection returns Success. Remember, that TCP port 22 on NFS server must be open to receive SSH connections.

  Save the configuration, open your Jenkins job/project configuration page and add another one "Post-build Action"

  ![](images/configure4.png)

  Configure it to send all files probuced by the build into our previouslys define remote directory. In our case we want to copy all files and directories – so we use **.
  If you want to apply some particular pattern to define which files to send – [use this syntax](http://ant.apache.org/manual/dirtasks.html#patterns).

  ![](images/configure5.png)

  Save this configuration and go ahead, change something in README.MD file in your GitHub Tooling repository.

  Webhook will trigger a new job and in the "Console Output" of the job you will find something like this:

  ```
  SSH: Transferred 25 file(s)
  Finished: SUCCESS
  ```

  To make sure that the files in /mnt/apps have been udated – connect via SSH/Putty to your NFS server and check README.MD file

  ```
  cat /mnt/apps/README.md
  ```
  If you see the changes you had previously made in your GitHub – the job works as expected.

