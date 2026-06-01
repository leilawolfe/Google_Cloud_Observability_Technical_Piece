# OpenLLMetry and Google Cloud Platform Integration Tutorial (UNFINISHED)

<div align="center">
  <table>
    <tr>
      <td align="center" valign="center">
        <img src="https://raw.githubusercontent.com/traceloop/openllmetry/main/img/logo-dark.png" alt="OpenLLMetry Logo" width="180"/>
      </td>
      <td align="center" valign="center" style="font-size: 24px; font-weight: bold; padding: 0 20px;">
        +
      </td>
      <td align="center" valign="center">
        <img src="https://www.gstatic.com/cgc/google-cloud-logo.svg" alt="Google Cloud Platform Logo" width="180"/>
      </td>
    </tr>
  </table>
</div>

## Introduction
Setting up observability is crucial for efficient troubleshoot, cost optimization, and code efficiency your applications. This is no different for applications that utilize LLMs. Fortunately, [OpenTelemetry](https://opentelemetry.io/) developed an observability framework specifically for AI use cases - [OpenLLMetry](https://github.com/traceloop/openllmetry). OpenLLMetry has multiple integrations available for use. In this guide, we will learn how to set up OpenLLMetry in your LLM Python application.

Tools:
* OpenLLMetry
* Google Cloud Trace Explorer
* Python
* Gemini 3.1 Flash Lite
* Uv

## Tutorial 
You have your Python LLM project managed by uv ready to go. Now we need to step up observability and tracing for all your LLM calls

#### **Step 1**: Install OpenLLMetry using uv

```bash
uv add traceloop-sdk
```

#### **Step 2**: Install Google OpenTelemtry Exporter
Since we are using Google Cloud Trace, we do not need to set any Traceloop environment variables. We can instead install the google open telemetry exporter - which OpenLLMetry will automatically detect

```bash
uv add \
opentelemetry-exporter-gcp-trace \
opentelemetry-exporter-gcp-monitoring \
opentelemetry-exporter-gcp-logging \
```

#### **Step 3**: Import Traceloop and Relevant in Your Code
Add the following code to the entry point of your script. In this case, I added it to the top of my main.py file.
You will want to add an import statement
```python
from traceloop.sdk import Traceloop
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
from opentelemetry.exporter.cloud_monitoring import CloudMonitoringMetricsExporter
from opentelemetry.exporter.cloud_logging import CloudLoggingExporter
```
#### **Step 4**: Initialize Traceloop in Your Code
Add the following code to the entry point of your script. In this case, I added it to the top of my main.py file.
You will want to add an initialization call
```python
trace_exporter = CloudTraceSpanExporter()
metrics_exporter = CloudMonitoringMetricsExporter()
logs_exporter = CloudLoggingExporter()

Traceloop.init(
  app_name="your-app-service",
  exporter=trace_exporter,
  metrics_exporter=metrics_exporter,
  logging_exporter=logs_exporter
)
```

#### **Step 5**: Ensure Cloud Trace API is enabled for your project
<img src="./assets/api_trace.png" alt="api trace ui screenshot" width="800" style="border-radius: 100%;"/>

#### **Step 6**: Enable Trace Storage (the option should be available in the Trace explorer UI)
<img src="./assets/enable_cloud_trace_storage.png" alt="enable cloud trace storage" width="800" style="border-radius: 100%;"/>

#### **Step 7**: Ensure your service account has all required roles (if using GCP). Some may include:
* Cloud Telemetry Metric Writer
* Cloud Trace Agent
* Cloud Trace User
* Logs Writer
* Monitoring Editor

#### **Step 8**: Set up necessary GCP API Keys or use `gcloud auth`

#### **Step 8**: Run your python app
```bash
uv run main.py
```

or if using a cloud run job, execute the job
```bash
gcloud run jobs execute LLM_APP_JOB_NAME
```

#### **Step 9**: View Traces in Trace Explorer
<img src="./assets/capturing_trace.png" alt="view cloud traces" width="800" style="border-radius: 100%;"/>

If you click on one of the trace spans, you can view the recorded metrics. In the following example, I can see the prompt, token count, and other metadata passed to the gemini model api call.

<img src="./assets/metadata_llm.png" alt="view llm traces" width="800" style="border-radius: 100%;"/>

### Configuring OpenLLMetry
Configuring OpenLLMetry can be accomplished by setting environment variables in your Python code. One reason you may want set a variable is to ensure rate limits are not surpassed. For example, if the GCP Monitoring API receives more requests than the minimum sampling window, the calls will throw an error. One way to circumvent this errors is by enabling metric aggregation with the OTEL_METRIC_EXPORT_INTERVAL

```python
import os
os.environ["OTEL_METRIC_EXPORT_INTERVAL"] = "60000"
```

Other useful environment variables include
| Variable | Description |
|   ---    |     -----   |
OTEL_TRACES_SAMPLER |  defines how traces are sampled |
OTEL_TRACES_SAMPLER_ARG | will export a random sample of traces. set as a fraction |
OTEL_BSP_EXPORT_TIMEOUT | maximum time (ms) allowed for a trace batch to export before failing

## References
* [OpenLLMetry GCP Integration Docs](https://www.traceloop.com/docs/openllmetry/integrations/gcp)
* [OpenLLMetry Docs](https://github.com/traceloop/openllmetry)
* [Google Cloud Observability Docs](https://docs.cloud.google.com/stackdriver/docs)
