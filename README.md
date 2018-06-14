# minipos-on-openshift

This tutorial will show you how to launch your own MiniPOS system to OpenShift Online for your business to begin accepting Bitcoin Cash. At the end of this tutorial you should have a URL that points to your own private MiniPoS server.

# Prerequisites
* `oc` binary to talk to a OpenShift cluster
* `golang` version 1.9+
* Bitcoin Cash wallet with xPub key (I recommend Electron Cash)

# Step 1: Create an OpenShift Online account
Visit https://www.openshift.com/products/online/ and sign up for an account on the free tier. After the instance is done provisioning, you will be able to access a web console. If you are not familiar with OpenShift/Kubernetes fear not... we just need to grab one command from the web console. In the top right corner where you see your name, click the drop down menu and click `copy login command`.

Log into the cluster by pasting this into the terminal. It should look like:
```
$ oc login https://api.starter-us-east-1.openshift.com --token=<token>
```

Create a new project:
```
$ oc new-project <project_name>
```

# Step 2: Install `sbcli`
In order to deploy MiniPoS, I recommend using a tool called `sbcli`. `sbcli` is a CLI tool that allows you to deploy Service Bundles/Ansible Playbook Bundles. It is a golang project, so I recommend installing via:
```
$ go get -u github.com/automationbroker/sbcli
```

We now need to configure `sbcli` to point to my DockerHub instance. To do this type:
```
$ sbcli registry add --name dymurray --org dymurray
$ sbcli bundle list
..........<snip>..............
 BUNDLE               IMAGE                                          REGISTRY                                                                
 ---------------- -+- ------------------------------------------ -+- --------                                                                
 demo-apb          |  docker.io/dymurray/demo-apb:latest          |  dylan                                                                   
 cee-training-apb  |  docker.io/dymurray/cee-training-apb:latest  |  dylan                                                                   
 jupyterhub-apb    |  docker.io/dymurray/jupyterhub-apb:latest    |  dylan                                                                   
 foo-apb           |  docker.io/dymurray/foo-apb:latest           |  dylan                                                                   
 minipos-test-apb  |  docker.io/dymurray/minipos-test-apb:latest  |  dylan                                                                   
 jupyter-apb       |  docker.io/dymurray/jupyter-apb:latest       |  dylan                                                                   
 minipos-apb       |  docker.io/dymurray/minipos-apb:latest       |  dylan   
```

# Step 3: Provision MiniPoS to OpenShift
Before the next step, you need to grab your xPub key from your wallet. `sbcli` will prompt you for both the xPub key along with an email address to send invoices to.
```
$ sbcli bundle provision minipos-apb -p <project_name>
Plan: default
Enter value for parameter [xpub], default: [<nil>]: <enter_xpub_key_here>
Enter value for parameter [email_address], default: [<nil>]: foo@example.com
WARN Failed to create a InternalClientSet: unable to load in-cluster configuration, KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT must be defined.
INFO OpenShift version: v3.9.14                                                                         
WARN Failed to create a InternalClientSet: unable to load in-cluster configuration, KUBERNETES_SERVICE_HOST and KUBERNETES_SERVICE_PORT must be defined.
INFO unable to retrieve the network plugin, defaulting to not joining networks - clusternetworks.network.openshift.io "default" is forbidden: User "rhn-engineering-dymurray" cannot get clusternetworks.network.openshift.io at the cluster scope: User "rhn-engineering-dymurray" cannot get clusternetworks.network.openshift.io at the cluster scope                                            
INFO Creating RoleBinding bundle-3039032c-f145-4179-ac84-50ade707c58d
INFO Successfully created apb sandbox: [ bundle-3039032c-f145-4179-ac84-50ade707c58d ], with edit permissions in namespace bchboys
INFO Running post create sandbox functions if defined.
Successfully created pod [bundle-3039032c-f145-4179-ac84-50ade707c58d] to provision [minipos-apb] in namespace [minipos]
```

# Step 4: Access your MiniPoS instance
The above step will launch a pod in OpenShift that will deploy the MiniPoS instance to your namespace. The entire process will take 2-4 minutes depending how lucky you are. If you access the URL and see a `503` response just give the application a little more time to finish deploying. You can either access the application by clicking through the web console to your namespace and click on the created route, or you can grab the route from the CLI:
```
$ oc get route
NAME      HOST/PORT                                              PATH      SERVICES   PORT      TERMINATION   WILDCARD
minipos   minipos-bch.1d35.starter-us-east-1.openshiftapps.com             minipos    web                     None
```

Just enter the above URL (`minipos-bch.1d35.starter-us-east-1.openshiftapps.com` in my instance) into your browser and you should see a MiniPoS Server that can accept payments and store the invoices!

# Caveats
There are a few known issues with my deployment of MiniPoS. Right now the email feature will say `succeeded` but I've found that the logs show an unknown error. I need to reach out to the MiniPoS project to see if this is a known issue or not. Also, for some reason the `=` button is the button you click to view your list of paid transactions. If you have any feature requests that I might be able to add feel free to open them in MiniPoS or my forked version which I will link below.

# References
* Live demo (not refunding so any payments I keep :D): http://minipos-bch.1d35.starter-us-east-1.openshiftapps.com/
* https://github.com/dymurray/minipos-apb
* https://github.com/dymurray/minipos (fork of simon-v project. HUGE thanks to him!)
* https://hub.docker.com/r/dymurray/minipos-apb/
* https://hub.docker.com/r/dymurray/minipos/
* https://github.com/automationbroker/sbcli
* https://github.com/openshift/origin/releases
