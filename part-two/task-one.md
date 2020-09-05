# LAB: Google Cloud Fundamentals: Getting Started with Cloud Storage and Cloud SQL

## Objectives:

In this lab, you will learn how to perform the following tasks:
	- Create a Cloud Storage bucket and place an image into it.
	- Create a Cloud SQL instance and configure it.
	- Connect to the Cloud SQL instance from a web server.
	- Use the image in the Cloud Storage bucket on a web page.
	
## Steps:

### Task 1: Deploy a web server VM instance

   `gcloud beta compute instances create bloghost --zone=us-central1-a --machine-type=e2-medium --subnet=default --network-tier=PREMIUM --metadata=startup-script=apt-get\ update$'\n'apt-get\ install\ apache2\ php\ php-mysql\ -y$'\n'service\ apache2\ restart --maintenance-policy=MIGRATE --service-account=408143798427-compute@developer.gserviceaccount.com --scopes=https://www.googleapis.com/auth/devstorage.read_only,https://www.googleapis.com/auth/logging.write,https://www.googleapis.com/auth/monitoring.write,https://www.googleapis.com/auth/servicecontrol,https://www.googleapis.com/auth/service.management.readonly,https://www.googleapis.com/auth/trace.append --tags=http-server --image=debian-9-stretch-v20200902 --image-project=debian-cloud --boot-disk-size=10GB --boot-disk-type=pd-standard --boot-disk-device-name=bloghost --reservation-affinity=any`

   `gcloud compute --project=qwiklabs-gcp-02-1f38b235a7c0 firewall-rules create default-allow-http --direction=INGRESS --priority=1000 --network=default --action=ALLOW --rules=tcp:80 --source-ranges=0.0.0.0/0 --target-tags=http-server`

### Task 2: Create a Cloud Storage bucket using the gsutil command line
1. On the Google Cloud Platform menu, Activate Cloud Shell.

2. For convenience, enter your chosen location into an environment variable called LOCATION.
    `export LOCATION=US`

3. In Cloud Shell, the DEVSHELL_PROJECT_ID environment variable contains your project ID.
   `gsutil mb -l $LOCATION gs://$DEVSHELL_PROJECT_ID`
   
4. Retrieve a banner image from a publicly accessible Cloud Storage location:
   `gsutil cp gs://cloud-training/gcpfci/my-excellent-blog.png my-excellent-blog.png`
   
5. Copy the banner image to your newly created Cloud Storage bucket:
   `gsutil cp my-excellent-blog.png gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png`
   
6. Modify the Access Control List of the object you just created so that it is readable by everyone:
   `gsutil acl ch -u allUsers:R gs://$DEVSHELL_PROJECT_ID/my-excellent-blog.png`
   
### Task 3: Create the Cloud SQL instance
	`gcloud sql instances create blog-db --root-password=PASSWORD --zone=us-central1-a --region=us-central1 --database-version=MYSQL_5_7`
	
1. Create an SQL user account
   `gcloud sql users create blogdbuser --instance=blog-db --password=PASSWORD`
   
2. Configure access to Cloud SQL instance
   - On the Cloud SQL Instances page in the Google Cloud Console.
   - Click the Connections tab, and then click Add network.
   - choose Public IP for purposes of this lab.
   - For Name, type **web front end**
   - For Network, type the external IP address of your bloghost VM instance, followed by /32
   
   	    **result** ---- *35.192.208.2/32*
   
### Task 4: Configure an application in a Compute Engine instance to use Cloud SQL
1. SSH in the VM instance bloghost.
   `gcloud compute ssh bloghost --zone=us-central1-a`
   
2. In your ssh session on bloghost, change your working directory to the document root of the web server:
   `cd /var/www/html`
   
3. Use the nano text editor to edit a file called index.php:
   `sudo nano index.php`
   
4. Paste the content below into the file:

*<html>*
*<head><title>Welcome to my excellent blog</title></head>*
*<body>*
*<h1>Welcome to my excellent blog</h1>*
*<?php*
 *$dbserver = "CLOUDSQLIP";*
*$dbuser = "blogdbuser";*
*$dbpassword = "DBPASSWORD";*
*// In a production blog, we would not store the MySQL*
*// password in the document root. Instead, we would store it in a*
*// configuration file elsewhere on the web server VM instance.*

*$conn = new mysqli($dbserver, $dbuser, $dbpassword);*

*if (mysqli_connect_error()) {*
        *echo ("Database connection failed: " . mysqli_connect_error());*
*} else {*
        *echo ("Database connection succeeded.");*
*}*
*?>*
*</body></html>*

5. Press Ctrl+O, and then press Enter to save your edited file.

6. Press Ctrl+X to exit the nano text editor.

7. Restart the web server:
   `sudo service apache2 restart`
   
8. Open a new web browser tab and paste into the address bar your bloghost VM instance's external IP address followed by /index.php. The URL will look like this:
   `35.192.208.2/index.php`
   
9. When you load the page, you will see that its content includes an error message beginning with the words:
    **result** ---- *Database connection failed: ...*
   
10. Return to your ssh session on bloghost. Use the nano text editor to edit index.php again.
   `sudo nano index.php`
   
11. In the nano text editor, replace CLOUDSQLIP with the Cloud SQL instance Public IP address that you noted above. Leave the quotation marks around the value in place.
12. In the nano text editor, replace DBPASSWORD with the Cloud SQL database password that you defined above. Leave the quotation marks around the value in place.

13.Press Ctrl+O, and then press Enter to save your edited file.

14.Press Ctrl+X to exit the nano text editor.

15.Restart the web server:
   `sudo service apache2 restart`
   
16. Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, the following message appears:
	**result** ---- *Database connection succeeded.*
	
	
### Task 5: Configure an application in a Compute Engine instance to use a Cloud Storage object
1. In the GCP Console, click Storage > Browser.

2. Click on the bucket that is named after your GCP project.

3. In this bucket, there is an object called my-excellent-blog.png. Copy the URL behind the link icon that appears in that object's Public access column, or behind the words "Public link" if shown.

4. Return to your ssh session on your bloghost VM instance.

5. Enter this command to set your working directory to the document root of the web server:
	`cd /var/www/html`
	
6. Use the nano text editor to edit index.php:
	`sudo nano index.php`
	
7. Use the arrow keys to move the cursor to the line that contains the h1 element. Press Enter to open up a new, blank screen line, and then paste the URL you copied earlier into the line.

8. Paste this HTML markup immediately before the URL:
	*<img src='*
9. Place a closing single quotation mark and a closing angle bracket at the end of the URL:
	*'>*
	**result** ---- *<img src='https://storage.googleapis.com/qwiklabs-gcp-0005e186fa559a09/my-excellent-blog.png'>*
	
10. Press Ctrl+O, and then press Enter to save your edited file

11. Press Ctrl+X to exit the nano text editor.

12. Restart the web server:
	   `sudo service apache2 restart`
	   
13. Return to the web browser tab in which you opened your bloghost VM instance's external IP address. When you load the page, its content now includes a banner image.

End your labs
