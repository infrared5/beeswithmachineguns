# Red5Pro Bees
> The suite of Red5 Pro Bees utilizes an AMI that is equipped with Red5-based java and cli programs that attempt to subscribe N-number of clients to a stream on a target server.

* [RTMPBee](https://github.com/red5pro/rtmpbee)
* [RTSPBee](https://github.com/red5pro/rtspbee)
* [RTCBee](https://github.com/red5pro/rtcbee)

## Bees

Distributions for the *RTMP Bee* are available in [rtmpbee-dist](rtmpbee-dist) directory and contain bees that can be run using with Java 7 or Java 8.

Distributions for the *RTSP Bee* are available in [rtspbee-dist](rtspbee-dist) directory and contains a Java 8 build only.

> Java 8 RTMP/RTSP Bee is recommended.

* [Requirements](#requirements)
  * [Python](#python)
  * [Java](#java)
  * [Chromium Browser](#chromium-browser)
  * [AWS](#aws)
* [Operations](#operations)
  * [attackStream](#attackstream)
  * [attackStreamManager](#attackstreammanager)
* [RTMPBee JAR](#rtmpbee-jar)
* [RTSPBee JAR](#rtspbee-jar)
* [RTCBee Bash](#rtcbee-bash)
* [System SetUp](#system-setup)
* [Running an Attack](#running-an-attack)
  * [Start a Broadcast](#start-a-broadcast)
  * [Verify](#verify)
  * [Attack](#attack)
    * [up](#up)
    * [attackStreamManager](#attackStreamManager)
    * [down](#down)
* [Tracking](#tracking)

# Requirements

The following dependencies are required on your system in order to perform a *beeswithmachineguns* attack with *RTMPBee*, *RTSPBee* and/or *RTCBee*.

> Check to be sure that you do not already have these dependencies on your system before installing with the `brew` examples.

## Python

Python 2.7 is required.

```sh
$ brew update
$ brew install python
```

Python is used to run the *beeswithmachineguns* program.

## Java

```sh
$ brew update
$ brew cask install java
```

You can use either:

* Java 1.7
* Java 1.8

_Java 8 is preferred._

## Chromium Browser

The Chromium Browser is used by the *RTCBee* to run an attack using WebRTC headlessly.

> You may need to modify the rtcbee scripts locally to point to your Chromium Browser install if not using Linux and installing `chromium-browser` through a package manager.

## AWS

Amazon Web Services is used to spin up instances from an AMI that will serve as a single bee with N-number of bullets.

### Infrared5 Bee AMIs

Currently, Infrared5 has create 2 Bee AMIs with differing virutalizations and in difference zones:

* *US East (N. Virginia)*: `red5pro-load-bee-paravirtual`
* *US West (N. California)*: `red5pro-load-bee-hvm`

### Create a Bee AMI

An AMI that contains the desired *RTMPBee* JAR is required in order to launch an attack.

> To create an RTSPBee AMI, perform similar instructions but with the rtspbee JAR.

To create the AMI with `paravirtual` virtualization:

1. Sign into your AWS account.
2. Navigate to the desired region (i.e., *US East (N. Virginia)*).
3. Select *Services > EC2*.
4. Select *Instances*.
5. Click *Launch Instance*.
6. Select an AMI that has *Virtualization* of `paravirtual`.
7. Select `t1.micro` as *Instance Type*.
8. Keep default *Instance Details* (or customize as seen fit).
9. Keey default *Instance Storage*.
10. Assign a `Name` tag to be able to easily find it for modification (i.e., `red5pro-load-bee`).
11. Assign a *Security Group* that has at least `22` and `1935` ports open.
12. Review the Instance Details and click *Launch*.
13. Create or Assign a security pair as desired (i.e., `red5proqa`). _The PEM will be used in the attack request, so hold on to it._
14. Select the agreement and *Launch*.
15. Once the new instance is available in *EC2 > Instances*, `ssh` into the box using the public IP and the PEM file from Step #13.

For example:
```sh
$ ssh -i ~/.ssh/red5proqa.pem ubuntu@xxx.xxx.xxx.xxx
```

> Be sure to assign your IP to the `22` port of the selected *Security Group*.

_The creation of one for HVM is very similar, with the only change being in Steps #6 and #7._

#### Install Java

> If you selected the Amazon AMI, Java is already installed. You can skip this step, but first make note of the Java version installed as you will need to upload the proper RTMPBee flavor.

While signed into the Instance, install Java:

```sh
$ sudo add-apt-repository ppa:webupd8team/java
$ sudo apt-get update
$ sudo apt-get install oracle-java8-installer
```

_The above installs Java 8 to use the [rtmpbee-dist/rtmpbee-java8.jar](rtmpbee-dist/rtmpbee-java8.jar)._

> Reference: [https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04](https://www.digitalocean.com/community/tutorials/how-to-install-java-with-apt-get-on-ubuntu-16-04)

#### Install Chromium Browser

```sh
$ sudo apt-get install -y chromium-browser
```

For client to open large number of Chromium instances need to increase limits:

Under `/etc/systemd/` update files with options (reboot required for it to be applied):

```sh
logind.conf: UserTasksMax=infinity
system.conf: DefaultTasksMax=infinity
```

#### Upload the RTMPBee JAR

SFTP into the instance, for example:

```sh
$ sftp -i ~/.ssh/red5proqa.pem ubuntu@xxx.xxx.xxx.xxx
```
> You will upload the Java 8 *or* Java 7 version of the **rtmpbee** depending on the Java version installed on the instance.

In the prompt, to upload the Java 8 bee:

```sh
> put rtmpbee-dist/rtmpbee-java8.jar rtmpbee.jar
```

In the prompt, to upload the Java 7 bee:

```sh
> put rtmpbee-dist/rtmpbee-java7.jar rtmpbee.jar
```

_SSH back into the instance to ensure that the JAR was uploaded properly._

#### Upload the RTSPBee JAR

SFTP into the instance, for example:

```sh
$ sftp -i ~/.ssh/red5proqa.pem ubuntu@xxx.xxx.xxx.xxx
```
> You will upload the Java 8 version of the **rtspbee**.

In the prompt, to upload the Java 8 bee:

```sh
> put rtspbee-dist/rtspbee-java8.jar rtspbee.jar
```

#### Upload the RTCBee Files

* Grab the latest release link address from [https://github.com/red5pro/rtcbee/releases](https://github.com/red5pro/rtcbee/releases).
* `ssh` into the instance.
* Execute the following command (replacing the version number): `$ wget https://github.com/red5pro/rtcbee/releases/download/vX.X.X/rtcbee-bash-X.X.X.zip`.

#### Create the AMI from the Instance

Navigate back to your AWS account in the browser. Locate the Instance in *EC2 > Instances* and:

1. Right-Click, *Image > Create Image*.
2. Provide a `Image Name` and `Image Description`.
3. Click *Create Image*.
4. Navigate to *EC2 > AMIs*.
5. Locate the newly created AMI and make note of the `AMI ID`.
6. Once the AMI is available, Navigate back to *EC2 > Instances* and terminate the instance as we don't need it any more - we will be launching new instances based on the AMI.

#### Important Information

The specifics of the AMI in this example that will be used in the attack:

| AMI ID | Name | Security Group | Region | Subnet | PEM | User |
| --- | --- | --- | --- | --- | --- | --- |
| ami-b669fba0| red5pro-load-paravirtual | red5-pro-ports | us-east-1a | subnet-259d4f52 | red5proqa | ec2-user |

> These are the details that match the AMI set up in *US East (N. Virginia)*. There is an additional AMI set up in *US West (N. California)*.

### Credential Requirements

The following are required credentials in order to properly run bees with an *RTMPBee*:

* PEM file for AWS account usage in attack
* KEY and SECRET for account to use with bees (paramiko)

These will be referred to as *PEM_FILE*, *AWS_KEY* and *AWS_SECRET* in any examples of this documentation to follow.

> Please ask your administrator for these credentials.

#### You may need to create a new IAM User

Infrared5 has created the user `red5probee` with _AdministratorAccess_ policy, which can be located in the Account Credentials.

> It is this User's *AWS_KEY* and *AWS_SECRET* values that will be used in an attack using the Infrared5 AWS account.

# Operations

The current setup and teardown of EC2 instances used by [beeswithmachineguns](https://github.com/newsapps/beeswithmachineguns) is utilized. However, since Red5 Pro/Infrared5 developed a Java client which aides in establishing N-number of subscription streams for 1 broadcast stream, the original `attack` operation from *bees* was not immediately applicable to achieve a "video sting". As such, Infrared5 has modified the *bees* code to issue any CLI command through `attackStream`.

## attackStream

`attackStream` is similar to `attack` in that it issues a request on each bee specified in `up`. The only option it accepts is `--cmd` which is a command `string` to run on the attached shell of the AMI instance that is spun up. The AMI contains a JAR file - considered the *RTMPBee* - on its root and is available to be invoked as such:

### RTMPBee

```sh
./bees attackStream --cmd "java -jar rtmpbee.jar xxx.xxx.xxx.xxx 1935 live qa12345678 5 5"
```

> [RTMP Bee Documentation](https://github.com/red5pro/rtmpbee)

### RTSPBee

```sh
./bees attackStream --cmd "java -jar rtspbee.jar xxx.xxx.xxx.xxx 1935 live qa12345678 5 5"
```

> [RTSP Bee Documentation](https://github.com/red5pro/rtspbee)

### RTCBee

```sh
./bees attackStream --cmd "cd rtcbee-bash && ./rtcbee.sh \"https://your.red5pro-deploy.com/live/viewer.jsp?host=your.red5pro-deploy.com&stream=qa12345678\" 5 5"
```

> [RTSP Bee Documentation](https://github.com/red5pro/rtspbee)


## attackStreamManager

`attackStreamManager` makes running an attack using an *RTMPBee* easier by just providing a full REST URL with `context` and `streamName` URI parameters (the webapp and stream name of to which a live broadcast is in session) as the endpoint. The provided URL will be used to run a `GET` request on the target Stream Manager to get the payload of a target (e.g., Edge) subscriber server details.

```sh
./bees attackStreamManager --endpoint http://xxx.xxx.xxx.xxx:5080/streammanager/api/2.0/event/live/streamName\?action\=subscribe\&accessToken\=abc123 --port 1935 --streamcount 5 --streamtimeout 5

```

### --endpoint
This URL is passed along to each Bee (RTMPBee) which handles making the RESTful request to access the endpoint for stream subscription.

[Stream Manager API Documentation](https://www.red5pro.com/docs/server/streammanagerapi/#rest-api-for-streams)

### --port
The port on the Edge server for the *RTMPBee* to make an RTMP connection to start consumption of the stream.

### --streamcount
The amount of Bees (subscribers) to issue

### --streamtimeout
The length of time to keep the subscription stream open (in seconds).

# RTMPBee JAR

Found in the [/rtmpbee-dist](rtmpbee-dist) directory of this repository are *RTMPBee* JARs for the desired Java version. This JAR file is the one that resides on the AMI server that is used to spawn the bees. The python scripts describe below will invoke that remote JAR upon spawn and attack. This section defines the API of the RTMPBee JAVA program.

The RTMPBee can be invoked with 2 separate sets of options:

* Full URL
* Partial URIs

## Full URL

Using the full url of the endpoint stream to consume can be done using the CLI as follows:

```sh
$ java -jar rtmpbee.jar [server-endpoint] [n-streams] [timeout-seconds]
```

## Partial URIs

Using the partials uris of the endpoint stream (for more fine grained detail) can be done using the CLI as follows:
```sh
$ java -jar rtmpbee.jar [server-url] [server-port] [application-name] [stream-name] [n-streams] [timeout-seconds]
```

# RTSPBee JAR

Found in the [/rtspbee-dist](rtspbee-dist) directory of this repository are *RTSPBee* JARs for the desired Java version. This JAR file is the one that resides on the AMI server that is used to spawn the bees. The python scripts describe below will invoke that remote JAR upon spawn and attack. This section defines the API of the RTSPBee JAVA program.

The RTSPBee can be invoked with 2 separate sets of options:

* Partial URIs

## Partial URIs

Using the partials uris of the endpoint stream (for more fine grained detail) can be done using the CLI as follows:
```sh
$ java -jar -noverify rtspbee.jar [server-url] [server-port] [application-name] [stream-name] [n-streams] [timeout-seconds]
```

# RTCBee Bash

Releases to the *RTCBee Bash* scripts can be found at [https://github.com/red5pro/rtcbee/releases](https://github.com/red5pro/rtcbee/releases).

The RTCBee is a bash script that runs `chromium-browser` headlessly.

```sh
$ ./rtcbee.sh [server-endpoint] [n-streams] [timeout-seconds]
```

## CLI Options
The following describes the command-line options regarding the two API.

### server-endpoint
The full URL endpoint that points to the stream to consume.

### server-url
The host IP address of the server on which the stream resides.

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

# System Setup

> Note: virtualenvwrapper is optional, but recommended

When working on multiple python projects, it is generally a recommended practice to isolate your environment for each project so as not to overwrite dependencies globally. The accepted tool to do so in the Python community is [virtualenv](http://www.virtualenv.org).

You can install virtualenv following these directions: [http://www.virtualenv.org/en/latest/virtualenv.html#installation](http://www.virtualenv.org/en/latest/virtualenv.html#installation)

There is an additional set of cli tools, [virtualenvwrapper](http://virtualenvwrapper) that makes working with and managing multiple Python projects under virtualenv easy.

You can install and modify your local environment by following these directions: [http://virtualenvwrapper.readthedocs.org/en/latest/install.html](http://virtualenvwrapper.readthedocs.org/en/latest/install.html)

Once [virtualenvwrapper](http://virtualenvwrapper) is installed, you will have several more command line tools at your disposal in activating and deactivating the current isolated environment in which you are developing your Python project.

To create and workon on the IR5 *bees*:

* Clone this repo:

```sh
$ git clone git@github.com:infrared5/beeswithmachineguns.git bees_ir5
```

* Create and workon:

```sh
$ cd bees_ir5
$ git checkout client/infrared5 origin/client/infrared5
$ mkvirtualenv bees
$ workon bees
```

* Pull in dependencies (_while still in /bees_ir5_):

> You may need to install `pip` first: [https://pip.pypa.io/en/stable/installing/](https://pip.pypa.io/en/stable/installing/)

```sh
$ pip install -r requirements.txt
```

# Running an Attack

In the context of this testing, the bees sent on attack are subscribers. As such, a broadcast session should be established prior to running an attack. This will provide more reliable information in requesting and establishing subscriptions in a load.

## Start a Broadcast

To start a broadcast session visit the Origin server and start an RTMP broadcast - mainly to get around CORS issues you may come across -, e.g., [http://xxx.xxx.xxx.xxx:5080/live/broadcast.jsp?host=xxx.xxx.xxx.xxx&view=rtmp](http://xxx.xxx.xxx.xxx:5080/live/broadcast.jsp?host=xxx.xxx.xxx.xxx&view=rtmp).

1. Enter a `Stream Name` (i.e., `test`).
2. `Accept the security considerations from the Flash Player`.
3. Click `Start Broadcast`.

> The `Stream Name` provided will be used in the attack commands. Additionally, the `context` options will be `live` if using the broadcast URL from above.

## Verify

To verify that you have established a broadcast session and have an available consumable endpoint for the *RTMPBee* subscribers, make the following similar `GET` request on the StreamManager:

```sh
$ curl -X GET http://xxx.xxx.xxx.xxx:5080/streammanager/api/2.0/event/live/streamName/stats?accessToken=abc123
```

_or copy and paste the url into a browser_

The result should be similar to the following JSON:

```js
{"currentSubscribers":0,"startTime":1492618401576,"name":"streamName","scope":"/live","serverAddress":"xxx.xxx.xxx.xxx","region":"us-west-1"}
```

Note the `name` and `scope` attributes; they correspond to the `stream name` and app `context`, respectively.

## Attack

There are 3 commands that will be used in issuing an attack with an RTMPBee: `up`, `attackStreamManager`, and `down`.

### AMI

An AMI is used to spin up *bees* as dynamic servers. The AMI setup by Infrared5 is named `IR5 - RTMPBee`. You will need the proper associated *PEM_FILE* that was used in creating the AMI. The security group used was `red5-pro-ports`.

Before proceeding, make sure you have the *PEM_FILE* in your `~/.ssh` directory and have access to the proper *AWS_KEY* and *AWS_SECRET* credentials.

> If you set up your system to use `virtualenvwrapper`, first issue: `workon bees` before proceeding.

### up

The `up` command is prepended with the definition of global properties related to credentials. The following command will spin up *3* servers based on the AMI with id *ami-49874122* with the security group *default* in the *us-east-1a* AWS zone.

Additionally, the *ec2-user* user, which is associated with the *PEM_FILE*, is the user that is logged into an SSH session when the bees are ready to attack.

```sh
$ AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees up -i ami-b669fba0 -k red5proqa -s 1 -g red5-pro-ports -t t1.micro -z us-east-1a -l ec2-user -v subnet-259d4f52
```

> Release of the console after issue `up` notifies of change to state of the EC2 instances requested. However, sometimes this is a falsey notification of the instances being able to receive SSH coammnds for the RTMPBees. Please allow an additional minute or two after the completion of `up` before issuing `attackStreamManager`.

### attackStreamManager

The `attackStreamManager` command invokes the *RTMPBee* with options explained in more detail previously in this document. The following command will invoke the RTMPBee to issue *5* subscription streams to the endpoint returned from `http://xxx.xxx.xxx.xxx:5080/streammanager/api/2.0/event/live/streamName?action=subscribe&accessToken=abc123` and request each stream to shut down *10* seconds after connecting.

```sh
$ AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees attackStreamManager --endpoint "http://xxx.xxx.xxx.xxx:5080/streammanager/api/2.0/event/live/streamName\?action\=subscribe\&accessToken\=abc123" --port 1935 --streamcount 5 --streamtimeout 10
```

> Note the quotation marks (`"`) around the stream manager API endpoint.

### down

The `down` command spins down the spun up instances through `up`. This should be run after the `attackStreamManager` has run its course.`

```sh
$ AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees down
```

# Tracking

## Stream Manager API

Revist the stats for the broadcast using the Stream Manager API while the attack is happening, e.g.,:

```sh
$ curl -X GET http://xxx.xxx.xxx.xxx:5080/streammanager/api/2.0/event/live/streamName/stats?accessToken=abc123
```

You should see the `connectedSubscriber` attribute value tally be equal to the number of (`Bees` * `Bullets`) defined in the attack.

> The stats are updated at intervals around 10 seconds. If you do not see the expected results right away, refresh the page or make the `curl` request at a later time.

## NetData

NetData was set up on the launched Edge (where subscribers are attacking). [https://github.com/firehol/netdata](https://github.com/firehol/netdata)

You can view the console for NetData and track CPU, load, etc., by visiting the instance at port `19999`. e.g., [http://xxx.xxx.xxx.xxx:19999/](http://xxx.xxx.xxx.xxx:19999/).

## TCPTrack

Using `tcptrack` on Origin and Edge servers. SSH into server and issue:

```sh
$ sudo tcptrack -i eth0 port 1935
```

