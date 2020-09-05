# LAB: Google Cloud Fundamentals: Getting Started with App Engine

## Objectives:

In this lab, you will learn how to perform the following tasks:
	- Initialize App Engine.
	- Preview an App Engine application running locally in Cloud Shell.
	- Deploy an App Engine application, so that others can reach it.
	- Disable an App Engine application, when you no longer want it to be visible.
	
## Steps:

### Task 1: Initialize App Engine
1. In GCP, open the gcloud terminal

2. Initialize your App Engine app with your project and choose its region:
	`gcloud app create --project=$DEVSHELL_PROJECT_ID`
	
3. Clone the source code repository for a sample application in the hello_world directory:
	`git clone https://github.com/GoogleCloudPlatform/python-docs-samples`

4. Navigate to the source directory:
	`cd python-docs-samples/appengine/standard_python3/hello_world`
	
### Task 2: Run Hello World application locally
1. Execute the following command to download and update the packages list.
	`sudo apt-get update`

2. Set up a virtual environment in which you will run your application. Python virtual environments are used to isolate package installations from the system.
	`sudo apt-get install virtualenv`
	*If prompted [Y/n], press Y and then Enter.*
	`virtualenv -p python3 venv`

3. Activate the virtual environment.
	`source venv/bin/activate`
   
4. Navigate to your project directory and install dependencies.
	`pip install  -r requirements.txt`
	
5. Run the application:
	`python main.py`
   
6. In Cloud Shell, click Web preview > Preview on port 8080 to preview the application.
	
7. To end the test, return to Cloud Shell and press Ctrl+C to abort the deployed service.
	
8. Using the Cloud Console, verify that the app is not deployed. In the Cloud Console, on the Navigation menu, click App Engine > Dashboard.
	`gcloud app instances list`
	
### Task 3: Deploy and run Hello World on App Engine
1. Navigate to the source directory:
	`cd ~/python-docs-samples/appengine/standard_python3/hello_world`
	
2. Deploy your Hello World application.
	`gcloud app deploy`
	*If prompted "Do you want to continue (Y/n)?", press Y and then Enter.*
	This app deploy command uses the app.yaml file to identify project configuration.
	
3. Launch your browser to view the app at http://YOUR_PROJECT_ID.appspot.com
	`gcloud app browse`
	
4. Copy and paste the URL into a new browser window.
	`curl URL`
	**result** ---- *html for web page*

### Task 4: Disable the application
1. In the Cloud Console, on the Navigation menu, click App Engine > Settings.
2. Click Disable application.
3. Read the dialog message. Enter the App ID and click DISABLE.
	`gcloud app version stop v1` *Note: Assuming version v1 is the version to disable*
	
4. If you refresh the browser window you used to view to the application site, you'll get a 404 error.
	`curl URL`
	**result** ---- *404 page*

End your labs
