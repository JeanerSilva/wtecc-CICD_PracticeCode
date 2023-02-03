Prerequisites

If you did not compete the previous labs, you will need to run the following commands to catch up and prepare your environment for this lab. If you have completed the previous labs, you may skip this step, although repeating it will not harm anything because Kubernetes is declarative and idempotent. It will always put the system in the same state given the same commands.

Issue the following commands to install everything from the previous labs.

    1

    tkn hub install task git-clone

Note: If the above command returns a error due to Tekton Version mismatch, please run the below command to fix this.

    1

    kubectl apply -f https://raw.githubusercontent.com/tektoncd/catalog/main/task/git-clone/0.9/git-clone.yaml

    1
    2

    tkn hub install task flake8
    kubectl apply -f tasks.yaml

Check that you have all of the previous tasks installed:

    1

    tkn task ls

You should see:

    1
    2
    3
    4
    5
    6

    NAME               DESCRIPTION              AGE
    cleanup            This task will clean...  2 minutes ago
    git-clone          These Tasks are Git...   2 minutes ago
    flake8             This task will run ...   1 minute ago
    echo                                        46 seconds ago
    nose                                        46 seconds ago
    
    
    
    
    Step 1: Check for ClusterTasks

Your pipeline currently has a placeholder for a build step that uses the echo task. Now it is time to replace it with a real image builder.

You search Tekton Hub for the word “build” and you see there is a task called buildah that will build images so you decide to use the buildah task in your pipeline to build your code.

Instead of installing it yourself, you first check the ClusterTasks in your cluster to see if it already exists. Luckily, the OpenShift environment you are using already has buildah installed as a ClusterTask. A ClusterTask is installed cluster-wide by an administrator and anyone can use it in their pipelines without having to install it themselves.

Check that the buildah task is installed as a ClusterTask using the Tekton CLI.

    1

    tkn clustertask ls

You should see the buildah task in the list with all the other available ClusterTasks.

    1
    2
    3

    NAME                       DESCRIPTION              AGE
    buildah                    Buildah task builds...   32 weeks ago
    ...

If you see it, you are ready to proceed.



Step 2: Add a Workspace to the Pipeline Task

Now you will update the pipeline.yaml file to use the new buildah task.

Open pipeline.yaml in the editor. To open the editor, click the button below.

In reading the documentation for the buildah task, you will notice that it requires a workspace named source.
Your Task

Add the workspace to the build task after the name, but before the taskRef. The workspace that you have been using is named pipeline-workspace and the name the task requires is source.

    1
    2
    3

        - name: build
          ... add code here...
          taskRef:

Hint
Click here for a hint.
Use the `workspaces` keyword with a `name` of "source" and a `workspace` with the name of "pipeline-workspace".
Solution
Click here for the answer.

    1
    2
    3
    4
    5

        - name: build
          workspaces:
            - name: source
              workspace: pipeline-workspace
          taskRef:
          
          
          
          Step 3: Reference the buildah Task

Now, you need to reference the new buildah task that you want to use. In the previous steps, you simply changed the name of the reference to the task. But since the buildah task is a ClusterTask, you need to add the statement kind: ClusterTask under the name so that Tekton knows to look for a ClusterTask and not a regular Task.
Your Task

Change the taskRef from echo to reference the buildah task and add a line below it with kind: ClusterTask to indicate that this is a ClusterTask:
Solution
Click here for the answer.

    1
    2
    3

          taskRef:
            name: buildah
            kind: ClusterTask
            
            
            Step 4: Update the Task Parameters

The documentation for the buildah task details several parameters but only one of them is required. You need to use the IMAGE parameter to hold the name of the image you want to build.

Since you might want to reuse this pipeline to build different images, you will make it a variable parameter that can be passed in when the pipeline runs. To do this, you need to change it here and add a parameter to the Pipeline itself.
Your Task

Change the message parameter to IMAGE and specify the value of $(params.build-image):
Solution 1
Click here for the answer.

Now that you are passing in the IMAGE parameter to this task, you need to go back to the top of the pipeline.yaml file and add the parameter there so that it can be passed into the pipeline when it is run.

Add a parameter named build-image to the existing list of parameters at the top of the pipeline under spec.params.
Solution 2
Click here for the answer.

    1
    2
    3

    spec:
      params:
        - name: build-image
        
        
        
        Step 4: Update the Task Parameters

The documentation for the buildah task details several parameters but only one of them is required. You need to use the IMAGE parameter to hold the name of the image you want to build.

Since you might want to reuse this pipeline to build different images, you will make it a variable parameter that can be passed in when the pipeline runs. To do this, you need to change it here and add a parameter to the Pipeline itself.
Your Task

Change the message parameter to IMAGE and specify the value of $(params.build-image):
Solution 1
Click here for the answer.

Now that you are passing in the IMAGE parameter to this task, you need to go back to the top of the pipeline.yaml file and add the parameter there so that it can be passed into the pipeline when it is run.

Add a parameter named build-image to the existing list of parameters at the top of the pipeline under spec.params.
Solution 2
Click here for the answer.

    1
    2
    3

    spec:
      params:
        - name: build-image
        
        
        
        
     Step 6: Apply Changes and Run the Pipeline
Apply the Pipeline

Apply the same changes you just made to pipeline.yaml to your cluster:

    1

    kubectl apply -f pipeline.yaml

Apply the PVC

Next, make sure that the persistent volume claim for the workspace exists by applying it using kubectl:

    1

    kubectl apply -f pvc.yaml

Start the Pipeline

When you start the pipeline, you need to pass in the build-image parameter, which is the name of the image to build.

This will be different for every learner that uses this lab. Here is the format:

image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest

Notice the variable $SN_ICR_NAMESPACE in the image name. This is automatically set to point to your container namespace.

Now, start the pipeline to see your new build task run. Use the Tekton CLI pipeline start command to run the pipeline, passing in the parameters repo-url, branch, and build-image using the -p option. Specify the workspace pipeline-workspace and volume claim pipelinerun-pvc using the -w option:

    1
    2
    3
    4
    5
    6

    tkn pipeline start cd-pipeline \
        -p repo-url="https://github.com/ibm-developer-skills-network/wtecc-CICD_PracticeCode.git" \
        -p branch=main \
        -p build-image=image-registry.openshift-image-registry.svc:5000/$SN_ICR_NAMESPACE/tekton-lab:latest \
        -w name=pipeline-workspace,claimName=pipelinerun-pvc \
        --showlog

You should see Waiting for logs to be available... while the pipeline runs. The logs will be shown on the screen. Wait until the pipeline run completes successfully.
Check the Run Status

You can see the pipeline run status by listing the pipeline runs with:

    1

    tkn pipelinerun ls

You should see:

    1
    2

    NAME                    STARTED         DURATION     STATUS
    cd-pipeline-run-fbxbx   1 minute ago    59 seconds   Succeeded

You can check the logs of the last run with:

    1

    tkn pipelinerun logs --last   
        
        
          

    
