<img src="img/logo.svg" width="100" />

Synthetic Monitoring Agent
==========================
This is the 'worker' for Grafana's [Synthetic Monitoring application](https://github.com/grafana/synthetic-monitoring-app). The agent provides probe functionality and executes network [checks](https://github.com/grafana/synthetic-monitoring-app/blob/master/README.md#check-types) for monitoring remote targets. 

Please [install](https://grafana.com/grafana/plugins/grafana-synthetic-monitoring-app/installation) Synthetic Monitoring 
in your Grafana Cloud or local Grafana instance before setting up your own private probe. You may need to generate a [new API key](https://grafana.com/profile/api-keys) to initialize the app.


Probes
------
Probes run [checks](https://github.com/grafana/synthetic-monitoring-app/blob/master/README.md#check-types) from 
distributed locations around the world and send the resulting metrics and events directly to 
[Grafana Cloud](https://grafana.com/products/cloud/) Prometheus and Loki services. 

You can select 1 or more **public** probes to run checks from or [run your own **private** probes](https://grafana.com/docs/grafana-cloud/synthetic-monitoring/private-probes/)
from any environment you choose.


To run your own probe
---------------------
![Add Probe](img/screenshot-probes.png)
### Add a new probe in your Grafana instance
* Navigate to **Synthetic Monitoring -> Probes**.
* Click **New**
* Enter a **Probe Name**, **Latitude**, **Longitude**, and **Region**.
* Optionally enter up to 3 custom labels to identify your probe.
* Click **Save**
* Copy the "Probe Authentication Token" and save for installing the agent.

### Install the agent on Debian based systems

* Add package repo GPG key

`wget -q -O - https://packages-sm.grafana.com/gpg.key | sudo apt-key add -`

* Add Debian package repo

`sudo add-apt-repository "deb https://packages-sm.grafana.com/deb stable main"`

* Install synthetic-monitoring-agent package


`sudo apt-get install synthetic-monitoring-agent`

* Edit `/etc/synthetic-monitoring/synthetic-monitoring-agent.conf` and add the token retrieved from Grafana

```
# Enter API token retrieved from grafana.com here
API_TOKEN='YOUR TOKEN HERE'
```

* Restart the agent

`sudo service synthetic-monitoring-agent restart`

Once the service is running, you will be able to select your new probe exactly the same as any public probe. You will need to manually add the new probe to any previously created checks.

#### Deploy it using Docker
We publish a docker image on [docker hub](https://hub.docker.com/r/grafana/synthetic-monitoring-agent)

Steps:
- Get an Authentication Token for your private probe, [see here for the steps](https://grafana.com/docs/grafana-cloud/synthetic-monitoring/private-probes/)

```bash
# pull image
docker pull grafana/synthetic-monitoring-agent:latest
# export configs
# replace YOUR_TOKEN_HERE with your Authentication Token
export API_TOKEN=YOUR_TOKEN_HERE
export API_SERVER="synthetic-monitoring-grpc.grafana.net:443"
# run
docker run grafana/synthetic-monitoring-agent --api-server-address=${API_SERVER} --api-token=${API_TOKEN} --verbose=true
```

Now you should have the agent reporting as private probe, and running checks (if you have created some) in the logs.

#### Deploy it using Kubernetes
See [examples/kubernetes](./examples/kubernetes) for the documentation and example yaml files

Architecture
------------
The agent obtains configuration information from the synthetic-monitoring-api and uses it to build a configuration for [blackbox_exporter](https://github.com/prometheus/blackbox_exporter), which is how checks are executed.

![agent process][process]

[process]: https://www.planttext.com/api/plantuml/svg/dLHDRy8m3BtdLqGz3wHTEKo88KsxJVi78VLA14swn47ityzE0ZHLcIQua8zdl-Tdf-k0ocFiZqAq2jLE1P3DTjE8WOwDDeEoA9lmOt4Fj5_qpXfqtjXkeGRJI1Ka_Sl_m3kmc0DuLKUGY0xoRLxMruDtFL3A61BajgrXHtV8adWXHEAHYvUaS2MrinOqIdS2Bzy-FrwdW02svTmxaCP-ETyhDCuAfT6S54BHpLWAsMuemeDsliG83nYzBGdUjwA5IUIKpyDtX81Ixq4VP40Fgh-apz2YAGEUnNAv_EFUYiwxE53nRf0alnoVXQJVbJhRotQaMpY3ZgdCXBe8BatWir8M6UwDfkOHtz5r8TsDIXn5P1dEQaZRYhwk_DP8EkeTPkDlKJErpkBci_CKF9oNpchZHbfNSeXXVx6aXYNI0hZwn8shXHPgD3suY8BPKdkdCzAQKERsxk2DCRCv7XxyUuIygJHLJbuVWB3iQ69DWAVyBjc8e7fWe8OG-C7kW6Y1RP0S9CIQblHL-WK0
[PlantUML]: https://www.planttext.com/?text=dLHDRy8m3BtdLqGz3wHTEKo88KsxJVi78VLA14swn47ityzE0ZHLcIQua8zdl-Tdf-k0ocFiZqAq2jLE1P3DTjE8WOwDDeEoA9lmOt4Fj5_qpXfqtjXkeGRJI1Ka_Sl_m3kmc0DuLKUGY0xoRLxMruDtFL3A61BajgrXHtV8adWXHEAHYvUaS2MrinOqIdS2Bzy-FrwdW02svTmxaCP-ETyhDCuAfT6S54BHpLWAsMuemeDsliG83nYzBGdUjwA5IUIKpyDtX81Ixq4VP40Fgh-apz2YAGEUnNAv_EFUYiwxE53nRf0alnoVXQJVbJhRotQaMpY3ZgdCXBe8BatWir8M6UwDfkOHtz5r8TsDIXn5P1dEQaZRYhwk_DP8EkeTPkDlKJErpkBci_CKF9oNpchZHbfNSeXXVx6aXYNI0hZwn8shXHPgD3suY8BPKdkdCzAQKERsxk2DCRCv7XxyUuIygJHLJbuVWB3iQ69DWAVyBjc8e7fWe8OG-C7kW6Y1RP0S9CIQblHL-WK0
