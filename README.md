# OpenLLMetry and Google Cloud Platform Integration Tutorial

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
Setting up observability is crucial for quick troubleshooting, cost optimization, and code efficiency of your applications. This is no different for applications that utilize LLMs. Fortunately, [OpenTelemetry](https://opentelemetry.io/) developed an observability framework specifically for AI use cases - [OpenLLMetry](https://github.com/traceloop/openllmetry). OpenLLMetry has multiple integrations available for use. In this guide, we will learn how to set up OpenLLMetry in your LLM Python application.

Tools:
* OpenLLMetry
* Google Cloud Trace Explorer
* Python
* Gemini 3.1 Flash Lite
* uv

## Prerequisites
Before starting this tutorial, ensure you have:
* A Google Cloud project with billing enabled.
* The Google Cloud CLI (`gcloud`) installed and initialized.
* An existing `uv` Python project with your LLM application logic and the `google-genai` package already configured.

## Tutorial 
You have your Python LLM project managed by uv ready to go. Now we need to step up observability and tracing for all your LLM calls

#### **Step 1**: Install OpenLLMetry using uv

```bash
uv add traceloop-sdk
```

#### **Step 2**: Install Google OpenTelemetry Exporter
Since we are using Google Cloud Trace, we do not need to set any Traceloop environment variables. We can instead install the Google OpenTelemetry Exporter - which OpenLLMetry will automatically detect

```bash
uv add \
opentelemetry-exporter-gcp-trace \
opentelemetry-exporter-gcp-monitoring \
opentelemetry-exporter-gcp-logging
```

#### **Step 3**: Import All Relevant Modules
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
<figure align="center">
  <img src="./assets/api_trace.png" alt="api trace ui screenshot" width="800"/>
  <figcaption><em>Figure 1: Cloud Trace API on APIS & Services UI</em></figcaption>
</figure>

#### **Step 6**: Enable Trace Storage (the option should be available in the Trace explorer UI)
<figure align="center">
  <img src="./assets/enable_cloud_trace_storage.png" alt="enable cloud trace storage" width="800"/>
  <figcaption><em>Figure 2: Message After Successfully Enabling Cloud Trace Storage</em></figcaption>
</figure>

#### **Step 7**: Ensure your service account has all required roles (if using GCP). Some may include:
* Cloud Telemetry Metric Writer
* Cloud Trace Agent
* Cloud Trace User
* Logs Writer
* Monitoring Editor

#### **Step 8**: Set up necessary GCP API Keys or use gcloud auth
Google Cloud Platform requires API keys provided as environment variables or proper authentication when using GCP services locally. One way to login is to run the following command in your terminal:
```bash
gcloud auth application-default login
```

#### **Step 9**: Run your python app
```bash
uv run main.py
```

or if using a cloud run job, execute the job
```bash
gcloud run jobs execute LLM_APP_JOB_NAME
```

#### **Step 10**: View Traces in Trace Explorer
<figure align="center">
  <img src="./assets/capturing_trace.png" alt="view cloud traces" width="800"/>
  <figcaption><em>Figure 3: Viewing Trace Explorer UI</em></figcaption>
</figure>
<br>
<br>

If you click on one of the trace spans, you can view the recorded metrics. Figure 4 shows the prompt, token count, and other metadata passed to the gemini model api call.

<figure align="center">
  <img src="./assets/metadata_llm.png" alt="view llm traces" width="800"/>
  <figcaption><em>Figure 4: Viewing token usage and prompt metadata within a Trace Explorer span</em></figcaption>
</figure>

### Configuring OpenLLMetry
Configuring OpenLLMetry can be accomplished by setting environment variables in your Python code. One reason you may want set a variable is to ensure rate limits are not surpassed. For example, if the GCP Monitoring API receives more requests than the minimum sampling window, the calls will throw an error. One way to circumvent these errors is by enabling metric aggregation with the OTEL_METRIC_EXPORT_INTERVAL variable.

```python
import os
os.environ["OTEL_METRIC_EXPORT_INTERVAL"] = "60000"
```

Other useful environment variables include:

| Variable | Description |
| :--- | :--- |
| `OTEL_TRACES_SAMPLER` | Defines how traces are sampled.<br/>Options include: `always_on`, `always_off`, or `parentbased_always_on`. |
| `OTEL_TRACES_SAMPLER_ARG` | Exports a random sample of traces.<br/>Set this value as a fraction between `0.0` and `1.0`. |
| `OTEL_BSP_EXPORT_TIMEOUT` | The maximum time (in milliseconds) allowed for a trace batch to export before failing. |

### Let's Put It All Together!
Here is a an example of what the module could look like:

```python
import os
from google import genai
from traceloop.sdk import Traceloop
from opentelemetry.exporter.cloud_trace import CloudTraceSpanExporter
from opentelemetry.exporter.cloud_monitoring import CloudMonitoringMetricsExporter
from opentelemetry.exporter.cloud_logging import CloudLoggingExporter

# Configure rate limit sampling to prevent errors with GCP Monitoring API
os.environ["OTEL_METRIC_EXPORT_INTERVAL"] = "60000"

# Initialize exporters with proper instantiation
trace_exporter = CloudTraceSpanExporter()
metrics_exporter = CloudMonitoringMetricsExporter()
logs_exporter = CloudLoggingExporter()

# Initialize Traceloop
Traceloop.init(
    app_name="gemini-observability-service",
    exporter=trace_exporter,
    metrics_exporter=metrics_exporter,
    logging_exporter=logs_exporter
)

  def generate(prompt: str, model_name: str = 'gemini-3.1-flash-lite') -> str:
    client = genai.Client()
    response = client.models.generate_content(
        model=model_name, # Updated for production stability
        contents=prompt,
    )
    return response.text

if __name__ == "__main__":
  prompt = "assess the importance of writing a tutorial on observability for llm applications"
  response = generate(prompt)
  print(response)

```

### ✅ Verifying Success
When you run the script, Traceloop will output initialization logs to your terminal, followed by the Gemini response:

```text
Traceloop exporting traces to custom exporter
Traceloop exporting metrics to a custom exporter
Observability in LLM applications is vital because it allows developers to track non-deterministic model behaviors, monitor token costs, and debug latency bottlenecks in real-time.
```

## 📖 Further Reading
If you're interested in diving deeper, here are some resources on telemetry, observability, and authentication:
* **[Running OpenTelemetry at Scale: Architecture Patterns for 100s of Services](https://sematext.com/blog/running-opentelemetry-at-scale-architecture-patterns-for-100s-of-services/):** A comprehensive breakdown of OpenTelemetry architecture, context propagation, and span lifecycle management.
* **[Google Cloud Observability Architecture Guide](https://cloud.google.com/products/observability):** Learn how to architect centralized logging and metric dashboards at enterprise scale across multiple Google Cloud projects.
* **[An Introduction to Observability for LLM-based applications using OpenTelemetry](https://opentelemetry.io/blog/2024/llm-observability/):** An industry look into why traditional APM tooling fails to capture non-deterministic LLM behaviors and how semantic conventions solve it.
* **[How Application Default Credentials works](https://docs.cloud.google.com/docs/authentication/application-default-credentials):** Learn how to transition from local `application-default login` workflows to secure, keyless IAM configurations using Workload Identity in production.
## 📚 References
* [OpenLLMetry GCP Integration Docs](https://www.traceloop.com/docs/openllmetry/integrations/gcp)
* [OpenLLMetry Docs](https://github.com/traceloop/openllmetry)
* [Google Cloud Observability Docs](https://docs.cloud.google.com/stackdriver/docs)
