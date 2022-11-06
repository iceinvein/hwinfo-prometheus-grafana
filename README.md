## Guide on setting up PC monitoring

The following tools were used
- [HWiNFO](https://www.hwinfo.com)
  - This tool captures data from various sensors in your host
- [PromDapter](https://github.com/kallex/PromDapter/releases)
  - This plugin for HWiNFO allows you to broadcast the sensor data for Prometheus to scrape  
- [Prometheus](https://prometheus.io/)
  - A time series data collection tool to collect sensor data
- [Grafana](https://grafana.com/get/?pg=graf&plcmt=hero-btn-1&tab=self-managed)
  - A visualisation tool to display collected data in various ways

### Steps
- Download and install HWiNFO and PromDapter to your target host
  - By default the PromDapter broadcasts metric on port `10445` so be sure to open that port for incoming requests
- Setting up prometheus server
  - Best to set this up on a NAS or dedicated server but if that's not an option you can install it on the target machine too
  - We'll be using docker to simplify installation.
    - Create a yaml file with the following in server. Replace <target ip/hostname> with your target details e.g. `10.0.0.34`
      - ```yml
        global:
          scrape_interval: 15s
          scrape_timeout: 10s
          evaluation_interval: 15s
        alerting:
          alertmanagers:
          - follow_redirects: true
            enable_http2: true
            scheme: http
            timeout: 10s
            api_version: v2
            static_configs:
            - targets: []
        scrape_configs:
        - job_name: prometheus
          honor_timestamps: true
          scrape_interval: 2s
          scrape_timeout: 10s
          metrics_path: /metrics
          scheme: http
          follow_redirects: true
          enable_http2: true
          static_configs:
          - targets:
            - <target ip/hostname>:10445
        ``` 
    - Start prometheus server with the following command (Note: Update the params for volume and port as required)
      - ```bash
        docker run \
        -p 9090:9090 \
        -v /host/path/to/prometheus.yml:/etc/prometheus/prometheus.yml \
        prom/prometheus
      ```
  - Setting up grafana server
    - We'll be using docker for installation simplicity
      - Start grafana server (Note: Update port as required)
        - ```
          docker run -d -p 3000:3000 --name grafana grafana/grafana-oss
          ```
  - Once the server has started. Go to grafana end point `http://<server>:3000`
    - Add data source
      - Prometheus
      - URL: `http://<server>:9090`
      - Save & test
    - Add Dashboard
      - Add a new Panel
      - Metric > Select metric
        - If everything is set correctly you'll find metric prefixed with `hwi_`
    - Create dashboard to your liking
