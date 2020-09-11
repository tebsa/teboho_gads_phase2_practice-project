#LAB: Virtual Machines - Creating Virtual Machines (Essential Google Cloud Infrastructure: Foundation)

##Objectives:

In this lab, you learn how to perform the following tasks:

    - Customize an application server

    - Install and configure necessary software

    - Configure network access

    - Schedule regular backups

##Steps

1. Create the VM

    gcloud compute instances create mc-server --zone=us-central1-a --machine-type=n1-standard-1 --tags=minecraft-server --image=debian-9-stretch-v20200910 --image-project=debian-cloud --create-disk=mode=rw,size=50,type=pd-ssd,name=minecraft-disk,device-name=minecraft-disk

    gcloud compute addresses create mc-server-ip --region us-central1 --ip-version IPV4


2.  Prepare the data disk

    - Connect to mc-server
        
        gcloud compute ssh mc-server

    - Create a directory that serves as the mount point for the data disk, run the following command:

        sudo mkdir -p /home/minecraft
    
    - Format the disk

        sudo mkfs.ext4 -F -E lazy_itable_init=0,\
        lazy_journal_init=0,discard \
        /dev/disk/by-id/google-minecraft-disk
    
    - Mount the disk

        sudo mount -o discard,defaults /dev/disk/by-id/google-minecraft-disk /home/minecraft

3. Install and run the application

    - Install the Java Runtime Environment (JRE) and the Minecraft server

        1. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        2. Update the Debian repositories on the VM

            sudo apt-get update
        
        3. Install the headless JRE

            sudo apt-get install -y default-jre-headless
        
        4. Navigate to the directory where the persistent disk is mounted

            cd /home/minecraft
        
        5. Install wget

            sudo apt-get install wget
        
        6. If prompted to continue, type Y
        
        7. Download the current Minecraft server JAR file (1.11.2 JAR)

            sudo wget https://launcher.mojang.com/v1/objects/d0d0fe2b1dc6ab4c65554cb734270872b72dadd6/server.jar
    
    - Initialize the Minecraft server
    
        1. Initialize the Minecraft server

            sudo java -Xmx1024M -Xms1024M -jar server.jar nogui
        
        2. See the files that were created in the first initialization of the Minecraft server

            sudo ls -l
        
        3. Edit the EULA

            sudo nano eula.txt
        
        4. Change the last line of the file from eula=false to eula=true

        5. Save and Exit

            Press Ctrl+O, ENTER to save the file and then press Ctrl+X to exit nano.

    - Create a virtual terminal screen to start the Minecraft server

        1. Install screen

            sudo apt-get install -y screen
        
        2. Start your Minecraft server in a screen virtual terminal

            sudo screen -S mcs java -Xmx1024M -Xms1024M -jar server.jar nogui
        
    - Detach from the screen and close your SSH session

        1. To detach the screen terminal, press Ctrl+A, Ctrl+D. The terminal continues to run in the background. To reattach the terminal, run the following command:

            sudo screen -r mcs
        
        2. If necessary, exit the screen terminal by pressing Ctrl+A, Ctrl+D.

        3.  exit the SSH terminal

            exit

4. Allow client traffic

    - Create a firewall rule

        1. Create firewall rule

            gcloud compute firewall-rules create minecraft-rule --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:25565 --source-ranges=0.0.0.0/0 --target-tags=minecraft-server
        
    - Verify server availability

        1. Locate and copy the External IP address for the mc-server VM

            gcloud compute instances list

        2. Use the following website to test your Minecraft server: https://mcsrvstat.us/

5. Schedule regular backups

    - Create a Cloud Storage bucket

        1. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        2. Create a globally unique bucket name, and store it in the environment variable YOUR_BUCKET_NAME

            export YOUR_BUCKET_NAME=my_big_bucket
        
        3. Verify it with echo:

            echo $YOUR_BUCKET_NAME
        
        4. Create the bucket using the gsutil tool:

            gsutil mb gs://$YOUR_BUCKET_NAME-minecraft-backup
    
    - Create a backup script

        1. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        2. Navigate to your home directory:

            cd /home/minecraft
        
        3. Create the script

            sudo nano /home/minecraft/backup.sh
        
        4. Copy and paste the following script into the file:

            #!/bin/bash
            screen -r mcs -X stuff '/save-all\n/save-off\n'
            /usr/bin/gsutil cp -R ${BASH_SOURCE%/*}/world gs://${YOUR_BUCKET_NAME}-minecraft-backup/$(date "+%Y%m%d-%H%M%S")-world
            screen -r mcs -X stuff '/save-on\n'
        
        5. Save and Exit: Press Ctrl+O, ENTER to save the file, and press Ctrl+X to exit nano

        6. Make the script executable

            sudo chmod 755 /home/minecraft/backup.sh
    
    - Test the backup script and schedule a cron job

        1. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        2. Run the backup script:

            . /home/minecraft/backup.sh
        
        3. After the script finishes, return to the Cloud Console

        4. gsutil ls -r gs://my_big_bucket/**

        5. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        6.  Open the cron table for editing:

            sudo crontab -e
        
        7.  When you are prompted to select an editor, type the number corresponding to nano, and press ENTER.

        8. At the bottom of the cron table, paste the following line:

            0 */4 * * * /home/minecraft/backup.sh
        
        9.  Press Ctrl+O, ENTER to save the cron table, and press Ctrl+X to exit nano.

6. Server maintenance

    - Connect via SSH to the server, stop it and shut down the VM

        1. Connect to mc-server
        
            gcloud compute ssh mc-server
        
        2. Run the following command

            sudo screen -r -X stuff '/stop\n'

        3. Return to the Cloud Console
        
        4. Stop the mc-server instance.

            gcloud compute instances stop mc-server

    - Automate server maintenance with startup and shutdown scripts

        gcloud compute instances add-metadata mc-server --metadata=startup-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/startup.sh,shutdown-script-url=https://storage.googleapis.com/cloud-training/archinfra/mcserver/shutdown.sh