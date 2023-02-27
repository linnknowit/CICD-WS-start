# Workshop CI/CD
### Preparation
* Create a GitHub account
* Create an Azure account (https://azure.microsoft.com/sv-se/free/students/)

## Step 1 - CI with GitHub Actions
### Preparation
1. Fork this repository. https://github.com/linnknowit/CICD-WS-start/

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
