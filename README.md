# Workshop CI/CD

### Preparation
* Create a GitHub account
* Create an Azure account (https://azure.microsoft.com/sv-se/free/students/)

## Step 1 - CI with GitHub Actions
### Preparation
1. Fork this repository into a public repository. https://github.com/linnknowit/CICD-WS-start/ 

2. Create a new YAML-file in .github/workflows using GitHub. Call the file cicd-pipeline.yaml
<img width="751" alt="image" src="https://user-images.githubusercontent.com/125378671/221523160-af825a69-5c09-40e3-815e-528fa649b165.png">
<img width="848" alt="image" src="https://user-images.githubusercontent.com/125378671/221523456-9c7ec770-15c2-4d7b-8fec-bb09ae6d6332.png">

### Set up the workflow
*The workflow is created in the YAML-file. Since YAML depends on indentation to run correctly, make sure that you use 2 spaces or 1 tab when defining hierarchies within the file.*

3. Within the YAML file you just created, add a name for the workflow. This will be shown on the Actions-tab when you run the workflow. Let's call ours "Build and Deploy to Azure.".

```yml
name: Build and Deploy to Azure
```

4. Below the name, let's add a push trigger using the keyword *on*. This will trigger the workflow whenever a push is made within the repository. You can specify specific branch/es using the *branches* keyword, but we'll skip that for now.

```yml
name: Build and Deploy to Azure

on: 
  push:
```
5. Let's also make it possible to run our workflow from the Actions tab, without pushing any changes to the repository. We do this by adding the keyworkd *workflow_dispatch*.

```yml
name: Build and Deploy to Azure

on: 
  push:
  workflow_dispatch:
  
```

### Add jobs
*A workflow can have multiple jobs, which run in parallel if not specified otherwise. This is where we specify which actions (building blocks) we want to use in our CI/CD pipeline. You can add code quality tests, run the tests in your solution or set up whatever you think is nessecary for your pipeline. Today, we'll focus on building the solution for deployment.*

6. We add jobs using the keyword *jobs*

```yml
name: Build and Deploy to Azure

on: 
  push:
  workflow_dispatch:
  
jobs:
```

7. Jobs and all of it's actions run on a virtual machine (VM), which emulates a physical computer. GitHub has it's own predefined VMs, which we'll use. Specify that our VM should run on the latest windows version using these keywords:
```yml
jobs:
  build:
    runs-on: windows-latest
```

#### Add actions
*Actions are defined under the keyword *steps*. We're focusing on building the solution for future deployment, so our actions will be focusing on this.*

*The code we're using is written in .NET, so some of the actions will be .NET specific, but the theory behind this is the same for whatever programming language or framework you're using in your solution.*

8. Let's add the keyword *steps* to have somewhere to define our actions, and add an action that checks out our code using the keyword *uses*.
```yml
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3
```

9. Now the code is checked out on our VM, but we'll also have to set up .NET on the VM - which is the platform our code is written in. Let's add an action for this, and for this action, let's add a name and specify which .NET version we want to set up.

```yml
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'
```

10. The VM is set up with everything we need to build the code. To build, we'll not use a predefined action but instead run a command on the VM. Since our VM runs on Windows, we'll use Command shell commands. The commands are defined using the keyword *run*. 
```yml
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build --configuration Release
      
```

11. We'll also add a command which takes the built project and publishes it to a folder where we can find it for the next action.
```yml
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Publish with dotnet
        run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp
      
```

12. The last part of building our solution is to upload the artifact we just built and published for the deployment job. For this we'll use a predefined action called Upload Artifact. We'll specify what the uploaded artifact should be called using the keyword *name*, and the path where we published our artifact on the VM using the keyword *path*.
```yml
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Publish with dotnet
        run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: .net-app
          path: ${{env.DOTNET_ROOT}}/myapp
```

### Commit
*The entire YAML-file should now look like this:*
```yml
name: Build and Deploy to Azure

on: 
  push:
  workflow_dispatch:
  
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Publish with dotnet
        run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: .net-app
          path: ${{env.DOTNET_ROOT}}/myapp
```

13. Let's commit our file directly into the master branch.
<img width="621" alt="image" src="https://user-images.githubusercontent.com/125378671/221529128-7c88a0c9-aaf8-4c9b-b714-af51b599074c.png">

### Check out the running workflow
14. Click the Actions tab and see thet our workflow is running. The pipeline is running because we set a trigger for each push event within the repository. When we commited the YAML-file, we also pushed it to the repository. 

15. Click our workflow in the left pane menu, and click the ongoing build to see the logs for what's being done. Notice that the names we set for the actions are shown in the logs.
<img width="124" alt="image" src="https://user-images.githubusercontent.com/125378671/221539469-10febfec-3fa7-44e9-a4a7-956f767983b3.png">

<img width="458" alt="image" src="https://user-images.githubusercontent.com/125378671/221538228-c7dd8c59-f17d-4bff-97de-4eb995150091.png">


## Step 2 - CD with GitHub Actions and Azure
Now it's time to deploy our application to Azure. To do this, we first have to set up an App Service in Azure and then add some deployment steps to our YAML file.

### Set up an App Service in Azure
1. Go to https://portal.azure.com/. If you're asked to sign in - sign in using the account you created before.

2. Search for App Services and click Create.
<img width="401" alt="image" src="https://user-images.githubusercontent.com/125378671/221562360-b747f61d-a0a1-429e-93af-2514e35aef77.png">

3. Set up the Web App as follows:

**Project Details**
* Subscription: Your free subscription
* Resource Group: Create a new with a name you decide, a suggestion is cicd-ws

**Instance Details**
* Name: *This name has to be gobally unique, a suggestion is cicd-demo-ws-[your name or a number]*
* Publish: Code
* Runtime stack: .NET 6 (LTS)
* Operating System: Windows
* Region: West Europe or Sweden Central (you choose a region that is close to your users)

**Pricing plans**
* Windows Plan: Create a new or use the default one
* Pricing plan: Free F1 (Shared infrastructure) 

4. Click Next: Deployment >. Here you can set up your CD connection to GitHub, but we're going to do this in a later step so let's skip this for now.

5. Click Review + create, as we don't need to configure anything on the other tabs for this workshop. Check that your configuration looks like this, but with the names you decided:
<img width="340" alt="image" src="https://user-images.githubusercontent.com/125378671/221568337-dc0d78b5-f5d5-4e26-ac33-1574e225d73d.png">

6. Click Create, the ceration can take a few minutes. When it's done, click Go to resource. 

<img width="584" alt="image" src="https://user-images.githubusercontent.com/125378671/221569711-f1be2690-d10c-4e93-b52f-96d6313e74f6.png">

#### Set up connection between Azure and GitHub 
Now we've successfully created a web app in Azure! But, if you click the Browse button in the Azure portal, you'll see a default Azure page since we haven't connected our GitHub repository with our Azure Web App. Let's set that up!
<img width="854" alt="image" src="https://user-images.githubusercontent.com/125378671/221570595-ca1b176b-f94d-4792-b2f8-86c99758e80f.png">

<img width="648" alt="image" src="https://user-images.githubusercontent.com/125378671/221570881-2a46127a-aa90-4213-ac1b-757c93a4379d.png">

The deployment to Azure can be set up automatically via the portal, by using the Deployment Center in the left pane menu and authenticating with your GitHub Account. We'll use part of this to create a publish profile, but we'll write the deployment workflow jobs on our own.

<img width="118" alt="image" src="https://user-images.githubusercontent.com/125378671/221571610-62c813e2-8e02-44a0-8668-911d05618879.png">

7. Select the Deployment Center in the left pane menu, and authorize using GitHub. Set it up with the repository you've used, select **Use available workflow** for the Workflow option. Click Save.
<img width="474" alt="image" src="https://user-images.githubusercontent.com/125378671/221578381-9b8ade8c-de4d-4bf2-8124-9deb6bcbd782.png">

8. Click Manage publish profile and download the publish profile.
<img width="361" alt="image" src="https://user-images.githubusercontent.com/125378671/221587547-71d0e78f-6642-4cd0-8bd0-eae830f712a0.png">

9. Open the file on your computer using a text editor, and copy the content.

10. Go back to GitHub, select the Settings tab and Actions under Secrets and variables in the left pane menu.
<img width="733" alt="image" src="https://user-images.githubusercontent.com/125378671/221588832-21599527-76fe-4318-ae4e-dd668fbac422.png">

11. Click New repository secret.
<img width="411" alt="image" src="https://user-images.githubusercontent.com/125378671/221589923-7092fca5-24cb-41d0-a7b7-fe1286b5dc62.png">

12. Add your copied publish profile value. Set the Name to AZURE_WEBAPP_PUBLISH_PROFILE. Click Add secret.
<img width="424" alt="image" src="https://user-images.githubusercontent.com/125378671/221590895-0c13ef37-a887-4872-9ec6-8b3469623195.png">


##### Add deployment jobs in GitHub
Let's edit the YAML workflow file. Go to it by clicking Code, your workflows folder, and then the file name.

13. Edit the YAML file by clicking the pen icon.
<img width="571" alt="image" src="https://user-images.githubusercontent.com/125378671/221572121-1c63bb6b-baae-4bdb-8692-fe5356593ffe.png">

Notice that the Marketplace showed up to the right on the screen. This is where we've gotten our actions from when we wrote 
```yml
- uses: actions/xxx 
```
in the CI job. If you click an action you'll see how to use them in your YAML file. As you can see, there're options for environments other than .NET as well.

<img width="309" alt="image" src="https://user-images.githubusercontent.com/125378671/221574715-943d241f-0f74-4a2c-aa11-6a352705f0be.png">

14. Below the build job, add a deployment job:

```yml
  deploy:
    runs-on: windows-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
```

* The *runs-on* keyword defines that we're using a VM with the latest version of Windows for this job as well.

* Jobs run in parallel if not defined otherwise, since our deployment job depends on that the build job is completed, we add the *needs* keyword to make GitHub Actions wait with this job until the build job is completed.

* The *environment* keyword defines the environment variables. The name of the environment we're deploying to is Production. We'll get the url from an action we'll define in step 10.

15. Let's add the actions. First, add an action that downloads the artifact from the build job we created before. The *with* keyword let's us define the name of the artifact.

```yml
    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: .net-app
```

16. Add a job that deploys to Azure. For this, we'll use the keyword *id* which has to be the same as the second part of the url value in the environments variables we created in step 8. Also notice that the *uses* keyword gets this action from Azure instead of Actions, since this action is created by Azure. 

Replace the app-name value with the name you created in Azure. The publish-profile will get the publish profile secret we created earlier, it contains the credentals we need to deploy to our Azure Web App.

```yml
  steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: .net-app
        
      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: {The name of your web app in Azure, probably something like cicd-demo-ws}
          slot-name: Production
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: .
```

17. Commit to your master branch. The file should look like this now:

```yml
name: Build and Deploy to Azure

on: 
  push:
  workflow_dispatch:
  
jobs:
  build:
    runs-on: windows-latest
  
    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core SDK
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: '6.0.x'

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Publish with dotnet
        run: dotnet publish -c Release -o ${{env.DOTNET_ROOT}}/myapp

      - name: Upload artifact for deployment job
        uses: actions/upload-artifact@v3
        with:
          name: .net-app
          path: ${{env.DOTNET_ROOT}}/myapp

  deploy:
    runs-on: windows-latest
    needs: build
    environment:
      name: 'Production'
      url: ${{ steps.deploy-to-webapp.outputs.webapp-url }}
    
    steps:
      - name: Download artifact from build job
        uses: actions/download-artifact@v3
        with:
          name: .net-app
      
      - name: Deploy to Azure Web App
        id: deploy-to-webapp
        uses: azure/webapps-deploy@v2
        with:
          app-name: cicd-demo-ws
          slot-name: Production
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
          package: .
        
```

## Step 3 - Test out the CI/CD workflow
We've now set up a CI/CD workflow using GitHub Actions and Azure. Let's test it out!

1. Click the Code tab in GitHub. Click the Pages folder.
<img width="463" alt="image" src="https://user-images.githubusercontent.com/125378671/221599651-c6569c0d-69bf-470c-8853-03da3e11e12c.png">

2. Edit the Index.cshtml file. Change the text in the h1 and/or p tags to whatever you'd like.
<img width="635" alt="image" src="https://user-images.githubusercontent.com/125378671/221599947-30465db0-24c4-46c3-a5e4-e24c993386ab.png">

3. Commit the file. This will trigger the push trigger in in our workflow and the changes will automatically get built and deployed to our web app in Azure.

4. When the build and release jobs are done, refresh your web app in your browser and see that your changes have been deployed.

<img width="713" alt="image" src="https://user-images.githubusercontent.com/125378671/221600471-be4b41f8-d6da-49df-b052-4f3c47c589c8.png">


## Step 4 - Additional CI actions
As mentioned before, there're more ways to use workflows other than just building and deploying your code. 
