Here is your finalized `README.md` content in Markdown format:

````markdown
# 📊 Grafana + Prometheus + Loki Integration with Node.js

This guide helps you integrate **Grafana**, **Prometheus**, and **Loki** with a **Node.js** backend to monitor metrics and logs in real-time.

---

## ✅ Features Monitored

- 🔢 Total HTTP Requests
- ⏱️ Response Time per Route (Histogram)
- 📄 Logs streamed to Loki (via Winston)

---

## ⚙️ Step 1: Prometheus Setup

### 🔧 1. Create `prometheus.yml`

```yaml
global:
  scrape_interval: 4s

scrape_configs:
  - job_name: 'nodejs_app'
    static_configs:
      - targets: ['<NODEJS_SERVER_ADDRESS>']  # Example: ['localhost:8904']
````

> Replace `<NODEJS_SERVER_ADDRESS>` with your actual backend host and port.

---

### 🚀 2. Run Prometheus with Docker Compose

Create a `docker-compose.yml`:

```yaml
version: "3"

services:
  prometheus:
    image: prom/prometheus
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
```

Run the Prometheus server:

```bash
docker-compose up -d
```

Prometheus will be accessible at:
🔗 [http://localhost:9090](http://localhost:9090)

---

## 📈 Step 2: Grafana Setup

```bash
docker run -d -p 3000:3000 --name=grafana grafana/grafana-oss
```

Grafana will be accessible at:
🔗 [http://localhost:3000](http://localhost:3000)

> Default login: `admin / admin`

---

## 🪵 Step 3: Loki Setup (for Logging)

```bash
docker run -d --name=loki -p 3100:3100 grafana/loki
```

Loki will be accessible at:
🔗 [http://localhost:3100](http://localhost:3100)

---

## 🧑‍💻 Step 4: Node.js Backend Configuration

### 📦 Install Required Packages

```bash
npm install prom-client winston winston-loki response-time
```

---

### ✏️ Logging to Loki

```ts
import { createLogger } from "winston";
import LokiTransport from "winston-loki";

const logger = createLogger({
  transports: [
    new LokiTransport({
      host: "http://127.0.0.1:3100",
      labels: { appName: "loadster" },
    }),
  ],
});
```

---

### 📊 Prometheus Metrics Initialization

```ts
import client from "prom-client";
import responseTime from "response-time";

const collectDefaultMetrics = client.collectDefaultMetrics;
collectDefaultMetrics({ register: client.register });

const reqResTime = new client.Histogram({
  name: "http_express_req_res_time",
  help: "Duration of HTTP requests in ms",
  labelNames: ["method", "route", "status_code"],
  buckets: [1, 50, 100, 200, 400, 500, 1000, 2000],
});

const totalReqCounter = new client.Counter({
  name: "total_req",
  help: "Total number of HTTP requests",
});
```

---

### 🧩 Middleware Setup

```ts
app.use(
  responseTime((req: any, res: any, time: number) => {
    if (req.originalUrl === "/metrics") return;

    totalReqCounter.inc();

    reqResTime.labels({
      method: req.method,
      route: req.route?.path || req.originalUrl || req.url,
      status_code: res.statusCode.toString(),
    }).observe(time);
  })
);
```

---

### 📡 Metrics Endpoint

```ts
app.get("/metrics", async (_req, res) => {
  try {
    res.setHeader("Content-Type", client.register.contentType);
    const metrics = await client.register.metrics();
    res.send(metrics);
  } catch (err) {
    logger.error("Error generating Prometheus metrics", err);
    res.status(500).send("Error collecting metrics");
  }
});
```

---

## 📊 Step 5: Grafana Dashboard Setup

### ➕ Add Data Sources:

1. Go to [Grafana → Settings → Data Sources](http://localhost:3000/datasources)
2. Add:

   * **Prometheus**: `http://localhost:9090`
   * **Loki**: `http://localhost:3100`

---

### 📈 Create Dashboards:

* Use `http_express_req_res_time` for histogram-based performance graphs.
* Use `total_req` for counters.
* Use Loki logs in **Explore** with filters like `{appName="loadster"}`.

---



---

✅ Now your Node.js backend is fully integrated with Prometheus, Grafana, and Loki for complete observability.

```

```
