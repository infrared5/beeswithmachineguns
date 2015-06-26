RTMPBee
===
> The RTMPBee project utilizes an AMI that is equipped with a Red5-based java program that attempts to subscribe N-number of clients to a stream on a target server.

[IR5 RTMPBee fork](https://github.com/infrared5/rtmpbee)

Credential Requirements
===
The following are required credentials in order to properly run **bees** with an RTMPBee:

* PEM file for AWS account using in attack
* KEY and SECRET for account to use with bees (paramiko)

These will be referred to as _PEM_FILE_, _AWS_KEY_ and _AWS_SECRET_ in any examples of this documentation to follow.

### Please ask your administrator for these credentials.

Operations
===
The current setup and teardown of EC2 instances used by [beeswithmachineguns](https://github.com/newsapps/beeswithmachineguns) is utilized. However, since Red5 Pro/Infrared5 developed a java client which aides in establishing N-number of subscription streams for 1 broadcast stream, the original `attack` operation from **bees** was not immediately applicable to achieve a "video sting". As such, Infrared5 has modified the **bees** code to issue any CLI command through `attack2`. 

## attack2

`attack2` is similar to `attack` in that it issues a request on each bee specified in `up`. The only option it accepts is `--cmd` which is a command `string` to run on the attached shell of the AMI instance that is spun up. The AMI contains a JAR file - considered the RTMPBee - on its root and is available to be invoked as such:
```
./bees attack2 --cmd "java -jar rtmpbee.jar 54.201.243.119 1935 live qa12345678 5 5"
```


## rtmpbee.jar
The following defines the CLI options for the rtmpbee JAR:

```
java -jar rtmpbee.jar [server-url] [server-port] [application-name] [stream-name] [n-streams] [timeout-seconds]
```

### server-url
The IP address of the server on which the stream resides.

### server-port
The RTMP port number on the server that is available.

### application-name
The application name that the broadcast stream resides in.

### stream-name
The stream name of the broadcast to consume.

### n-streams
The amount of stream clients to create.

### timeout-seconds
The amount of lapsed time (in seconds) after issuing a subscribe request to shut down the consumption session. The current API of RTMPClient in the Red5 code source does not support responding to subscription events of a client. As such, the determination of having successfully started playback is unreliable. At the moment, the Bee is requested to exit the subscription session after a period of time to allow for relinquish of control. The default is 10 seconds. This option allows you to define a desired number that best suits testing. 

Setup
===
### Note: virtualenvwrapper is optional, but recommended

When working on multiple python projects, it is generally a recommended practice to isolate your environment for each project so as not to overwrite dependencies globally. The accepted tool to do so in the Python community is [virtualenv](http://www.virtualenv.org).

You can install virtualenv following these directions: [http://www.virtualenv.org/en/latest/virtualenv.html#installation](http://www.virtualenv.org/en/latest/virtualenv.html#installation)

There is an additional set of cli tools, [virtualenvwrapper](http://virtualenvwrapper) that makes working with and managing multiple Python projects under virtualenv easy.

You can install and modify your local environment by following these directions: [http://virtualenvwrapper.readthedocs.org/en/latest/install.html](http://virtualenvwrapper.readthedocs.org/en/latest/install.html)

Once [virtualenvwrapper](http://virtualenvwrapper) is installed, you will have several more command line tools at your disposal in activating and deactivating the current isolated environment in which you are developing your Python project.

To create and workon on the IR5 **bees**:

* Clone this repo:

```
$ git clone git@github.com:infrared5/beeswithmachineguns.git bees_ir5
```

* Create and workon:

```
$ cd bees_ir5
$ mkvirtualenv bees
$ workon bees
```

* Pull in dependencies (_while still in /bees_ir5_):

```
$ pip install -r requirements.txt --system-site-packages
```

Structure
===
A pre-defined session has been established in order for testing - both usability testing and load testing. The id associated with this session is **qa12345678**. Additionally, this is the corresponding stream name that will be broadcast to by a publisher and consumes by subscribers. 

In the context of this testing, the bees sent on attack are subscribers. As such, a broadcast session should be established prior to running an attack. This will provide more reliable information in requesting and establishing subscriptions in a load.

## broadcast
To intiate a broadcast session, visit [http://streammanager-balancer-547145200.us-west-2.elb.amazonaws.com/broadcaster/broadcaster-index.html](http://streammanager-balancer-547145200.us-west-2.elb.amazonaws.com/broadcaster/broadcaster-index.html), accept the security considerations from the Flash Player, select the camera you wish to use and click **start broadcast**.

## subscribe
You can view your broadcast session to ensure that it is running properly at: [http://streammanager-balancer-547145200.us-west-2.elb.amazonaws.com/subscriber/viewer-index.html](http://streammanager-balancer-547145200.us-west-2.elb.amazonaws.com/subscriber/viewer-index.html). The subscriber client at this url performs additional service requests to obtain the edge server IP it should connect to. For the **bees**, we will use 3 pre-defined edge server IPs to issue requests on.

The 3 Edge servers that the broadcast is being distributed to are:

* 54.201.243.119
* 54.213.96.38
* 54.201.249.200

**Only use these 3 IPs when issung RTMPBee requests. The examples that follow in this document use 54.201.243.119 in their explanation.**

Running
===
There are 3 commands that will be used in issuing an attack with an RTMPBee: `up`, `attack2`, and `down`.

**Make sure you have the PEM_FILE in your _~/.ssh_ directory and have access to the proper AWS_KEY and AWS_SECRET credentials.**

## up
The `up` command is prepended with the definition of global properties related to credentials. The following command will spin up **3** servers based on the AMI with id **ami-0ba9e83b** with the security group **launch-wizard-3** in the **us-west-2a** AWS zone.

Additionally, the **ubuntu** user, which is associated with the PEM_FILE, is the user that is logged into an SSH session when the bees are ready to attack.

```
AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees up -i ami-0ba9e83b -k PEM_FILE -s 3 -g launch-wizard-3 -z us-west-2a -l ubuntu
```

**Release of the console after issue `up` notifies of change to state of the EC2 instances requested. However, sometimes this is a falsey notification of the instances being able to receive SSH coammnds for the RTMPBees. Please allow an additional minute or two after the completion of `up` before issuing `attack2`.

## attack2
The `attack2` command invokes the RTMPBee with options explained in more detail previously in this document. The following command will invoke the RTMPBee to issue **5** subscription streams to **rtmp://54.201.243.119:1935/live/qa12345678** and request each stream to shut down **10** seconds after connecting.

**The quotation marks (") are required around the command string provided to `--cmd` option**

```
AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees attack2 --cmd "java -jar rtmpbee.jar 54.201.243.119 1935 live qa12345678 5 10"
```

## down
The `down` command spins down the spun up instances through `up`.

```
AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees down
```

Tracking
===
Using tcptrack. SSH into edge server and issue:

```
$ sudo tcptrack -i eth0 port 1935
```

Stream Administration
===
The Red5 server endpoints that the RTMPBees are attacking have been equipped with an administrative panel that allows one to track the amount of client streams being requested on an application. To access the admin console, point your browser to the following location under the Red5 installation, replacing **SERVER_IP** with the IP used in the RTMPBee JAR command from `attack2`:

[http://SERVER_IP:5080/admin/Red5AdminAIR.swf](http://SERVER_IP:5080/admin/Red5AdminAIR.swf)

Once loaded, you will need to provide a **Server Address** and **Username**. Enter the _SERVER_IP_ in to the **Server Address field and _admin_ in the **Username** field. Leave the **Password** field blank.

In following with the examples from this document, the admin url would be accesible at the following url: [http://54.201.243.119:5080/admin/Red5AdminAIR.swf](http://54.201.243.119:5080/admin/Red5AdminAIR.swf)