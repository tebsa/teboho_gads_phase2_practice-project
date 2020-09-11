#LAB: Google Cloud Fundamentals: Getting Started with Compute Engine

##Objectives:

In this lab, you will learn how to perform the following tasks:

    - Create a Compute Engine virtual machine using the Google Cloud Platform (GCP) Console.

    - Create a Compute Engine virtual machine using the gcloud command-line interface.

    - Connect between the two instances.

##Steps

1. Create a virtual machine using the GCP Console

    gcloud compute instances create "my-vm-1" --zone "us-central1-a" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20200910" --subnet "default" --tags "http"

    gcloud compute firewall-rules create default-allow-http --direction "INGRESS" --network "default" --action "ALLOW" --rules "tcp:80" --target-tags "http"


2. Create a virtual machine using the gcloud command line

    - Display a list of all the zones and select a zone different from the one on my-vm-1

        gcloud compute zones list | grep us-central1

    - Set the default zone to the one just chosen, 

        gcloud config set compute/zone us-central1-b
    
    - Create a VM instance called my-vm-2 in the set zone

        gcloud compute instances create "my-vm-2" --machine-type "n1-standard-1" --image-project "debian-cloud" --image "debian-9-stretch-v20190213" --subnet "default"

3. Connect between VM instances

    1. Use the ping command to confirm that my-vm-2 can reach my-vm-1 over the network:
        
        - Connect to my-vm-2

            gcloud compute ssh my-vm-2

        - ping my-vm-1 from my-vm-2

            ping my-vm-1
        
        - Use the ssh command to open a command prompt on my-vm-1:

            ssh my-vm-1
        
        - At the command prompt on my-vm-1, install the Nginx web server:

            sudo apt-get install nginx-light -y
        
        - Use the nano text editor to add a custom message to the home page of the web server:

            sudo nano /var/www/html/index.nginx-debian.html
        
        - Add text like this just below the h1 header, and replace YOUR_NAME with your name:

            Hi from YOUR_NAME

        - Exit the text editor and confirm that the web server is serving your new page. At the command prompt on my-vm-1, execute this command:

            curl http://localhost/

            - Result

                The response will be the HTML source of the web server's home page, including your line of custom text.
        
        - Exit from my-vm-1 by running the command below.

            exit
        
        - To confirm that my-vm-2 can reach the web server on my-vm-1, at the command prompt on my-vm-2, execute this command:

            curl http://my-vm-1/

            - Result

                The response will be the HTML source of the web server's home page, including your line of custom text.
        
        - Get the external IP of my-vm-1 
        
            gcloud compute instances list --zone uscentral-1b
        
        - Copy the External IP address for my-vm-1 and paste it into the address bar of a new browser tab. You will see your web server's home page, including your custom text.

            - Result

                The response will be the HTML source of the web server's home page, including your line of custom text.

