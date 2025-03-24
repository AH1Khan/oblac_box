
# OBLAC Box

The **OBLAC Drives Box** is a physical computer running a pre-installed Linux operating system, which hosts OBLAC Drives and associated services. It supports access via Wi-Fi or a local network.

![OBLAC Box](https://github.com/user-attachments/assets/7e323ecc-9f89-4aef-9e6f-f756bed84f84)

---

# Oblac Provisioning

**Oblac Provisioning** refers to the process of installing the operating system and writing the OBLAC software to the device. This can be done in a series of steps:

### Step 1: Hardware Connection

To begin, power the OBLAC drive, connect the bootable USB drive and link the device to the Synapticon network. Below is a picture showing the hardware setup:

![Hardware Setup](https://github.com/user-attachments/assets/92e82b01-b994-489a-9e61-5d044ecb447f)  
![Hardware Setup 2](https://github.com/user-attachments/assets/ec5ba39f-da8a-474c-a97c-67bdb0c72277)

### Step 2: Install Required Libraries

Run the following commands to install the necessary libraries:

```bash
sudo apt-get install jq
ansible-galaxy install geerlingguy.docker
ansible-galaxy install geerlingguy.pip
ansible-galaxy collection install community.general
```

### Step 3: Custom HostName (Optional)

If you want to change the hostname of the OBLAC device to a custom name, you can do so by editing the `main.yml` file located in the `oblac-drives-box-provisioning/roles/up2/` directory.

1. Navigate to the `main.yml` file in the specified directory:
   
   ```plaintext
   oblac-drives-box-provisioning/roles/up2/main.yml
   ```

2. Open the file and locate the line of code responsible for setting the hostname. Modify it to the desired custom hostname as shown below:

   ![Screenshot from 2024-12-19 13-37-35](https://github.com/user-attachments/assets/4328c840-e20d-4629-97ee-73ef7115c2b5)

3. Save the file and proceed with the provisioning process to apply the changes.


### Step 4: Final Step

1. On your laptop, execute the following Git command to clone the Ansible provisioning repository and check out the correct branch:

    ```bash
    git clone git@github.com:synapticon/oblac-drives-box-provisioning.git
    ```

2. Run the `prepare_local_docker_registry.sh` script. This script sets up a local Docker registry server, downloads the latest Docker images of OBLAC Drives, and pushes them to the registry server. This helps avoid high traffic and server errors when pulling images from Docker Hub during provisioning.
3. Now run the following command for running the setup:
```bash
   ./commision_box.sh -u <Any name or word>
```
# How to SSH into the OBLAC Box

After completing the OBLAC commissioning, you can access the drive via SSH. To do so, you’ll need the `id_rsa` file, which can be found at `tmp/192.168.2.100`.

### Steps to SSH into the OBLAC Box:

1. **Obtain the Identification Key**: Ensure you have the `id_rsa` key file.

2. **Navigate to the Key Directory**: Open your terminal and navigate to the directory containing the `id_rsa` file.

3. **Add the Key to the SSH Agent**: Run the following command to start the SSH agent and add your key:

    ```bash
    eval $(ssh-agent -s) && ssh-add id_rsa
    ```

4. **Connect to the OBLAC Box**: Use the following command to SSH into the box:

    ```bash
    ssh administrator@oblac-drives-xxxxx.local
    ```

    The password is: `administrator`.

5. **Verify the Connection**: Enter the following command to verify the connection:

    ```bash
    ethercat sl
    ```

---

# OBLAC CI Provisioning

When provisioning the OBLAC for continuous integration (CI), additional Docker containers need to be installed inside the OBLAC to enable Jenkins detection. After completing the regular OBLAC commissioning, follow the steps below to integrate the device with Jenkins.

### Step 1: Set Up a New Node on Jenkins

1. **Go to Manage Jenkins**:

    ![Manage Jenkins](https://github.com/user-attachments/assets/d36865dd-2cf7-4152-be81-56c169863247)

2. **Go to Manage Nodes and Clouds**:

    ![Manage Nodes](https://github.com/user-attachments/assets/c3be7745-1c45-49b5-8ab9-d96243513a6d)

3. **Click on 'New Node'**:

    ![New Node](https://github.com/user-attachments/assets/0f764cb8-5ac4-4cff-b7b0-30fad04f9b9c)

4. **Enter the Following Configuration**:

    - **Name**: `robot1-1` (or any name you prefer)
    - **Executors**: 1
    - **Remote Root Directory**: `/home/jenkins`
    - **Labels**: (whatever name you prefer)
    
    The rest of the configuration should resemble this:

    ![Node Configuration](https://github.com/user-attachments/assets/a82c8b79-c9ae-45d3-aa9e-cf6028ab4b0e)

### Step 2: Final Configuration

1. Go to the `up2.hosts` file in the OBLAC commissioning repo and change the IP address to the box's hostname, as shown below:

    ![Update Hosts File](https://github.com/user-attachments/assets/8cf3977f-ccf5-4cc2-8fe2-58614d132b2a)

2. Run the following command from the OBLAC repo to start the Ansible script:

    ```bash
    sudo ansible-playbook -i up2.hosts -e serial_number=ci_robot1-1 -e box_brand=oblac -e '{ box_ci: True }' --tags ci -e "node_name=robot1-1" -e "slave_secret=ca34765dd7a87829e33d2d5bb8b1a51092256136530c61be0327d9bf2e2a526e" -e "wlan_interface=<wifi_interface>" yocto_box.playbook.yml -e 'ansible_python_interpreter=/usr/bin/python2.7'
    ```

    Alternatively:

    ```bash
    ansible-playbook -i up2.hosts -e serial_number=ci_robot1-1 -e box_brand=oblac -e '{ box_ci: True }' --tags ci -e "node_name=robot1-1" -e "slave_secret=ca34765dd7a87829e33d2d5bb8b1a51092256136530c61be0327d9bf2e2a526e" -e "wlan_interface=<wifi_interface>" yocto_box.playbook.yml -e 'ansible_python_interpreter=/usr/bin/python2.7'
    ```

    - Replace `<wifi_interface>` with the correct Wi-Fi interface for your device.
    - In the `node_name`, enter the name of the node you created in Jenkins.
    - In the `slave_secret`, use the secret key for the node (obtain it by clicking on the node in the Jenkins master).

At the end of this process, you should see in Jenkins that your node is now connected to the device and is in an idle state:

![Jenkins Node](https://github.com/user-attachments/assets/fd5be840-2d9b-47a8-b572-4e7cd6f8daf0)

# Changing the IP Address in Oblac Box

To change the IP address on an Oblac box, follow these steps:

1. **SSH into the Oblac Box:**
   - Use an SSH client to log into the Oblac box.

2. **Access the Repository:**
   - Navigate to the repository using the following command:
     ```
     cd /etc/systemd/network/
     ```

3. **List Communication Interfaces:**
   - Run the following command to list all communication interfaces:
     ```
     ls
     ```

4. **View Interface Details:**
   - To view the details of a specific network interface, use:
     ```
     cat <50-enp2s0.network>
     ```
5. **Edit Network Configuration:**
   - Use the below command to edit file:  
```sudo vi 50-enp2s0.network```  
   - Press `i` to enter editing mode.  
   - After making the necessary changes to the IP address, press `:wq` to save and exit the editor.
   - It will look something like this for static IP. Also please enter your own static IP.  
![Screenshot from 2024-12-19 15-46-45](https://github.com/user-attachments/assets/b5e7d1c6-ecca-41dc-9e04-6223d2527367)  

7. **Verify Changes:**
   - Run the following command again to confirm the changes were saved:
     ```
     cat <50-enp2s0.network>
     ```

8. **Restart the Device:**
   - Power cycle the device (turn it off and then on) to apply the changes.

9. **Test the Connection:**
   - Once the device has restarted, you should be able to see the new IP address or successfully ping the device.

# What to Do If the Firmware Is Incorrect

If incorrect firmware is used, devices may become stuck in the **INIT** state. This can happen if the device is flashed with incompatible firmware, a different version, or incorrect encryption. When this occurs, the EtherCAT network may stop working.

### To resolve the issue:

1. Run the following command in your terminal:

    ```bash
    while : ; do ethercat state boot ; done ;
    ```

2. After running this command, power off the device or setup, then turn it back on.

This process should help the device exit the INIT state and resume normal operation.

**Note**: Repeatedly issuing the command is necessary to handle any timing issues during the state change.

