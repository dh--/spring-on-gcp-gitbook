# Debugging

## Cloud Debugger

Cloud Debugger lets you inspect the state of an application, at any code location, without stopping or slowing down the running application.

Cloud Debugger is supported in all Google Cloud runtime environments \(except for Cloud Functions\) and can also be used when running applications in non-Google Cloud environments \(on-premises, other clouds\).

### Enable API

```bash
gcloud services enable clouddebugger.googleapis.com
```

### Snapshot

A Snapshot can introspect the context information on a given line of code as user go through the code flow.

![Example of a Snapshot](../../.gitbook/assets/image%20%286%29.png)

### Logpoint

A Logpoint can add additional log messages to a running application without modifying the code nor redeploying the code.

![Example of a Logpoint](../../.gitbook/assets/image%20%2814%29.png)

### Conditions

In both Snapshot and Logpoint, you can specify conditionals so you can capture specific information for a specific request \(e.g., match against a session ID, or request ID\).

## Java Agent

Cloud Debugger works by adding a Java agent to your JVM startup argument, and the agent can communicate with the Cloud Debugger service in the Cloud. Through the Cloud Console, you can then instruct your JVM instances to take a Snapshot of the application state at a specific line of code, or to add an additional log message on a specific line.

### Agent Files

There are 2 types of Cloud Debugger Java agents that authenticates with Cloud Debugger service differently:

| Type | When to use? | Latest Version | Versioned URL |
| :--- | :--- | :--- | :--- |
| Machine Credentials | Google Cloud runtime environments | [Download](https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz) | https://storage.googleapis.com/cloud-debugger/archive/java/${VERSION}/cdbg\_java\_agent\_gce.tar.gz |
| Service Account Key | Non-Google Cloud environments | [Download](https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_service_account.tar.gz) | https://storage.googleapis.com/cloud-debugger/archive/java/${VERSION}/cdbg\_java\_agent\_service\_account.tar.gz |

{% hint style="info" %}
You can find all the versions in the [cloud-debug-java](https://github.com/GoogleCloudPlatform/cloud-debug-java/releases) GitHub repository. For example, Cloud Debugger agent version `2.25` using Machine Credentials can be downloaded with URL: [https://storage.googleapis.com/cloud-debugger/archive/java/2.25/cdbg\_java\_agent\_gce.tar.gz](https://storage.googleapis.com/cloud-debugger/archive/java/2.25/cdbg_java_agent_gce.tar.gz)
{% endhint %}

### Agent Configurations

#### Agent Path

To use the agent, you'll need to configure the JVM command line using the standard  `-agentpath` , e.g.:

```bash
java -agentpath:/opt/cdbg/cdbg_java_agent.so \
  -jar ...
```

Rather than hard coding the startup command line, you can also configure it with the `JAVA_TOOL_OPTIONS` environmental variable:

```bash
JAVA_TOOL_OPTIONS="-agentpath:/opt/cdbg/cdbg_java_agent.so"
java -jar ...
```

#### System Properties

There are additional flags you can pass to the Java agent using Java's system properties.

<table>
  <thead>
    <tr>
      <th style="text-align:left">System Properties</th>
      <th style="text-align:left">Description</th>
      <th style="text-align:left">Required</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left">
        <p>com.google.cdbg</p>
        <p>.module</p>
      </td>
      <td style="text-align:left">The name of your application.</td>
      <td style="text-align:left">Not required for Cloud Run or App Engine.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>com.google.cdbg</p>
        <p>.version</p>
      </td>
      <td style="text-align:left">The version of your application.</td>
      <td style="text-align:left">Not required for Cloud Run or App Engine.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>com.google.cdbg</p>
        <p>.breakpoints</p>
        <p>.enable_canary</p>
      </td>
      <td style="text-align:left"><code>true</code> or <code>false</code>.Whether to turn on debugger for
        a subset of the running instances. See <a href="https://cloud.google.com/debugger/docs/setup/java#canary_snapshots_and_logpoints">Canary snapshots and logpoints documentation</a>.</td>
      <td
      style="text-align:left">Not required and defaults to <code>false</code>.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>com.google.cdbg</p>
        <p>.auth.serviceaccount.enable</p>
      </td>
      <td style="text-align:left"><code>true</code> or <code>false</code>. Whether to authenticate with a
        Service Account key file.</td>
      <td style="text-align:left">Required when running outside of Google Cloud.</td>
    </tr>
    <tr>
      <td style="text-align:left">
        <p>com.google.cdbg.auth</p>
        <p>.serviceaccount.jsonfile</p>
      </td>
      <td style="text-align:left">File path to the Service Account key file.</td>
      <td style="text-align:left">Required when running outside of Google Cloud.</td>
    </tr>
  </tbody>
</table>

For example, you can enable the snapshot using the system property:

```text
JAVA_TOOL_OPTIONS="-agentpath:/opt/cdbg/cdbg_java_agent.so \
  -Dcom.google.cdbg.breakpoints.enable_canary=true"
```

#### Logging

By default the Cloud Debugger agent writes its logs to `cdbg_java_agent.INFO` file in the default logging directory. You can overwrite the log file path:

```text
java -agentpath:/opt/cdbg/cdbg_java_agent.so=--log_dir=/tmp/cdbg.log \
  -jar ...
```

Alternatively you can make the Java Cloud Debugger log to `stderr`:

```text
java -agentpath:/opt/cdbg/cdbg_java_agent.so=--logtostderr=1 \
  -jar ...
```

{% hint style="info" %}
See [Setting Up Cloud Debugger for Java](https://cloud.google.com/debugger/docs/setup/java#overview) documentation for more information.
{% endhint %}

### Runtime Configuration

{% tabs %}
{% tab title="App Engine" %}
Follow [App Engine Hello World!](../../getting-started/helloworld/app-engine.md) instructions to deploy an application to App Engine.

Cloud Debugger agent is automatically added to your App Engine application.

In Cloud Debugger console, you can see the Default service in the drop down:

![](../../.gitbook/assets/image%20%287%29.png)
{% endtab %}

{% tab title="Cloud Run" %}
Add the Cloud Debugger Java agent to the container, and configure the agent in the startup command line.

#### Clone

```text
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Containerize with a Dockerfile

In the Dockerfile, download the Cloud Debugger and build it as part of the container image:

{% code title="Dockerfile" %}
```text
FROM openjdk:11

# Create a directory for the Debugger. Add and unzip the agent in the directory.
RUN mkdir /opt/cdbg && \
     wget -qO- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz | \
     tar xvz -C /opt/cdbg

COPY target/helloworld.jar /app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```
{% endcode %}

Then build and push the container:

```bash
mvn package

PROJECT_ID=$(gcloud config get-value project)
docker build -t gcr.io/${PROJECT_ID}/helloworld .
docker push gcr.io/${PROJECT_ID}/helloworld
```

#### Containerize with Jib

Download the Cloud Debugger Java agent into `src/main/jib` directory so that Jib can include the agent files as part of the container image:

```bash
# Make a directory to store the Java agent
mkdir -p src/main/jib/opt/cdbg

# Download and extract the Java agent to the directory
wget -qO- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz | \
  tar xvz -C src/main/jib/opt/cdbg
```

Create the container image with Jib:

```bash
PROJECT_ID=$(gcloud config get-value project)
mvn compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

#### Deploy

Deploy to Cloud Run with Debugger Enabled using the environmental variable:

```bash
gcloud run deploy helloworld \
  --region=us-central1 \
  --platform=managed \
  --allow-unauthenticated \
  --set-env-vars='JAVA_TOOL_OPTIONS="-agentpath:/opt/cdbg/cdbg_java_agent.so=--logtostderr=1"' \
  --image=gcr.io/${PROJECT_ID}/helloworld
```

In Cloud Debugger console, you can see the `helloworld` service in the drop down:

![](../../.gitbook/assets/image%20%2810%29.png)
{% endtab %}

{% tab title="Kubernetes Engine" %}
Add the Cloud Debugger Java agent to the container, and configure the agent in the startup command line.

#### Clone

```bash
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Containerize with a Dockerfile

In the Dockerfile, download the Cloud Debugger and build it as part of the container image:

{% code title="Dockerfile" %}
```text
FROM openjdk:11

# Create a directory for the Debugger. Add and unzip the agent in the directory.
RUN mkdir /opt/cdbg && \
    wget -qO- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz | \
    tar xvz -C /opt/cdbg

COPY target/helloworld.jar /app.jar

ENTRYPOINT ["java", "-jar", "/app.jar"]
```
{% endcode %}

Then build and push the container:

```bash
mvn package

PROJECT_ID=$(gcloud config get-value project)
docker build -t gcr.io/${PROJECT_ID}/helloworld .
docker push gcr.io/${PROJECT_ID}/helloworld
```

#### Containerize with Jib

Download the Cloud Debugger Java agent into `src/main/jib` directory so that Jib can include the agent files as part of the container image:

```bash
# Make a directory to store the Java agent
mkdir -p src/main/jib/opt/cdbg

# Download and extract the Java agent to the directory
wget -qO- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz | \
  tar xvz -C src/main/jib/opt/cdbg
```

Create the image with Jib:

```bash
PROJECT_ID=$(gcloud config get-value project)
mvn compile com.google.cloud.tools:jib-maven-plugin:2.4.0:build \
  -Dimage=gcr.io/${PROJECT_ID}/helloworld
```

#### Deploy

Deploy to Kubernetes Engine with Debugger Enabled using the environmental variable using a Deployment YAML:

```bash
# Make a directory to store Kubernetes YAMLs
mkdir k8s/
```

Create a  Deployment YAML file and configure the environmental variable:

{% code title="k8s/deployment.yaml" %}
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: helloworld
  name: helloworld
spec:
  replicas: 1
  selector:
    matchLabels:
      app: helloworld
  template:
    metadata:
      labels:
        app: helloworld
    spec:
      containers:
      - image: gcr.io/YOUR_PROJECT_ID/helloworld
        name: helloworld
        env:
        - name: JAVA_TOOL_OPTIONS
          value: "-agentpath:/opt/cdbg/cdbg_java_agent.so=--logtostderr=1 -Dcom.google.cdbg.module=helloworld-gke -Dcom.google.cdbg.version=1.0"
```
{% endcode %}

Deploy the YAML file:

```bash
kubectl apply -f k8s/deployment.yaml
```

In Cloud Debugger console, you can see the `helloworld-gke` service in the drop down:

![](../../.gitbook/assets/image%20%2820%29.png)
{% endtab %}

{% tab title="Compute Engine" %}
Follow the [Compute Engine Hello World!](../../getting-started/helloworld/compute-engine.md) to deploy an application in Compute Engine.

SSH into the Compute Engine instance:

```bash
gcloud compute ssh helloworld
```

From the Compute Engine instance, download the Java agent:

```bash
sudo mkdir -p /opt/cdbg
curl -s -o- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz \
  | sudo tar xvz -C /opt/cdbg
```

Run the Java application with the Cloud Debugger agent:

```bash
java -agentpath:/opt/cdbg/cdbg_java_agent.so=--logtostderr=1 \
    -Dcom.google.cdbg.module=helloworld-gce \
    -Dcom.google.cdbg.version=1.0 \
    -jar helloworld.jar
```

In Cloud Debugger console, you can see the `helloworld-gce` service in the drop down:

![](../../.gitbook/assets/image%20%2817%29.png)
{% endtab %}

{% tab title="Non-Google Cloud Environment" %}
You can attach the Cloud Debugger agent to any Java application even if it runs outside of the Google Cloud environment \(whether it's in a container, or on your local laptop, or in another cloud\). Authentication has to be done using Service Account key file rather than using the Machine Credentials.

{% hint style="danger" %}
This only works on a Linux x86 based system.
{% endhint %}

#### Clone

```bash
# Clone the sample repository manually
git clone https://github.com/saturnism/jvm-helloworld-by-example
cd jvm-helloworld-by-example/helloworld-springboot-tomcat
```

#### Build

```bash
mvn package
```

#### Download Agent

```bash
sudo mkdir -p /opt/cdbg
curl -s -o- https://storage.googleapis.com/cloud-debugger/compute-java/debian-wheezy/cdbg_java_agent_gce.tar.gz \
  | sudo tar xvz -C /opt/cdbg
```

#### Create a Service Account

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts create helloworld-app
```

#### Add IAM Permission

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud projects add-iam-policy-binding ${PROJECT_ID} \
  --member serviceAccount:helloworld-app@${PROJECT_ID}.iam.gserviceaccount.com \
  --role roles/clouddebugger.agent
```

#### Create a Service Account Key File

```bash
PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts keys create \
  $HOME/helloworld-app-sa.json \
  --iam-account helloworld-app@${PROJECT_ID}.iam.gserviceaccount.com
```

#### Use Service Account Cloud Debugger Agent

```bash
java -agentpath:/opt/cdbg/cdbg_java_agent.so=--logtostderr=1 \
    -Dcom.google.cdbg.module=helloworld-local \
    -Dcom.google.cdbg.version=1.0 \
    -Dcom.google.cdbg.auth.serviceaccount.enable=true \
    -Dcom.google.cdbg.auth.serviceaccount.jsonfile=$HOME/hellworld-app-sa.json \
    -jar target/helloworld.jar
```
{% endtab %}
{% endtabs %}

## Source Code

Cloud Debugger needs to have access to the application's source code in order for you to add a Snapshot or Logpoint from the Cloud Debugger console. There are severals ways to associating the source code:

* Existing Git Repository
* Source code capture / upload
* Git repository reference from `git.properties`
* IntelliJ Cloud Code plugin

### Git Repository

From Cloud Debugger console, navigate to **Deployed Files** &gt; **Add source code**.

![Cloud Debugger console and find &quot;Add source code&quot;](../../.gitbook/assets/image%20%285%29.png)

Choose an **Alternative source code**.

![Alternative source code choices](../../.gitbook/assets/image%20%289%29.png)

For example, using an existing GitHub repository:

![Select source from GitHub.com](../../.gitbook/assets/image%20%2813%29.png)

Once selected, the contents of the Git repository will be available to navigate.

### Upload

#### Upload from Browser

From Cloud Debugger console, navigate to **Deployed Files** &gt; **Add source code**.

![Cloud Debugger console and find &quot;Add source code&quot;](../../.gitbook/assets/image%20%285%29.png)

Choose an **Alternative source code**.

![](../../.gitbook/assets/image%20%2815%29.png)

Click on **Local files's Select Source**, then simply select the folder/directory that contains the source code.

#### Upload from Command Line

You can use `gcloud` CLI to upload the source code into a Source Captures repository.

Create a Source Captures repository:

```bash
# Enable API
gcloud services enable sourcerepo.googleapis.com

# Create a source capture repository
gcloud source repos create google-source-captures
```

In the **Alternative source code** choices, scroll to the very bottom is **Upload a source code capture to Google servers**.

{% hint style="danger" %}
Do not click on **Select source** yet.
{% endhint %}

![Alternative source code choices](../../.gitbook/assets/image%20%2811%29.png)



Use the command line to upload the source code \(for example, if you deployed the [Helloworld Application](../../getting-started/helloworld/app-engine.md#clone)\):

```bash
# Clone the sample repository manually
git clone https://github.com/GoogleCloudPlatform/java-docs-samples
cd java-docs-samples/appengine-java11/springboot-helloworld

# Upload just the `src/` directory.
# Note that the `branch` value is important and you must use the same value
# that's shown in the UI
gcloud beta debug source upload \
  --project=<FROM THE UI> \
  --branch=<FROM THE UI> \
  src/
```

Once uploaded, click **Select source.**

### **Use git.properties**

You can associate a Git repository using the [`git-commit-plugin`](https://github.com/git-commit-id/git-commit-id-maven-plugin) that generates a `git.properties` file, which contains the information to the Git repository. This only works if the repository is publicly accessible.

```bash
<plugin>
  <groupId>pl.project13.maven</groupId>
  <artifactId>git-commit-id-plugin</artifactId>
  <version>4.0.1</version>
  <executions>
    <execution>
      <goals>
        <goal>revision</goal>
      </goals>
    </execution>
	</executions>
</plugin>
```

Cloud Debugger service will automatically examine this file, and clone the code, and checkout the corresponding revision.

### IntelliJ with Cloud Code

You can use the Cloud Code plugin to directly add a Snapshot point without using the Cloud Debugger console.

Navigate to **Tools** &gt; **Cloud Code** &gt; **Attach Cloud Debugger**.

![](../../.gitbook/assets/image%20%2812%29.png)

Once configured the IntelliJ profile, you can add Snapshot to source code directly from the IDE.

![Debug in the Cloud](../../.gitbook/assets/image%20%288%29.png)

## Learn More

* [Troubleshooting and Debugging Microservices in Kubernetes](https://saturnism.me/talk/troubleshooting-debugging-microservices/)

