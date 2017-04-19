# RTMPBee
> The RTMPBee project utilizes an AMI that is equipped with a Red5-based java program that attempts to subscribe N-number of clients to a stream on a target server.

[IR5 RTMPBee fork](https://github.com/infrared5/rtmpbee)

## Bee
Distributions for the *RTMP Bee* are available in [rtmpbee-dist](rtmpbee-dist) directory and contain bees that can be run using with Java 7 or Java 8.

_Java 8 RTMP Bee is recommended._

* [Requirements](#requirements)
* [Operations](#operations)
  * [attackStream](#attackstream)
  * [attackStreamManager](#attackstreammanager)
* [RTMPBee JAR](#rtmpbee-jar)
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
The following dependencies are required on your system in order to perform a *beeswithmachineguns* attack with *RTMPBee*.

> Check to be sure that you do not already have these dependencies on your system before installing with the `brew` examples.

## Python 2.7

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

## AWS
Amazon Web Services is used to spin up instances from an AMI that will serve as a single bee with N-number of bullets.

## Credential Requirements
The following are required credentials in order to properly run bees with an *RTMPBee*:

* PEM file for AWS account using in attack
* KEY and SECRET for account to use with bees (paramiko)

These will be referred to as _PEM_FILE_, _AWS_KEY_ and _AWS_SECRET_ in any examples of this documentation to follow.

> Please ask your administrator for these credentials.

#### You may need to create a new IAM User.
Infrared5 has created the user `invaluablebee` with _AdministratorAccess_ policy: [https://console.aws.amazon.com/iam/home#users/invaluablebee](https://console.aws.amazon.com/iam/home#users/invaluablebee)

If is this User's *KEY* and *SECRET* values that will be used in an attack using the Infrared5 AWS account.

# Operations

The current setup and teardown of EC2 instances used by [beeswithmachineguns](https://github.com/newsapps/beeswithmachineguns) is utilized. However, since Red5 Pro/Infrared5 developed a Java client which aides in establishing N-number of subscription streams for 1 broadcast stream, the original `attack` operation from **bees** was not immediately applicable to achieve a "video sting". As such, Infrared5 has modified the **bees** code to issue any CLI command through `attack2`.

## attackStream

`attackStream` is similar to `attack` in that it issues a request on each bee specified in `up`. The only option it accepts is `--cmd` which is a command `string` to run on the attached shell of the AMI instance that is spun up. The AMI contains a JAR file - considered the *RTMPBee* - on its root and is available to be invoked as such:

```ssh
./bees attackStream --cmd "java -jar rtmpbee.jar 54.201.243.119 1935 live qa12345678 5 5"
```

> [RTMP Bee Documentation](https://github.com/infrared5/rtmpbee)

## attackStreamManager

`attackStreamManager` makes running an attack unsing an *RTMPBee* easier by just providing a full REST URL with `context` and `streamName` URI parameters (the webapp and stream name of to which a live broadcast is in session) as the endpoint. The provided URL will be used to run a `GET` request on the target Stream Manager to get the payload of a target (e.g., Edge) subscriber server details.

```ssh
./bees attackStreamManager --endpoint http://52.9.184.79:5080/streammanager/api/2.0/event/live/todd\?action\=subscribe\&accessToken\=xyz123 --port 1935 --streamcount 5 --timeout 5
```

### --endpoint
This URL is passed along to each Bee (RTMPBee) which handles making the RESTful request to access the endpoint for stream subscription.

[Stream Manager API Documentation](https://www.red5pro.com/docs/server/streammanagerapi/#rest-api-for-streams)

### --port
The port on the Edge server for the *RTMPBee* to make an RTMP connection to start consumption of the stream.

### --streamcount
The amount of Bees (subscribers) to issue

### --timeout
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
$ git checkout invaluable origin/client/invaluable
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
To start a broadcast session visit the Origin server and start an RTMP broadcast - mainly to get around CORS issues you may come across -, e.g., [http://54.193.21.251:5080/live/broadcast.jsp?host=54.193.21.251&view=rtmp](http://54.193.21.251:5080/live/broadcast.jsp?host=54.193.21.251&view=rtmp).

1. Enter a `Stream Name` (i.e., `test`).
2. `Accept the security considerations from the Flash Player`.
3. Click `Start Broadcast`.

> The `Stream Name` provided will be used in the attack commands. Additionally, the `context` options will be `live` if using the broadcast URL from above.

## Verify
To verify that you have established a broadcast session and have an available consumable endpoint for the *RTMPBee* subscribers, make the following similar `GET` request on the StreamManager:

```sh
$ curl -X GET http://52.9.184.79:5080/streammanager/api/2.0/event/live/todd/stats?accessToken=xyz123
```

_or copy and paste the url into a browser_

Thew result should be similar to the following JSON:

```js
{"currentSubscribers":0,"startTime":1492618401576,"name":"todd","scope":"/live","serverAddress":"54.193.21.251","region":"us-west-1"}
```

Note the `name` and `scope` attributes; they correspond to the `stream name` and app `context`, respectively.

## Attack

There are 3 commands that will be used in issuing an attack with an RTMPBee: `up`, `attackStreamManager`, and `down`.

### AMI
An AMI is used to spin up *bees* as dynamic servers. The AMI setup by Infrared5 is named `IR5 - RTMPBee`. You will need the proper associated *PEM_FILE* that was used in creating the AMI. The security group used was `default`. _Visit the EC2 instance named "IR5 - invaluabledev-bee" for more detail_.

> Before proceeding, make sure you have the *PEM_FILE* in your `~/.ssh` directory and have access to the proper *AWS_KEY* and *AWS_SECRET* credentials.

### up
The `up` command is prepended with the definition of global properties related to credentials. The following command will spin up *3* servers based on the AMI with id *ami-49874122* with the security group *default* in the *us-east-1a* AWS zone.

Additionally, the *ec2-user* user, which is associated with the *PEM_FILE*, is the user that is logged into an SSH session when the bees are ready to attack.

```ssh
$ AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees up -i ami-49874122 -k PEM_FILE -s 3 -g default -z us-east-1a -l ec2-user
```

> Release of the console after issue `up` notifies of change to state of the EC2 instances requested. However, sometimes this is a falsey notification of the instances being able to receive SSH coammnds for the RTMPBees. Please allow an additional minute or two after the completion of `up` before issuing `attackStreamManager`.

### attackStreamManager
The `attackStreamManager` command invokes the *RTMPBee* with options explained in more detail previously in this document. The following command will invoke the RTMPBee to issue *5* subscription streams to the endpoint returned from `http://52.9.184.79:5080/streammanager/api/2.0/event/live/todd?action=subscribe&accessToken=xyz123` and request each stream to shut down *10* seconds after connecting.

```sh
$ AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET /bees attackStreamManager --endpoint http://52.9.184.79:5080/streammanager/api/2.0/event/live/todd\?action\=subscribe\&accessToken\=xyz123 --port 1935 --streamcount 5 --timeout 10
```

### down
The `down` command spins down the spun up instances through `up`. This should be run after the `attackStreamManager` has run its course.`

```
AWS_ACCESS_KEY_ID=AWS_KEY AWS_SECRET_ACCESS_KEY=AWS_SECRET ./bees down
```

# Tracking

## Stream Manager API
Revist the stats for the broadcast using the Stream Manager API while the attack is happening, e.g.,:

```sh
$ curl -X GET http://52.9.184.79:5080/streammanager/api/2.0/event/live/todd/stats?accessToken=xyz123
```

You should see the `connectedSubscriber` attribute value tally be equal to the number of (`Bees` * `Bullets`) defined in the attack.

> The stats are updated at intervals around 10 seconds. If you do not see the expected results right away, refresh the page or make the `curl` request at a later time.

## TCPTrack

Using `tcptrack` on Origin and Edge servers. SSH into server and issue:

```ssh
$ sudo tcptrack -i eth0 port 1935
```
