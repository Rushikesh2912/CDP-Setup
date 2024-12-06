# CDP-Setup

To launch a Red Hat 8 EC2 instance and install Cloudera Data Platform (CDP), follow the steps below. This guide will cover the EC2 instance creation, prerequisite installation, and the process to install CDP using the installer and runtime configuration.

---

### **Step 1: Launch Red Hat 8 EC2 Instance**

1. **Log into AWS Console:**
   - Go to the [AWS EC2 console](https://console.aws.amazon.com/ec2/).
   
2. **Launch a New EC2 Instance:**
   - Choose **Launch Instance**.
   - Select the **Red Hat Enterprise Linux 8** AMI.
     - AMI ID: `ami-0619404f9180a28b3`
   - Select the instance type, configure your instance settings, and create or select an existing key pair for SSH access.

3. **Security Groups:**
   - Open necessary ports:
     - **SSH** (port 22) for remote access.
     - **Cloudera ports** (e.g., 7180 for CM Web UI).

4. **Launch the Instance:**
   - Once everything is configured, launch your instance.

---

### **Step 2: Install Prerequisites**

1. **SSH into Your EC2 Instance:**
   - Once your instance is running, SSH into it:
     ```bash
     ssh -i your-key.pem ec2-user@<instance-public-ip>
     ```

2. **Install wget:**
   Install `wget` to download the Cloudera Manager installer:
   ```bash
   sudo yum install wget -y
   ```

3. **Check vm.swappiness:**
   ```bash
   sudo sysctl -a | grep vm.swappiness
   ```

4. **Set Swappiness:**
   Set the `vm.swappiness` value to `1`:
   ```bash
   sudo su -c 'cat >>/etc/sysctl.conf <<EOL
   vm.swappiness=1
   EOL'
   sudo sysctl -p
   ```

5. **Disable Transparent HugePages:**
   Disable Transparent HugePages (THP) by adding the following lines:
   ```bash
   echo "echo never > /sys/kernel/mm/transparent_hugepage/defrag" | sudo tee -a /etc/rc.d/rc.local
   echo "echo never > /sys/kernel/mm/transparent_hugepage/enabled" | sudo tee -a /etc/rc.d/rc.local
   ```

6. **Make rc.local Executable:**
   ```bash
   sudo chmod +x /etc/rc.d/rc.local
   ```

7. **Check Transparent HugePages Configuration:**
   Verify that Transparent HugePages are disabled:
   ```bash
   cat /sys/kernel/mm/transparent_hugepage/enabled
   cat /sys/kernel/mm/transparent_hugepage/defrag
   ```

8. **Disable SELinux:**
   Set SELinux to disabled:
   ```bash
   echo "SELINUX=disabled" | sudo tee /etc/selinux/config
   sudo sestatus
   ```

9. **Configure SSH Keys:**
   Disable strict host key checking and generate SSH keys:
   ```bash
   sudo su -c "touch /home/centos/.ssh/config; echo -e 'Host *\n  StrictHostKeyChecking no\n  UserKnownHostsFile=/dev/null' >> /home/centos/.ssh/config"
   echo -e  'y\n'| ssh-keygen -t rsa -P "" -f $HOME/.ssh/id_rsa
   cat $HOME/.ssh/id_rsa.pub >> $HOME/.ssh/authorized_keys
   sudo systemctl restart sshd.service
   ```

---

### **Step 3: Download and Install Cloudera Manager (CM)**

1. **Download Cloudera Manager Installer:**
   - Download the Cloudera Manager Installer for version 7.4.4 using `wget`:
   ```bash
   wget https://archive.cloudera.com/cm7/7.4.4/cloudera-manager-installer.bin
   ```

2. **Make the Installer Executable:**
   ```bash
   chmod +x cloudera-manager-installer.bin
   ```

3. **Run the Installer:**
   - To start the installation of Cloudera Manager, run the following command:
   ```bash
   sudo ./cloudera-manager-installer.bin
   ```
   - Follow the prompts for the installation. Refer to the official [Cloudera installation documentation](https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/installation/topics/cdp-quick-start-streams-run-cm-server-installer.html) for detailed guidance.

---

### **Step 4: Web CM Runtime Installation**

1. **Start Cloudera Manager Web UI:**
   - Once the installation completes, you can access the Cloudera Manager Web UI by navigating to:
   ```
   http://<instance-public-ip>:7180
   ```
   - Use default login credentials (`admin`/`admin`).

2. **Run Cloudera Manager Server:**
   - Follow the runtime installation guide for deploying the server:
   [Cloudera Manager Runtime Installation Guide](https://docs.cloudera.com/cdp-private-cloud-base/7.1.7/installation/topics/cdp-quick-start-deployment-streams-install-runtime.html).

---

### **Step 5: Final Setup**

1. **Install CDP Runtime:**
   - After completing the Cloudera Manager installation, follow the steps in the runtime installation guide to configure services and resources.

2. **Verify Installation:**
   - Ensure that all necessary services (like HDFS, YARN, etc.) are running correctly through the Cloudera Manager Web UI.

---

### Author : Rushikesh Shinde
### contact : +91 9623548002
