# Instrument your applications using OpenTelemetry

## Java example

### pom.xml

```xml
<dependencies>
    <dependency>
        <groupId>io.opentelemetry.instrumentation</groupId>
        <artifactId>opentelemetry-spring-boot-starter</artifactId>
    </dependency>
</dependencies>
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>io.opentelemetry.instrumentation</groupId>
            <artifactId>opentelemetry-instrumentation-bom</artifactId>
            <version>2.16.0</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>
```

### application.properties

```ini
otel.sdk.disabled=true
otel.exporter.otlp.endpoint=${OTEL_COLLECTOR_ENDPOINT}
otel.exporter.otlp.logs.protocol=grpc
otel.exporter.otlp.metrics.protocol=grpc
otel.exporter.otlp.traces.protocol=grpc
```

## TypeScript example

### src/services/otel.ts

```ts
import { WebTracerProvider } from "@opentelemetry/sdk-trace-web";
import { BatchSpanProcessor } from "@opentelemetry/sdk-trace-base";
import { OTLPTraceExporter } from "@opentelemetry/exporter-trace-otlp-http";
import { registerInstrumentations } from "@opentelemetry/instrumentation";
import { FetchInstrumentation } from "@opentelemetry/instrumentation-fetch";
import { DocumentLoadInstrumentation } from "@opentelemetry/instrumentation-document-load";
import { resourceFromAttributes } from "@opentelemetry/resources";
import { ATTR_SERVICE_NAME } from "@opentelemetry/semantic-conventions";
import { W3CTraceContextPropagator } from "@opentelemetry/core";

const resource = resourceFromAttributes({
  [ATTR_SERVICE_NAME]: "react-frontend",
});

const otlpExporter = new OTLPTraceExporter({
  url: import.meta.env.VITE_OTLP_COLLECTOR_URL,
});

const tracerProvider = new WebTracerProvider({
  resource,
  spanProcessors: [new BatchSpanProcessor(otlpExporter)],
});

tracerProvider.register({
  propagator: new W3CTraceContextPropagator(),
});

registerInstrumentations({
  tracerProvider,
  instrumentations: [
    new DocumentLoadInstrumentation(),
    new FetchInstrumentation({
      propagateTraceHeaderCorsUrls: [/./],
    }),
  ],
});
```

### main.tsx

```tsx
import "./services/otel.ts";
...
```

## Deploy the SigNoz Helm Chart to a Minikube cluster

This Helm chart contains a visualization platform, OpenTelemtry collector (which is where our applications will send their telemetry data) and ClickHouse (an analytical processing database for persistence).

1. Provision Minikube cluster

   ```bash
   minikube start
   ```

2. Deploy Helm chart

   ```bash
   helm upgrade --install -n platform --create-namespace signoz-release ./signoz -f signoz/values.yaml
   ```

3. Open `k9s` and create 2 **port-forwards** from the UI for the OTEL collector **Service** on ports 4317 (otlp/gRPC) and 4318 (otlp/http) with **Shift + F**.
