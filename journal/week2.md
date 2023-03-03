# Week 2 — Distributed Tracing

Playing with Xray inside the back-end 

```
export AWS_REGION="ca-central-1"
gp env AWS_REGION="ca-central-1"
```

Add to the  inside back end flask `requirements.txt`

```
aws-xray-sdk
```

Add python dependencies using pip install 
```
pip install aws-xray-sdk
```

Then run this at the back end-flask folder

```
pip install -r requirements.txt
```

Add to `app.py`

```
from aws_xray_sdk.core import xray_recorder
from aws_xray_sdk.ext.flask.middleware import XRayMiddleware

xray_url = os.getenv("AWS_XRAY_URL")
xray_recorder.configure(service='Cruddur', dynamic_naming=xray_url)
XRayMiddleware(app, xray_recorder)
```

Setup AWS X-ray resources 

Add `aws/json/xray.json`

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

```
FLASK_ADDRESS="https://4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}"
aws xray create-group \
   --group-name "Cruddur" \
   --filter-expression "service(\"$FLASK_ADDRESS\") {fault OR error}"
```

Type at the Terminal 

```
aws xray create-sampling-rule --cli-input-json file://aws/json/xray.json
```
and you will get this inside your AWS console 

![xray_sampling](assets/xray_sampling_group.png)

[Install X-ray Daemon](https://docs.aws.amazon.com/xray/latest/devguide/xray-daemon.html)

[Github aws-xray-daemon](https://github.com/aws/aws-xray-daemon) 

[X-Ray Docker Compose example](https://github.com/marjamis/xray/blob/master/docker-compose.yml)

```
 wget https://s3.us-east-2.amazonaws.com/aws-xray-assets.us-east-2/xray-daemon/aws-xray-daemon-3.x.deb
 sudo dpkg -i **.deb
```

add daemon service to Docker-compose

```
  xray-daemon:
    image: "amazon/aws-xray-daemon"
    environment:
      AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
      AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"
      AWS_REGION: "ca-central-1"
    command:
      - "xray -o -b xray-daemon:2000"
    ports:
      - 2000:2000/udp
```
Note: When you run the docker-compose up you will see the log for warning for this 2 env variables. you can export it manually or put it in your awscli folder. this sometimes the issue for credential error when gitpod trying to access the aws xray. 

![aws_access_key_issue](assets/gitpod_aws_access_key_issue_xray.png)


```
AWS_ACCESS_KEY_ID: "${AWS_ACCESS_KEY_ID}"
AWS_SECRET_ACCESS_KEY: "${AWS_SECRET_ACCESS_KEY}"

```

We need to add these two env vars to our backend-flask in our `docker-compose.yml` file
```
AWS_XRAY_URL: "*4567-${GITPOD_WORKSPACE_ID}.${GITPOD_WORKSPACE_CLUSTER_HOST}*"
AWS_XRAY_DAEMON_ADDRESS: "xray-daemon:2000"
```

check service data for last 10 minutes


```
EPOCH=$(date +%s)
aws xray get-service-graph --start-time $(($EPOCH-600)) --end-time $EPOCH
```

When everything is good you will see this 

![xray.png](assets/xray.png)

also in the cloud watch dashboard 

![dashboard1.png](assets/xray_dashboard1.png)
![dashboard2.png](assets/xray_dashboard2.png)
![dashboard3.png](assets/xray_dashboard3.png)

and in the xray old console dashboard 

![xray_wordking_now.png](assets/xray_working_now.png)

a great place to troubleshoot and debug if you having issue is the docker-compose container 

![xray_log.png](assets/xray_log.png)

