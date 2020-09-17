# Runtime Environments

## Basics

<table>
  <thead>
    <tr>
      <th style="text-align:left"></th>
      <th style="text-align:left">Cloud Functions</th>
      <th style="text-align:left">App Engine</th>
      <th style="text-align:left">Cloud Run</th>
      <th style="text-align:left">Kubernetes Engine</th>
      <th style="text-align:left">Compute Engine</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align:left"><b>Fully Managed</b>
      </td>
      <td style="text-align:left">FaaS</td>
      <td style="text-align:left">PaaS</td>
      <td style="text-align:left">CaaS/PaaS</td>
      <td style="text-align:left">Kubernetes Clusters</td>
      <td style="text-align:left">Virtual Machines</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Deployable Artifact</b>
      </td>
      <td style="text-align:left">Source or JAR</td>
      <td style="text-align:left">Source or JAR</td>
      <td style="text-align:left">Container Image</td>
      <td style="text-align:left">Container Image</td>
      <td style="text-align:left">Anything, and Container Image</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Locality</b>
      </td>
      <td style="text-align:left">Regional</td>
      <td style="text-align:left">Regional</td>
      <td style="text-align:left">Regional</td>
      <td style="text-align:left">Zonal/Regional</td>
      <td style="text-align:left">Zonal/Regional</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Billing Units</b>
      </td>
      <td style="text-align:left">Instance execution time seconds and Invocations</td>
      <td style="text-align:left">Instance up time minutes</td>
      <td style="text-align:left">Instance execution time seconds</td>
      <td style="text-align:left">Control plane and VM instance hours</td>
      <td style="text-align:left">VM instance hours</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>vCPU</b>
      </td>
      <td style="text-align:left">1</td>
      <td style="text-align:left">1, up to 4.8GHz</td>
      <td style="text-align:left">Up to 2</td>
      <td style="text-align:left">Up to 416 per node, and up to 5000 nodes.</td>
      <td style="text-align:left">0.5 to 416</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Memory</b>
      </td>
      <td style="text-align:left">Up to 2GB</td>
      <td style="text-align:left">Up to 2GB</td>
      <td style="text-align:left">Up to 4GB</td>
      <td style="text-align:left">Up to 11TB per node.</td>
      <td style="text-align:left">Up to 11TB</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Disk</b>
      </td>
      <td style="text-align:left">Writable <code>/tmp</code> directory</td>
      <td style="text-align:left">Writable <code>/tmp</code> directory</td>
      <td style="text-align:left">Writable <code>/tmp</code> directory</td>
      <td style="text-align:left">Attach Tmpfs/PD/SSD</td>
      <td style="text-align:left">Attach PD/SSD</td>
    </tr>
    <tr>
      <td style="text-align:left"><b>Use For</b>
      </td>
      <td style="text-align:left">
        <p>Webhooks</p>
        <p>Event Handlers</p>
        <p>Tasks</p>
      </td>
      <td style="text-align:left">
        <p>Web Apps</p>
        <p>Microservices</p>
        <p>Event Handlers</p>
        <p>Tasks</p>
      </td>
      <td style="text-align:left">
        <p>Web Apps</p>
        <p>Microservices</p>
        <p>Event Handlers</p>
        <p>Tasks</p>
      </td>
      <td style="text-align:left">
        <p>Any container workload</p>
        <p>JEE applications</p>
        <p>Microservices</p>
      </td>
      <td style="text-align:left">
        <p>Any workload</p>
        <p>JEE applications</p>
        <p>Databases</p>
      </td>
    </tr>
  </tbody>
</table>

## Application Lifecycle

|  | Cloud Functions | App Engine | Cloud Run | Kubernetes Engine | Compute Engine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Liveness Check** | Port is listening | /\_ah/health | Port is listening | Liveness Probe | Manual |
| **Readiness/Warmup Check** | No | /\_ah/warmup | No | Readiness Probe | Manual |
| **Graceful Shutdown** | No Signal | /\_ah/stop | No Signal | `SIGTERM` or custom hooks | Manual |

## Scaling

|  | Cloud Functions | App Engine | Cloud Run | Kubernetes Engine | Compute Engine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Scaling** | 0 to N in seconds | 0 to N in seconds | 0 to N in seconds | 1 to N in seconds or minutes | 1 to N in minutes |
| **Autoscaling** | Yes | Yes | Yes | Yes, with HPA and Cluster Autoscaler | Yes, with Managed Instance Group |
| **Scaling Min/Max** | No | Yes | Yes \(alpha\) | Yes, with HPA and Cluster Autoscaler | Yes, with Managed Instance Group |
| **Manual Scaling** | No | Yes | No | Yes | Yes |

## Load Balancing

|  | Cloud Functions | App Engine | Cloud Run | Kubernetes Engine | Compute Engine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Network Load Balancer** | No | No | No | Yes, with `Service` | Yes, with Network Load Balancer |
| **HTTP** | Yes | Yes | Yes | Yes, with `Ingress` | Yes, with HTTP\(s\) Load Balancer |
| **HTTPs** | Yes | Yes | Yes | Yes, with `Ingress` | Yes, with HTTP\(s\) Load Balancer  |
| **Custom Domain** | No | Yes | Yes | Yes, manual configuration | Yes, manual configuration |
| **Managed SSL Certificate** | Yes | Yes | Yes | Yes, with `ManagedCertificate` | Yes, with HTTP\(s\) Load Balancer |

## Networking

|  | Cloud Functions | App Engine | Cloud Run | Kubernetes Engine | Compute Engine |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Use VPC | Yes | Yes | Yes | Yes | Yes |
| Expose on VPC Only | Yes | No | No | Yes | Yes |
| Internal VPC Only Load Balancing | No | No | No | Yes | Yes |
