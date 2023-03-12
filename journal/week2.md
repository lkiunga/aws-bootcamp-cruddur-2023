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
add this below the app `app = Flask(__name__)`
```
# Initialize tracing and an exporter that can send data to Honeycomb
provider = TracerProvider()
processor = BatchSpanProcessor(OTLPSpanExporter())
provider.add_span_processor(processor)
trace.set_tracer_provider(provider)
tracer = trace.get_tracer(__name__)
```



### Run queries to explore traces within Honeycomb.io
### Instrument AWS X-Ray into backend flask application
### Configure and provision X-Ray daemon within docker-compose and send data back to X-Ray API
### Observe X-Ray traces within the AWS Console
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

