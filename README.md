# Google Cloud Observability and OpenLLMetry Deep-Dive (UNFINISHED)
## Introduction
LLMs can be costly and can be prone to hallucinations. Best to monitor token usage, model drift, and cost.

Tools:
* OpenLLMetry
* Google Cloud Observability
* Gemini 3.1 Flash Lite

## Connection 
You have your Python LLM project managed by uv ready to go. Now we need to step up observability and tracing for all your LLM calls

Step 1: Install OpenLLMetry using uv
`uv add traceloop-sdk`

Step 2: Install Google OpenTelemtry Exporter
Since we are using Google Cloud Trace, we do not need to set any Traceloop environment variables. We can instead install the google open telemetry exporter - which OpenLLMetry will automatically detect
`uv add opentelemetry-exporter-gcp-trace`

Step 3: Initialize Traceloop in Your Code
Generally, you will want to add the following lines of code wherever you make a generate_content() call to a gemini model
You will want to add an import statement
```
from traceloop.sdk import Traceloop
```
and an initialization call
```
Traceloop.init(
  app_name="your-app-service",
)
```

Step 4: Ensure Cloud Trace API is enabled for your project
<img src="./assets/api_trace.png" alt="Leila Wolfe" width="800" style="border-radius: 100%;"/>

Step 5: Run your python app
```
uv run main.py
```

or if using a cloud run job, execute the job
```
gcloud run jobs execute LLM_APP_JOB_NAME
```

## References
* [Google Cloud Observability Docs](https://docs.cloud.google.com/stackdriver/docs)
* [OpenLLMetry Docs](https://github.com/traceloop/openllmetry)
