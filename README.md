# OpenTelemetry on K8s

## Distributed Tracing
Distributed tracing is observing requests as they propagate through distributed systems. It’s a diagnostic technique that reveals how a set of services coordinate to handle individual requests. A single trace typically shows the activity for an individual transaction or request within the application being monitored. In aggregate, a collection of traces can show which backend service or database is having the biggest impact on performance as it affects your users’ experiences.

**Little Terminology:**
- A request is how applications, services, and functions talk to one another.
- A trace represents an end-to-end request made up of single or multiple spans. It shows performance data about requests as they flow through the distributed system.
- A span represents work done by a single-service with time intervals and associated metadata.
- Tags are metadata to help contextualize a span
- A root span is the first span in a trace.
- A child span is a subsequent span, which can be nested.

### Distributed Tracing Components

#### Instrumentation
Instrumenting your services environment means adding code to services to monitor and track trace data. There are many open source tools and open instrumentation standards to instrument your environment. OpenTelemetry, part of the Cloud Native Computing Foundation (CNCF), is becoming the one standard for open source instrumentation and telemetry collection.

#### Trace Context
To make the trace identifiable across all the different components in your applications and systems, distributed tracing requires trace context. This means assigning a unique ID to each request, assigning a unique ID to each step in a trace, encoding this contextual information, and passing (or propagating) the encoded context from one service to the next as the request makes its way through an application environment. This lets your distributed tracing tool correlate each step of a trace, in the correct order, along with other necessary information to monitor and track performance.

W3C Trace Context is becoming the standard for propagating trace context across process boundaries. It lets all tracers and agents that conform to the standard participate in a trace, with trace data propagated from the root service all the way to the terminal service.

#### Metrics and Metadata 
A single trace typically captures data about:
- Spans (service name, operation name, duration, and other metadata)
- Errors
- Duration of important operations within each service (such as internal method calls and functions)
- Custom attributes

#### Analysis and visualization
Collecting trace data would be wasted if there were no easy way to analyze and visualize the data across complex architectures. A comprehensive observability platform enables viewing telemetry and business data in one place. It also provides the context to quickly derive meaning and take the right action, and work with the data in ways that are meaningful to the business.

### Sampling
The volume of trace data can grow exponentially over time as the volume of requests increases and as more services are deployed within the environment. To manage the complexity and cost associated with transmitting and storing vast amounts of trace data, organizations can store representative samples of the data for analysis instead of saving all the data. There are two approaches to sampling distributed traces:

#### Head-based Sampling
Makes the decision to collect and store trace data randomly while the root trace is being processed.

**Advantages**
- Works well for applications with low throughput
- Fast and simple to get up and running
- Works well with a blend of monoliths and microservices
- Little-to-no impact on application performance

**Disadvantages**
- Traces with errors or unusual latency can be sampled out and missed because the trace is sampled randomly and the decision is made before the trace has fully completed.


#### Tail-based Sampling
Makes the decision to sample the request when it has completed and all information about the trace has been collected

**Advantages**
- Works well for highly distributed, high-volume app environments
- Captures and analyzes 100% of traces across a distributed system
- Observes every span within a request and the decides which traces are worth saving
- Visualizes data with errors, latency and anomalies

**Disadvantages**
- Requires operational effort in deploying, scaling for on-premise solutions


## OpenTelemetry
[OpenTelemetry]((https://opentelemetry.io/)) is a set of APIs, SDKs, tooling and integrations designed for creating and managing telemetry data such as traces, metrics, and logs. The project provides a vendor-agnostic implementation that can be configured to send telemetry data to the backend(s) of your choice. It supports many open-source projects like Jaeger and Prometheus.
OpenTelemetry is not an observability backend like Jaeger or Prometheus, it supports exporting data to a variety of open-source and commercial back-ends and provides a pluggable architecture so additional technology protocols and formats can be easily added.

### Components
OpenTelemetry project consists of multiple components, these components are made available as single implementation to ease adoption.

#### Specification
Describes the cross-language requirements and expectations for all implementations. The specification defines the following:
- API: Used to generate telemetry data. Defined per data source as well as for other aspects including baggage and propagators.
- SDK: Implementation of the API with processing and exporting capabilities. Defined per data source as well as for other aspects including resources and configuration.
- Data: Defines semantic conventions to provide vendor-agnostic implementations as well as the OpenTelemetry protocol (OTLP).

#### Collector
OpenTelemetry Collector offers a vendor-agnostic implementation on how to receive, process, and export telemetry data. It removes the need to run, operate, and maintain multiple agents/collectors in order to support open-source observability data formats (e.g. Jaeger, Prometheus, etc.). The Collector is the default location instrumentation libraries export their telemetry data.

#### Instrumentation Libraries
OpenTelemetry tries to make every library and application observable out of the box by having them call the OpenTelemetry API directly. Until then, there is a need for a separate library which can inject this information. A library that enables observability for another library is called an instrumentation library. The OpenTelemetry project provides an instrumentation library for multiple languages. All instrumentation libraries support manual (code modified) instrumentation and several support automatic (byte-code) instrumentation.

#### Proto
Language independent interface types (protobuf messages) defined per data source for instrumentation libraries (tracing/metrics/logs) as well as the collectors

## Signoz
[Signoz](https://signoz.io/) is an open source application performance management tool that helps developers monitor their application by providing an easy-to-setup OpenTelemetry based monitoring system. Signoz provide setup for an OLAP backend using either [Clickhouse](https://clickhouse.com/) or [Druid](https://druid.apache.org/) + [Kafka](https://kafka.apache.org/). 

### Signoz Architecture 

**ClickHouse setup Architecture​**
  - OpenTelemetry Collector
  - ClickHouse
  - Query Service
  - Frontend

**Kafka + Druid Setup Architecture​**
  - OpenTelemetry Collector
  - Kafka
  - Stream Processors
  - Apache Druid
  - Query Service
  - Frontend

There are a lot of resources comparing both solutions and I have listed here a few if you were interested:
  - Signoz introducing Clickhouse as a new backend [here](https://dev.to/signoz/launching-support-for-clickhouse-as-storage-backend-for-signoz-1fk9)
  - Hackernews post about Clickhouse vs Druid+Kafka [here](https://news.ycombinator.com/item?id=22869770)
  - Cloudfare blog post comparing Druid and Clickhouse in DNS analytics usecase [here](https://blog.cloudflare.com/how-cloudflare-analyzes-1m-dns-queries-per-second/)
  - Comparison between OLAP systems for big data [here](https://leventov.medium.com/comparison-of-the-open-source-olap-systems-for-big-data-clickhouse-druid-and-pinot-8e042a5ed1c7)

*For more info check Signoz Docs [here](https://signoz.io/docs/architecture/)*

### Installation
First we need to install a local kubernetes cluster, here I am using microk8s from Canonical.
  - Download and install Multipass
  - Download and install Microk8s
  - If using windows we need to run `multipass set local.privileged-mounts=true` to enable mounting of local paths
  - Run `microk8s install`
  - Run `microk8s enable rbac dns helm helm3 storage ingress`
  - Copy k8s configuration to our user directory using `microk8s config > ~/.kube/config` if you want to use `kubectl` or use `microk8s kubectl` instead 

Next, we will install a container attached storage solution to use it as persisted local storage.
  - Run `multipass exec microk8s-vm -- sudo systemctl enable --now iscsid` to Make sure `iscsi` is enabled
  - Run `microk8s enable openebs` and follow instructions to make sure openebs is deployed
  - Since we are only using a single node cluster, we will use the `openebs-hostpath` as storage class using 
  - We need to create a volume for Clickhouse `kubectl apply -f storage-clickhouse-volumes.yaml`
  - We need to create a volume for Kafka+Druid `kubectl apply -f storage-druid-volumes.yaml`

#### Druid + Kafka:
- Clone Signoz github repository using `git clone https://github.com/SigNoz/signoz.git` 
- Kubernetes deployment files are available only for Druid + Kafka at `signoz/deploy/kubernetes` which has folders
  - Platform: Contains the required files to install the Signoz platform on k8s using Kafka and Druid
  - Jobs: Contains configuration to add a druid supervisor to ingest Kafka data and configuring the retention period
  - Otel-Collector: Configuration for the OpenTelemetry collector
- Steps:
  - Mount our cloned signoz repository into `microk8s-vm` or clone the repository again inside the VM. To mount the folder use `multipass mount ./signoz microk8s-vm:~/signoz`
  - Edit the `values.yaml` file:
	- Configuration for Druid database connection can be changed at `configVars.druid_metadata_storage_type`, `configVars.druid_metadata_storage_connector_connectURI`,   `configVars.druid_metadata_storage_connector_user` and `configVars.druid_metadata_storage_connector_password`
	- You can use `postgres.enabled` to control whether you want helm to install postgres for you, configure the connection credentials and database name as well using `postgres.postgresqlUsername`, `postgres.postgresqlPassword`, and `postgres.postgresqlDatabase`.
	- We can follow the tutorial from signoz website [here](https://signoz.io/docs/deployment/helm_chart/)
	  ``` bash 
	  microk8s helm3 dependency update ./signoz/deploy/kubernetes/platform/
	  kubectl create ns platform`
	  microk8s helm3 -n platform install signoz ./signoz/deploy/kubernetes/platform
	  kubectl -n platform apply -Rf ./signoz/deploy/kubernetes/jobs
	  kubectl -n platform apply -f ./signoz/deploy/kubernetes/otel-collector
	    ``` 
	- From `https://github.com/wiremind/wiremind-helm-charts/blob/main/charts/druid/values.yaml`, to change the storage class we need to set `Values.historical.persistence.storageClass` and `Values.middleManager.persistence.storageClass` to `openebs-hostpath`,. However, we need to create a volume to be used by druid as deep storage as well which is not possible because the helm chart doesn't have the necessary configurations to create volumes. We are forced to use other extensions like Cassandra or any cloud storage.
- Notes: This druid helm chart is deprecated and the recommended way is to use `druid-operator` available [here](https://github.com/druid-io/druid-operator) 

#### Clickhouse
- Clone Signoz github repository
- Signoz only provides docker-swarm deployment file for Clickhouse, we will need to use [kompose](https://kompose.io/) to convert them to kubernetes files.
  - Download kompose binary for your system from [kompose releases](https://github.com/kubernetes/kompose/releases)
  - Copy downloaded binary to /signoz/deploy/docker-swarm/clickhouse-setup and rename it to `kompose`
  - Open terminal window, navigate to the same folder
  - Create a new folder to output the converted files `mkdir k8s-files`
  - Run `kompose convert --with-kompose-annotation=false --volumes configMap --out k8s-files`
  - Many files will be created, configuration files will be converted into config-map files.
  - One of query-service-cm(0/1/2)-configmap.yaml and one of clickhouse-cm(0/1/2)-configmap.yaml contains data configuration and we need to change these to use PVC, find these config maps and make note of their names as we will need them later  
  - Create PVC for clickhouse in a file named `clickhouse-pvc.yaml`
    ```yaml
    kind: PersistentVolumeClaim
    apiVersion: v1
    metadata:
      name: clickhouse-pvc
    spec:
      storageClassName: openebs-hostpath
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20G
    ```
  - Create PVC for query-service in a file named `query-service-pvc.yaml`
    ```yaml
    kind: PersistentVolumeClaim
       apiVersion: v1
       metadata:
         name: query-service-pvc
       spec:
         storageClassName: openebs-hostpath
         accessModes:
           - ReadWriteOnce
         resources:
           requests:
             storage: 20G
    ```
  - We need to change the deployment files for clickhouse and query-service to use the PVCs we just created
    - Open file `clickhouse-deployment.yaml`
    - Find the `configMap` in `volumes` with a name matching the name of the configmap file from before and replace the block with:
      ```yaml
  	- persistentVolumeClaim:
        claimName: clickhouse-pvc
      name: clickhouse-storage
  	```
	- Do the same with `query-service-deployment.yaml` but replace the `configMap` block with 
	  ```yaml
	  - persistentVolumeClaim:
          claimName: query-service-pvc
        name: query-service-storage
	  ```
  - Edit `clickhouse-service.yaml` and remove port `8123-tcp` because port `8123` is mapped twice
  - Enable ingress files to allow access to signoz front-end
    - Create file `frontend-external-service.yaml`
	- Paste the following 
	  ```yaml
	  kind: Service
      apiVersion: v1
      metadata:
        name: frontend-external-service
        namespace: ingress
      spec:
        type: ExternalName
        externalName: frontend.signoz.svc.cluster.local
        ports:
        - port: 3000
	  ```
    - Create file `frontend-ingress.yaml`
	- Paste the following 
	  ```yaml
	  apiVersion: networking.k8s.io/v1
      kind: Ingress
      metadata:
        name: ingress-frontend
        namespace: ingress
      spec:
        rules:
        - http:
            paths:
            - path: "/"
              pathType: Prefix
              backend:
                service:
                  name: frontend-external-service
                  port:
                    number: 3000
        ingressClassName: public            
	  ```
  - To control retention period, Clickhouse provide many ways to control the retention of the data in its tables. TTL is one of them and it can be configured for tables or even separate columns. More information can be found [here](https://clickhouse.com/docs/en/engines/table-engines/mergetree-family/mergetree/#table_engine-mergetree-ttl). The table initialization can be found in the config-map with `data: init-db.sql`. Other retention methods can be found [here](https://clickhouse.com/docs/en/faq/operations/delete-old-data/)
  - Next, we start applying those files into k8s
    ```bash
	kubectl create namespace signoz
	kubectl -n signoz apply -f clickhouse-cm0-configmap.yaml -f clickhouse-cm1-configmap.yaml -f clickhouse-pvc.yaml -f clickhouse-deployment.yaml -f clickhouse-service.yaml
	kubectl -n signoz apply -f query-service-cm0-configmap.yaml -f query-service-cm1-configmap.yaml -f query-service-pvc.yaml -f query-service-deployment.yaml -f query-service-service.yaml
	kubectl -n signoz apply -f frontend-configmap.yaml -f frontend-deployment.yaml  -f frontend-service.yaml
	kubectl -n ingress apply -f frontend-external-service.yaml -f frontend-ingress.yaml
	```
  - Check that you can access signoz frontend app using `http://{microk8s-vm-IP}`
  - After that, we can deploy `hotrod` and `load-hotrod` to make sure everything is working.
    - Run `kubectl -n signoz apply -f hotrod-deployment.yaml -f hotrod-service.yaml
	- Edit `load-hotrod-deployment.yaml` file, change the value for `ATTACKED_HOST` to the `http://hotrod.signoz.svc.cluster.local:9000` for `hotrod-service`
	- Run `kubectl -n signoz apply -f load-hotrod-cm0-configmap.yaml -f load-hotrod-deployment.yaml -f load-hotrod-service.yaml`
  - Check the frontend app again, you should start seeing some data.

## Promscale
[Promscale](https://github.com/timescale/promscale) is an observability backend that uses PostgreSQL/TimescaleDB with native support for Prometheus and OpenTelemetry to create one open source storage system and query language for metrics and traces. You can check a performance benchmark comparing TimescaleDB with ClickHouse [here](https://gitlab.com/gitlab-org/incubation-engineering/apm/apm/-/issues/4) and a video of the demo [here](https://youtu.be/cMdQsxolcqc)

### Installation
For Kubernetes deployments, Promscale is recommending we use [tobs](https://github.com/timescale/tobs) which is a tool to install a full observability stack into a kubernetes cluster. This stack includes:
- Kube-Prometheus the Kubernetes monitoring stack
  - Prometheus to collect metrics
  - AlertManager to fire the alerts
  - Grafana to visualize what's going on
  - Node-Exporter to export metrics from the nodes
  - Kube-State-Metrics to get metrics from kubernetes api-server
  - Prometheus-Operator to manage the life-cycle of Prometheus and AlertManager custom resource definitions (CRDs)
- Promscale to store metrics for the long-term and allow analysis with both PromQL and SQL.
- TimescaleDB for long term storage of metrics and provides ability to query metrics data using SQL.
- Promlens tool to build and analyse promql queries with ease.
- Opentelemetry-Operator to manage the lifecycle of OpenTelemetryCollector Custom Resource Definition (CRDs)
- Jaeger Query to visualise the traces

To use tobs, we can just install a CLI tool which will pickup the kube-config file and install the stack automatically, or we can helm charts as shown [here](https://github.com/timescale/tobs/blob/master/chart/README.md). CLI tool usage guide can be found [here](https://github.com/timescale/tobs/tree/master/cli)
- Mount kubeconfig folder to the VM using `multipass mount ./.kube microk8s-vm:~/.kube` 
- Jump into microk8s-vm shell using `multipass shell microk8s-vm`
- Make sure kubectl is installed insnide VM using `sudo snap install --classic kubectl` 
- Run `curl --proto '=https' --tlsv1.2 -sSLf  https://tsdb.co/install-tobs-sh |sh` to download tobs CLI tool
- To modify the default configuration, download the `values.yaml` using `curl -sSLf https://github.com/timescale/tobs/raw/master/chart/values.yaml -o values.yaml` and edit it
- As of now, there's a problem with the `kube-webhook-cert` image used in the `kube-prometheus-stack` so we have to modify the `values.yaml` file.
- Edit `values.yaml` file, add the following inside `kube-prometheus-stack` config.
  ```yaml
  prometheusOperator:
    admissionWebhooks:
      patch: 
        image:
          repository: liangjw/kube-webhook-certgen
          tag: v1.1.1
          sha: ""
  ```
- Run `tobs install -f values.yaml --tracing` to install the observability stack with tracing.
- To access Grafana, we need to create external service and ingress
  - Create file `grafana-external-service` and paste in the following
    ```yaml
    kind: Service
    apiVersion: v1
    metadata:
      name: grafana-external-service
      namespace: ingress
    spec:
      type: ExternalName
      externalName: tobs-grafana.default.svc.cluster.local
      ports:
      - port: 80
    ---
    kind: Ingress
    apiVersion: networking.k8s.io/v1
    metadata:
      name: ingress-grafana
      namespace: ingress
    spec:
      rules:
      - http:
          paths:
            - path: /
              pathType: Prefix
              backend:
                service:
                  name: grafana-external-service
                  port: 
                    number: 80
    
    
    ```
  - Run `kubectl apply -f grafana-external-service` and Grafana should be accessible through the VM IP. Use `admin` as username and get the password from `tobs grafana password`
- To see some traces, we can deploy the `hotrod` and `load-hotrod` from before, we need to modify the URLs for the `JAEGER_ENDPOINT` and `ATTACKED_HOST` respectively
  - Create a new file `hotrod.yaml` and paste the following
    ```yaml
	apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: hotrod
      name: hotrod
    spec:
      replicas: 1
      selector:
        matchLabels:
          io.kompose.service: hotrod
      strategy: {}
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: hotrod
        spec:
          containers:
            - args:
                - all
              env:
                - name: JAEGER_ENDPOINT
                  value: http://tobs-opentelemetry-collector.default.svc.cluster.local:14268/api/traces
              image: jaegertracing/example-hotrod:latest
              name: hotrod
              ports:
                - containerPort: 8080
              resources: {}
          restartPolicy: Always
    status: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: hotrod
      name: hotrod
    spec:
      ports:
        - name: "9000"
          port: 9000
          targetPort: 8080
      selector:
        io.kompose.service: hotrod
    status:
      loadBalancer: {}
	```
  - Create another file `load-hotrod.yaml` and paste the following
    ```yaml
	apiVersion: v1
    data:
      locustfile.py: |
        from locust import HttpUser, task, between
        class UserTasks(HttpUser):
            wait_time = between(5, 15)
    
            @task
            def rachel(self):
                self.client.get("/dispatch?customer=123&nonse=0.6308392664170006")
            @task
            def trom(self):
                self.client.get("/dispatch?customer=392&nonse=0.015296363321630757")
            @task
            def japanese(self):
                self.client.get("/dispatch?customer=731&nonse=0.8022286220408668")
            @task
            def coffee(self):
                self.client.get("/dispatch?customer=567&nonse=0.0022220379420636593")
    kind: ConfigMap
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: load-hotrod
      name: load-hotrod-cm0
    ---
    apiVersion: apps/v1
    kind: Deployment
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: load-hotrod
      name: load-hotrod
    spec:
      replicas: 1
      selector:
        matchLabels:
          io.kompose.service: load-hotrod
      strategy:
        type: Recreate
      template:
        metadata:
          creationTimestamp: null
          labels:
            io.kompose.service: load-hotrod
        spec:
          containers:
            - env:
                - name: ATTACKED_HOST
                  value: http://hotrod.default.svc.cluster.local:9000
                - name: LOCUST_MODE
                  value: standalone
                - name: LOCUST_OPTS
                  value: --headless -u 10 -r 1
                - name: NO_PROXY
                  value: standalone
                - name: QUIET_MODE
                  value: "false"
                - name: TASK_DELAY_FROM
                  value: "5"
                - name: TASK_DELAY_TO
                  value: "30"
              image: grubykarol/locust:1.2.3-python3.9-alpine3.12
              name: load-hotrod
              ports:
                - containerPort: 8089
              resources: {}
              volumeMounts:
                - mountPath: /locust
                  name: load-hotrod-cm0
          hostname: load-hotrod
          restartPolicy: Always
          volumes:
            - configMap:
                name: load-hotrod-cm0
              name: load-hotrod-cm0
    status: {}
    ---
    apiVersion: v1
    kind: Service
    metadata:
      creationTimestamp: null
      labels:
        io.kompose.service: load-hotrod
      name: load-hotrod
    spec:
      ports:
        - name: "8089"
          port: 8089
          targetPort: 8089
      selector:
        io.kompose.service: load-hotrod
    status:
      loadBalancer: {}
	```
	- Apply those files using `kubectl apply -f hotrod.yaml -f load-hotroal.yaml`
	- After a while you can start seeing some data in the Explore screen on Grafana. More on this [here](https://grafana.com/docs/grafana/latest/datasources/jaeger/)