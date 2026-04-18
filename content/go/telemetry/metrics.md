+++
title = 'Metrics'
date = '2026-01-17T10:46:32-05:00'
weight = 30
draft = false
+++

Metrics are essential tools to help you understand the behavior and performance of your application. They provide insight into your system's memory usage, CPU load, and goroutine counts. The Go runtime exposes performance data through its `runtime` and `net/pprof` packages.

## Prometheus

Prometheus is the most popular third-party libraries that can define and collect metrics.

### Install and run

1. Download the Prometheus client server and run it. This command maps port 9091 on your machine to port 9090 in the container. The `-v` command mounts your configuration file inside the container at `etc/prometheus/`:
   
   ```bash
   docker pull prom/prometheus
   docker run -p 9091:9090 -v /<path>/<to>/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus
   ```
1. Open `localhost:9091` on your machine to view the Prometheus dashboard.
1. Start your server application, then go to `<server-addr>:<port>/metrics` in the browser to view raw Prometheus metrics.


docker run -p 9091:9090 -v /home/ryanseymour/Development/go-projects/system-programming/telemetry/prometheus.yml:/etc/prometheus/prometheus.yml prom/prometheus




### Types of metrics

Metric collection helps you derive actionable insights. Choosing the right metric helps you understand your application and make informed changes.

#### Counter

A counter is a number that only goes up and resets to 0 when the process restarts. Use this for tracking occurrences of events, such as the number of times a user performs an action on your site. Other examples include the following:
- Total HTTP requests
- Total errors
- Jobs completed
- Bytes sent


You almost never graph the raw value. You graph the rate:

```bash
rate(http_requests_total[5m])
```

#### Gauge

A gauge is a number that can go up and down that represents a value right now. It is similar to a thermometer. Use it for values that flucuate over time. For example:
- Current memory usage
- Number of goroutines
- Concurrent sessions
- Queue depth
- In-flight requests
- Sensor readings, such as CPU load


How you use it in queries:

```bash
avg_over_time(inflight_requests[5m])
max(inflight_requests)
```

#### Histogram 

A histogram counts how many observations fall into configurable buckets and also provides a sum of all observed values. Use a histogram when you need to understand the distribution of a metric, not just the average. Histograms can instances, pods, and regions.

You can find how the events are spread out. Use them to track the following events:
- Latency of requests
- Size of responses
- Job runtimes

Example query (p95 latency):

```bash
histogram_quantile(
  0.95,
  sum by (le) (rate(http_request_duration_seconds_bucket[5m]))
)
```

#### Summary

A summary can calculate the sliding window quantiles instead of providing buckets, and they are more computation intensive than histograms. Typical use cases include the following:
- Single-instance latency visibility
- Debugging
- When you don’t need aggregation

For example, here is sample output:

```bash
request_duration_seconds{quantile="0.5"} 0.120
request_duration_seconds{quantile="0.9"} 0.340
request_duration_seconds{quantile="0.99"} 0.900
```

Summaries cannot be aggregated across instances, so do not use them for distributed systems.

### Server with Prometheus metrics

1. Install the Prometheus client:

   ```bash
   go get github.com/prometheus/client_golang/prometheus
   go get github.com/prometheus/client_golang/prometheus/promhttp
   ```

2. Create a variable that defines a Prometheus counter. `CounterVec` is a counter that is parameterized by labels. The label here is `status_code`. This creates a counter for all status codes (`http_requests_processed{status_code="200"}`):


    ```go
    var (
    	requestsProcessed = prometheus.NewCounterVec(
    		prometheus.CounterOpts{
    			Name: "http_requests_processed",
    			Help: "Total number of processed HTTP requests.",
    		}, []string{"status_code"})
    )
    ```

3. Register the metric that you created with Prometheus. `MustRegister` registers the metric with Prometheus's default registry:

    ```go
    func init() {
        prometheus.MustRegister(requestsProcessed)
    }
    ```

4. The `main` method creates the handlers to collect the metrics and manage the Prometheus counter:
   1. Expose the `/metrics` endpoint. This registers the HTTP endpoint with the server. `promhttp.Handler()` collects all registered metrics and formats them in Prometheus text format.
   2. Registers a handler inline:
      1. Add artificial latency to the request.
      2. Random status code selector. If the request is made in an even second, it returns a `500` error. Otherwise, it returns a `200` status code.
   3. Increments the Prometheus counter. It converts the status code to a string, then selects the counter with that label, such as `http_requests_processed{status_code="200"}`. Finally, it increments the counter with `Inc()`.
   4. Write the header in the response.
   5. Sends a simple response body.

    ```go
    func main() {
    	http.Handle("/metrics", promhttp.Handler())                             // 1
    	http.HandleFunc("/", func(w http.ResponseWriter, r *http.Request) {     // 2
    		time.Sleep(50 * time.Millisecond)                                   // 2.1
    		code := http.StatusOK                                               // 2.2
    		if time.Now().Unix()%2 == 0 {
    			code = http.StatusInternalServerError
    		}
    		requestsProcessed.WithLabelValues(fmt.Sprintf("%d", code)).Inc()    // 3
    		w.WriteHeader(code)                                                 // 4
    		fmt.Fprintf(w, "Request processed")                                 // 5
    	})

    	fmt.Println("Starting server on port 8080...")
    	http.ListenAndServe(":8080", nil)
    }
    ```

## OTel project

OpenTelemetry (OTel) is an open standard project under the Cloud Native Computing Foundation (CNCF) that provides a set of tools (standars, SDKs, APIs) for observability that lets you generate, collect, and export telemetry data from your application in a vendor-neutral way. It includes the following:

- [Traces](https://pkg.go.dev/go.opentelemetry.io/otel/trace): to follow the flow of requests through systems
- [Metrics](https://pkg.go.dev/go.opentelemetry.io/otel/metric): system behavior measurements
- [Context propagation](https://pkg.go.dev/go.opentelemetry.io/otel/propagation)

{{< admonition "Logging" warning >}}
The Logs SDK is still in development. Use uber/zap for logging until the SDK is complete.
{{< /admonition >}}

These SDKs support [multiple vendors](https://opentelemetry.io/ecosystem/vendors/) that so you can export your telemetry data for analysis.