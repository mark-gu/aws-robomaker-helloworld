# AWS RoboMaker Builder Session - Hello World


## Preparation

### Instructor Preparation

1. Create 10 environments and 10 IAM users, and share the environments with users.
2. Deploy RoboMaker-Common and CertGenerator
3. Setup WiFi connection for all robots.

 For each robot, print robot name, IP address on paper.
 1. laminate iam user name , robot name, IP, and leave the password field empty, so we can wipe and write on it before each session.

### Attendee Preparation

1. Clone the repository from: https://github.com/mark-gu/aws-robomaker-helloworld.git
2. Navigate to "...\aws-robomaker-helloworld\infrastructure\robomaker\\**cert_generator**", compress all files into a ZIP archive named **cert_generator.zip**
3. Physically connect to the Jetbot (via HDMI and USB cables), and connect it to the provided WiFi network. Attendees laptops must also join the same network.
4. Login to the AWS Management Console using assigned username and password.


## Instructions

### Create AWS Resources

1. Navigate to **CloudFormation** and deploy [01-robomaker-common.yml](infrastructure/robomaker/templates/01-robomaker-common.yml).
    - Set *Stack name* to *RoboMaker-Common*
    - Save the output values for later use

    **Note**: This step may have already been completed by your instructor.

2. Navigate to **S3**, and click into the bucket deployed in the previous step.
    - Create a new folder named *utils*
    - Upload *cert_generator.zip* into the *utils* folder

    **Note**: This step may have already been completed by your instructor.

3. Navigate to **CloudFormation** and deploy [02-robomaker-cert_generator.yml](infrastructure/robomaker/templates/02-robomaker-cert_generator.yml).
    - Set *Stack name* to *RoboMaker-CertGenerator*
    - Set other parameters appropriately

    **Note**: This step may have already been completed by your instructor.

4. Navigate to **CloudFormation** and deploy [03-robomaker-fleet.yml](infrastructure/robomaker/templates/03-robomaker-fleet.yml).
    - Set *Stack name* to *RoboMaker-Fleet-Jetbots*
    - Set *FleetName* to a value of your choice

5. Navigate to **CloudFormation** and deploy [04-robomaker-robot.yml](infrastructure/robomaker/templates/04-robomaker-robot.yml).
    - Set *Stack name* to *RoboMaker-Robot-Jetbot01*
    - Set *Name* to a value of your choice
    - Set *Architecture* to *ARM64*
    - Set others parameters appropriately
    - After deployment, download all the files from the *Outputs* tab to a local directory, e.g. `/path/to/robot_files/`.

### Configure the Robot

1. Power up the robot and wait for it to boot up.

2. Copy [resources/90-i2c.rules](resources/90-i2c.rules) to `/path/to/robot_files/` also.

3. Copy all the required files onto the robot.

    ```bash
    # Copy all the downloaded files including certs and config files
    scp /path/to/robot_files/* jetbot@<ip-address>:/home/jetbot/
    ```

4. Configure AWS IoT Greengrass on the robot.

    ```bash
    # SSH to the robot
    ssh jetbot@<ip-address>

    # Switch to the root user
    sudo su -s /bin/bash

    # Copy Amazon Root CA1 and the robot's own certificate to Greengrass
    cp *.pem /greengrass/certs/

    # Copy the robot's own certificate private key to Greengrass
    cp *.private.key /greengrass/certs/

    # Copy the configuration file to Greengrass
    cp config.json /greengrass/config/

    # Keep the SSH session live and continue to the next step...
    ```

5. Update robot devices

    ```bash
    # Change the owner of the file
    sudo chown root:root 90-i2c.rules

    # Move the file to udev
    sudo cp 90-i2c.rules /etc/udev/rules.d

    # Reload device permissions
    sudo udevadm control --reload && sudo udevadm trigger

    # Verify the updated permissions are in effect
    ls -la /dev/i2c-1

    # You should see permissions like the following:
    # Ensure UID = ggc_user and GID = i2c
    #
    #              UID      GID 
    #              ^^^      ^^^ 
    > crw-rw---- 1 ggc_user i2c 89, 1 Nov  8 16:16 /dev/i2c-1


    # Keep the SSH session live and continue to the next step...
    ```

6. Start Greengrass

    ```bash
    # Change the owner of the file
    sudo /greengrass/ggc/core/greengrassd start

    # Verify Greengrass is running
    ps aux | grep greengrass

    # Terminate the ssh connection
    exit # or Ctrl-d
    ```

### Develop the ROS Application

1. In the AWS Management Console, navigate to **RoboMaker**.

2. Click on **Development environments** on the left, then **Create environment** to create a new development environment.
   - Set *Name* to a value of your choice
   - Set *Pre-installed software suite* to *Melodic*
   - Set other fields appropriately

    **Note**: This step may have already been completed by your instructor.

3. Wait until your development environment is ready to use.


### Deploy the ROS Application


### Clean up

1. Remove certs, config
2. Delete all Robot and Fleet Stacks.
3. Delete files under S3 bucket/robots/ folder.
4. Deploy empty ROS app to bot???

