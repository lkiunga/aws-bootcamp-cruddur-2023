# Week 2 — Distributed Tracing

### Instrument our backend flask application to use Open Telemetry (OTEL) with Honeycomb.io as the provider
#### What is Honeycomb
Honeycomb is a tool that provides deep visibility into the performance of your software systems, allowing you to quickly identify and resolve issues that impact the user experience. It does this by collecting and analyzing real-time data on how each component of your system is behaving, so you can gain insights into what's happening "under the hood."

Think of it like a microscope that helps you examine the inner workings of your system. Honeycomb allows you to zoom in on specific areas of your code and see exactly what's happening at a granular level, which can help you pinpoint issues more quickly and improve the overall performance of your application.

1. Create a Honeycomb ACcount using Google Account
#### What is OpenTelemetry (OTEL)
OpenTelemetry (OTEL) is a tool that acts like a fitness tracker for software applications, collecting data on how the application is performing. This helps developers optimize the application's speed and efficiency, and troubleshoot issues for a better user experience.
#### How does Honeycomb and OpenTelemetry(OTEL) work together
OTEL collects data about how the application is performing, while Honeycomb provides a platform for analyzing and visualizing that data in real-time.  OpenTelemetry exporter allows you to send your performance data to Honeycomb. Once the data is in Honeycomb, you can use its powerful analytics and visualization tools to gain deep insights into how your application is performing. You can create custom dashboards to track key performance metrics, set up alerts to notify you when issues arise, and collaborate with your team to quickly resolve any problems.
#### Setup OpenTelemetry on the application
add the following to the `requirements.txt`

```
opentelemetry-api 
opentelemetry-sdk 
opentelemetry-exporter-otlp-proto-http 
opentelemetry-instrumentation-flask 
opentelemetry-instrumentation-requests
```
install the depencies by ;
``` 
cd backedflask #directory
pip install -r requirements.txt
```
Add the following to the app.py
```
from opentelemetry import trace
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.exporter.otlp.proto.http.trace_exporter import OTLPSpanExporter
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
```
add this above the `app = Flask(__name__)`
```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```
Initialize automatic instrumetation of the end points by adding the following code under the `app = Flask(__name__)` 
```
# Initialize automatic instrumentation with Flask
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)
RequestsInstrumentor().instrument()
```
Add Env variable in Docker Compose to ALLOW OpenTelemetry exporter service to send performace data to Honeycomb 
```
OTEL_EXPORTER_OTLP_ENDPOINT: "https://api.honeycomb.io"
OTEL_EXPORTER_OTLP_HEADERS: "x-honeycomb-team=${HONEYCOMB_API_KEY}"
OTEL_SERVICE_NAME: "${HONEYCOMB_SERVICE_NAME}"
```
Set the Honeycomb API keys to be able to visualize the data on the honeycomb account
```
export HONEYCOMB_API_KEY=""
//The service name should be for the particular endpoint you intend to visualize
export HONEYCOMB_SERVICE_NAME="backend-flask"
gp env HONEYCOMB_API_KEY=""
gp env HONEYCOMB_SERVICE_NAME="backend-flask"
```
** insert Image for HoneyComb Dashboard
### Run queries to explore traces within Honeycomb.io
Honeycomb has a powerful set of query capabilities that allow you to explore and analyze your data in a variety of ways. Here are some of the key capabilities:
1. Advanced filtering: Honeycomb allows you to filter your data based on multiple criteria, including time ranges, values in specific columns, and complex logical expressions.
2. Aggregations: You can use Honeycomb's built-in aggregation functions to summarize your data and calculate statistics like counts, averages, and percentiles.
3. Grouping and pivoting: Honeycomb allows you to group your data by one or more columns and pivot your results to display them in different ways.
4. Visualization: You can visualize your query results using a variety of chart types, including line charts, bar charts, and heatmaps.
5. Saved queries: You can save your queries and share them with other members of your team, allowing you to collaborate and gain insights together.
6. Alerts: Honeycomb allows you to set up alerts based on specific query results, so you can be notified when certain conditions are met.

** insert image of sample honeycomb query
### Instrument AWS X-Ray into backend flask application
AWS X-Ray is a powerful tool for developers working in a microservices architecture. AWS X-Ray is a distributed tracing service that helps developers analyze and debug applications in a microservices architecture. With AWS X-Ray, developers can trace requests made to their application as they travel across different services, identify performance bottlenecks and errors, and visualize the end-to-end architecture of their application.
steps to Instrument AWS X-Ray for flask
Add the AWS xray to the `requirements.txt` file
```
aws-xray-sdk
```
install the Python DEPENDENCIES; `pip install -r requirements.txt`
add the following to `app.py` to setup observability points;
Before the ` app = Flask(__name__)` add the following
```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
```
after the ` app = Flask(__name__)` add the following
```
XRayMiddleware(app, xray_recorder)
```
Setup the sampling rules by adding the following code to `aws/json/xray.json`
```
{
  "SamplingRule": {
      "RuleName": "Cruddur",
      "ResourceARN": "*",
      "Priority": 9000,
      "FixedRate": 0.1,
      "ReservoirSize": 5,
      "ServiceName": "Cruddur",
      "ServiceType": "*",
      "Host": "*",
      "HTTPMethod": "*",
      "URLPath": "*",
      "Version": 1
  }
}
```
Run the following code on the terminal to create a group on Aws X-ray - this important since it allows developers to organize traces and visualize the performance of application at different levels of granularity. By grouping traces based on a specific criteria, such as service name, environment, or user ID, itis easy to quickly identify trends and anomalies in the performance of your application.
```
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"backend-flask\")
 ```
 create a rule using the terminal by running the below command and using the previous generated key. NB- you can achieve this using the AWS X-Ray Gui interface(keeps changing which can be straineous hence the preference to use the terminal.
 ```
 aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
 ```
### Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
Reasons why need X-Ray Daemon  configured;
AWS X-Ray requires a daemon service to connect to Docker Compose because it needs to collect and analyze tracing data from all the containers running in the Compose environment. 
1. Inter-container communication: In a Docker Compose environment, containers communicate with each other over a network bridge. The X-Ray daemon intercepts this network traffic and collects trace data from all the containers.
2. Inter-container communication: In a Docker Compose environment, containers communicate with each other over a network bridge. The X-Ray daemon intercepts this network traffic and collects trace data from all the containers.
3. Resource utilization: The X-Ray daemon is designed to use minimal system resources, making it easy to run alongside other services in a Docker Compose environment.
Add the following to docker compose to pull the amazon/aws-xray-daemon image that is maintained by Amazon and is designed specifically for running the AWS X-Ray daemon. It contains all the necessary dependencies and configurations to run the daemon in a containerized environment.

```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "us-east-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
  ```
Additionally add the following variables to the backend environment variables on docker compose
```
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```
### Observe X-Ray traces within the AWS Console
** insert image
The AWS X-ray keeps logging to cloud watch but after a couple refresh it opens the old AWS XRay interface.
### Integrate CloudWatch Logs

Add to `requirements.txt`
```
watchtower
```
`cd /bacK` directory
Run `pip install -r requirements.txt`

Add the imports from cloudwatch to app.py
```
import watchtower
import logging
from time import strftime
```
Before the ` app = Flask(__name__)` add the following
```
Configuring Logger to Use CloudWatch
LOGGER = logging.getLogger(__name__)
LOGGER.setLevel(logging.DEBUG)
console_handler = logging.StreamHandler()
cw_handler = watchtower.CloudWatchLogHandler(log_group='cruddur')
LOGGER.addHandler(console_handler)
LOGGER.addHandler(cw_handler)
LOGGER.info("test Log")
```
Below the ` app = Flask(__name__)` add the following
```
@app.after_request
def after_request(response):
    timestamp = strftime('[%Y-%b-%d %H:%M]')
    LOGGER.error('%s %s %s %s %s %s', timestamp, request.remote_addr, request.method, request.scheme, request.full_path, response.status)
    return response
```
Go to `homeactivities.py` add the following code after the def function

```
    lOGGER:info("HomeActivities")
```
Set the env var in your backend-flask for `docker-compose.yml`
```
 AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
 AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
 ```
 **insert images
 #### Understanding the use case of AWS CloudWatcg vs AWS X-Ray

Let's say you have a microservice architecture deployed in AWS, where several microservices interact with each other to fulfill requests. You want to monitor the performance of the system and identify any bottlenecks or errors.

 -In this scenario, you can use CloudWatch to monitor the health and performance of your microservices. For example, you can use CloudWatch to monitor the CPU utilization, memory usage, and network traffic of your EC2 instances, and set alarms to notify you when certain thresholds are breached. You can also use CloudWatch Logs to collect and monitor logs from your microservices and applications, and use CloudWatch Metrics to create custom dashboards and charts to visualize the performance of your system.

 -However, if you want to understand the flow of requests through your microservices and identify any issues, you can use AWS X-Ray. For example, you can use X-Ray to trace a specific request as it travels through your microservices, and see how long each service takes to process the request. You can also see any errors or exceptions that occur during the request processing, and drill down into the root cause of the problem.
 
### Integrate Rollbar for Error Logging

Add to `requirements.txt`
```
blinker
rollbar
```
`cd /bacK` directory
Run `pip install -r requirements.txt`

Add access tokens - grap from the Flask SDK in rollbar
```
export ROLLBAR_ACCESS_TOKEN=""
gp env ROLLBAR_ACCESS_TOKEN=""
```
Add the imports from rollbar to app.py
```
import rollbar
import rollbar.contrib.flask
from flask import got_request_exception
```
Add the following code to implement the rollbar access tokens

```
rollbar_access_token = os.getenv('ROLLBAR_ACCESS_TOKEN')
@app.before_first_request
def init_rollbar():
    """init rollbar module"""
    rollbar.init(
        # access token
        rollbar_access_token,
        # environment name
        'production',
        # server root directory, makes tracebacks prettier
        root=os.path.dirname(os.path.realpath(__file__)),
        # flask already sets up logging
        allow_logging_basic_config=False)

    # send exceptions from `app` to rollbar, using flask's signal system.
    got_request_exception.connect(rollbar.contrib.flask.report_exception, app)
```
Add the rollbar sample end point 
```
@app.route('/rollbar/test')
def rollbar_test():
    rollbar.report_message('Hello World!', 'warning')
    return "Hello World!"
```

### Trigger an error an observe an error with Rollbar
Simulate an error by Commenting out`return` on the Homeactivities function.
The error is visible on rollbar and we can be able to debug it from here. THis is especially useful when debugging in production since Debugging isset to false in production mode.
**** Add rolebar GUI image
### Install WatchTower and write a custom logger to send application log data to - CloudWatch Log group

