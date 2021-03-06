# telegraf-particle-webhook-plugin
Particle Webhook Plug-in for Telegraf

##Download the Plugin

```
export MY-PROJECT=[YOUR PROJECT DIRECTORY]
mkdir -p $MY-PROJECT/go
export GOPATH=$MY-PROJECT/go
export PATH=$PATH:$GOPATH/bin
cd $MY-PROJECT
go get github.com/DazWilkin/telegraf-particle-webhook-plugin
```

This should get [Telegraf](https://github.com/influxdata/telegraf) too.

All being well both of the following should list files:

```
ls go/src/github.com/DazWilkin/telegraf-particle-webhook-plugin/
ls go/src/github.com/influxdata/telegraf/
```

Telegraf needs to be configured to find this telegraf-particle-webhook-plugin. There are several changes that need be made.

###1. Edit 'webhooks.go'

In the directory:

    $MY-PROJECT/go/src/github.com/influxdata/telegraf/plugins/inputs/webhooks

Edit the file 'webhooks.go'

####1a. Add the following line to the imports.

    "github.com/DazWilkin/telegraf-particle-webhook-plugin"

Convention encourages the imports to be order alphabetically.

The result should look like this:

```
import (
	"fmt"
	"log"
	"net/http"
	"reflect"

	"github.com/DazWilkin/telegraf-particle-webhook-plugin"

	"github.com/gorilla/mux"
	"github.com/influxdata/telegraf"
	"github.com/influxdata/telegraf/plugins/inputs"

	"github.com/influxdata/telegraf/plugins/inputs/webhooks/github"
	"github.com/influxdata/telegraf/plugins/inputs/webhooks/rollbar"
)
```

####1b. Edit 'Webhooks' struct

Add the following line:

    Particle *particle.ParticleWebhook

The result should look like this:

```
type Webhooks struct {
	ServiceAddress string

	Github  *github.GithubWebhook
	Particle *particle.ParticleWebhook
	Rollbar *rollbar.RollbarWebhook
}
```

####1c. Edit the 'SampleConfig' function

Add the following lines:

```
[inputs.webhooks.particle]
    path = "/particle"
```

The result should look like this:

```
func (wb *Webhooks) SampleConfig() string {
	return `
  ## Address and port to host Webhook listener on
  service_address = ":1619"

  [inputs.webhooks.github]
    path = "/github"
		
  [inputs.webhooks.particle]
    path = "/particle"

  [inputs.webhooks.rollbar]
    path = "/rollbar"
 `
}
```

####1d. Save (and you may close) the file.

###2. Edit 'webhooks_test.go

In the directory:

    $MY-PROJECT/go/src/github.com/influxdata/telegraf/plugins/inputs/webhooks

Edit the file 'webhooks_test.go'

####2a. Add the following line to the imports.

    "github.com/DazWilkin/influxdata/telegraf/plugins/inputs/webhooks/particle"

The result should look like:

```
import (
	"reflect"
	"testing"

	"github.com/DazWilkin/influxdata/telegraf/plugins/inputs/webhooks/particle"

	"github.com/influxdata/telegraf/plugins/inputs/webhooks/github"
	"github.com/influxdata/telegraf/plugins/inputs/webhooks/rollbar"
)
```

####2b. Edit the 'TestAvailableWebhooks' function

Add the following lines between the block that starts wb.Github and the block that starts wb.Rollbar

```
wb.Particle = &particle.ParticleWebhook{Path: "/particle"}
	expected = append(expected, wb.Particle)
	if !reflect.DeepEqual(wb.AvailableWebhooks(), expected) {
		t.Errorf("expected to be %v.\nGot %v", expected, wb.AvailableWebhooks())
	}
```

The result should look like this:

```
func TestAvailableWebhooks(t *testing.T) {
	wb := NewWebhooks()
	expected := make([]Webhook, 0)
	if !reflect.DeepEqual(wb.AvailableWebhooks(), expected) {
		t.Errorf("expected to %v.\nGot %v", expected, wb.AvailableWebhooks())
	}

	wb.Github = &github.GithubWebhook{Path: "/github"}
	expected = append(expected, wb.Github)
	if !reflect.DeepEqual(wb.AvailableWebhooks(), expected) {
		t.Errorf("expected to be %v.\nGot %v", expected, wb.AvailableWebhooks())
	}

	wb.Rollbar = &rollbar.RollbarWebhook{Path: "/rollbar"}
	expected = append(expected, wb.Rollbar)
	if !reflect.DeepEqual(wb.AvailableWebhooks(), expected) {
		t.Errorf("expected to be %v.\nGot %v", expected, wb.AvailableWebhooks())
	}
}
```

####2c. Save (and you may close) the file.

###3. Add Gorilla Scheme to the godeps

The Plugin uses [Gorilla web toolkit](http://www.gorillatoolkit.org/) [package schema](http://www.gorillatoolkit.org/pkg/schema). This must be included in Telegraf's godeps

####3a. Edit 'Godeps'

In the directory:

    $MY-PROJECT/go/src/github.com/influxdata/telegraf

Edit the file 'Godeps'

####3b. Reference Gorilla schema

Add the following line:

    github.com/gorilla/schema 8aac656cd31cfb73a9dfd142494864fa0b84f723

The file is long so only the gorilla packages are referenced here but the result should look like this:

```
github.com/gorilla/context 1ea25387ff6f684839d82767c1733ff4d4d15d0a
github.com/gorilla/mux c9e326e2bdec29039a3761c07bece13133863e1e
github.com/gorilla/schema 8aac656cd31cfb73a9dfd142494864fa0b84f723
```


###4. Run 'make'

From the telegraf root directory, run 'make'

If not, already

    cd $MY-PROJECT/go/src/github.com/influxdata/telegraf
    make

You should see the dependencies restored and finally 'go install'

```
> Restoring $MY-PROEJCT/go/src/gopkg.in/fatih/pool.v2 to cba550ebf9bce999a02e963296d4bc7a486cb715
> Restoring $MY-PROEJCT/go/src/gopkg.in/mgo.v2 to d90005c5262a3463800497ea5a89aed5fe22c886
> Restoring $MY-PROEJCT/go/src/gopkg.in/yaml.v2 to a83829b6f1293c91addabc89d0571c246397bbf4
go install -ldflags "-X main.version=1.0.0-beta2-39-g300d9ad" ./...
```

###5 Create a configuration file

Before running Telegraf, we must create a default configuration file and include the telegraf-particle-webhook-plugin.

####5a. Create a configuration file

Run the following command to generate a default configuration file.

    telegraf -sample-config > telegraf.conf

####5b. Edit the configuration file

Find or scroll to the very end of the configuration file. You should see:

```
# # A Webhooks Event collector
# [[inputs.webhooks]]
#   ## Address and port to host Webhook listener on
#   service_address = ":1619"
#
#   [inputs.webhooks.github]
#     path = "/github"
#
#   [inputs.webhooks.particle]
#     path = "/particle"
#
#   [inputs.webhooks.rollbar]
#     path = "/rollbar"
```

Uncomment 

1. the section name '[[inputs.webhooks]];
1. the entry 'service_address'
1. the section [input.webhooks.particle]

Your file should look like this:

```
# # A Webhooks Event collector
[[inputs.webhooks]]
#   ## Address and port to host Webhook listener on
   service_address = ":1619"
#
#   [inputs.webhooks.github]
#     path = "/github"
#
 [inputs.webhooks.particle]
   path = "/particle"
#
#   [inputs.webhooks.rollbar]
#     path = "/rollbar"
```

####5c. Save (and you may close) the file.

###6 Run Telegraf

Telegraf should now be configured to with the telegraf-particle-webhook-plugin. Use the following command to start Telegraf

    telegraf -config telegraf.conf

You should see output similar to this:

```
2016/07/16 14:25:41 Starting Telegraf (version 1.0.0-beta2-39-g300d9ad)
2016/07/16 14:25:41 Loaded outputs: influxdb
2016/07/16 14:25:41 Loaded inputs: cpu mem system webhooks disk diskio kernel processes swap
2016/07/16 14:25:41 Tags enabled: host=skullcanyon
2016/07/16 14:25:41 Agent Config: Interval:10s, Debug:false, Quiet:false, Hostname:"skullcanyon", Flush Interval:10s 
2016/07/16 14:25:41 Started the webhooks service on :1619
2016/07/16 14:25:41 Started 'particle' on /particle
```

NB the webhooks service started and 'particle' (our Webhook) started on '/particle'

###7 Test the Plugin

If you have a [Photon](https://www.particle.io/prototype#photon) (or other Particle device) and the device is Internet connected and the port (1619) on the machine on which the Plugin is running is also Internet accessible, you will be able to test the Plugin by configuring one of Particle's "Integrations" using the Particle dashboard.

####7a. Publish events

If you have an existing example, feel free to use it. To help and for convenience, the following trivial example publishes random numbers every 10 seconds. Deploy this code to one of your Photon devices.

The Photon code is and can be deployed using [Particle Build IDE](https://build.particle.io/build)

```
void loop() {
    Particle.publish("randomnumber", String(random(100)), PRIVATE);
    delay(10000);
}
```

The [Dashboard](https://dashboard.particle.io/user/logs) should show something like:

[TBD]


####7b. Configure Integration

1. Visit the [Particle Dashboard](https://dashboard.particle.io/user/devices)
1. Login if necessary
1. Click [Integrations](https://dashboard.particle.io/user/integrations)
1. Click "New Integration"
1. Choose "Webhook"

And then complete the form using the IP address of the Plugin:

[TBD]

##Curl

Alternatively, you may cURL the endpoint:

```
curl \
--data "event=randomnumber&data=999&published_at=2016-07-16T23%3A59%3A59Z&coreid=123456789abcdef12345678" \
http://localhost:1619/particle
```

##Dockerize

1. Create a Docker Network
1. Edit the Telegraf configuration ('telegraf.conf') to reference the InfluxDB database
1. Build a container for Telegraf with the telegraf-particle-webhook-plugin
1. Run #1, InfluxDB and Grafana
1. Query InfluxDB 'telegraf' database
1. Add a Grafana graph display the measurements

Telegraf is configured to use InfluxDB but assumes InfluxDB is running on localhost. This will not be true when the servers are running in containers. Edit 'telegraf.conf', find the section '[[outputs.influxdb]]' and change the value of 'urls' to point to influxdb rather than localhost. The result should look like this:

```
[[outputs.influxdb]]
  ...
  urls = ["http://influxdb:8086"]
```

In order to easily connect Grafana, Telegraf to InfluxDB, we'll run the containers on the same Docker Network

    docker network create influxdata

Create a Dockerfile (proposed name: 'Dockerfile.ubuntu') in your project directory:

```
cd $MY-PROJECT
touch Dockerfile.ubuntu
```

Use the following content for the Dockerfile

```
from golang:wheezy

env root="src/github.com"
env telegraf="$root/influxdata/telegraf"

run apt-get -y update && apt-get -y upgrade

add ./go .
workdir $telegraf
run make prepare build-for-docker
expose 1619
entrypoint ["./telegraf","-config","telegraf.conf"]
```

Build the Docker image:

```
docker build \
--tag=telegraf:ubuntu \
--file=Dockerfile.ubuntu \
.
```

NB Don't forget the final "."

Now we'll run the 3 containers using the Docker Network

```
docker run -d \
--name=influxdb \
--net=influxdata \
--publish=8083:8083 \
--publish=8086:8086 \
influxdb

docker run -d \
--name=telegraf \
--net=influxdata \
--publish=1619:1619 \
telegraf:ubuntu

docker run -d \
--name=grafana  \
--net=influxdata \
--publish=3000:3000 \
grafana/grafana
```

If you're familiar with InfluxDB's UI, you should be able to access it on your host (localhost?) as

    http://localhost:8083

Alternatively, you can make queries directly against the database's query endpoint (:8086), e.g.

    http://localhost:8086/query?q=select+%22data%22+from+%22particle%22&db=telegraf

The InfluxDB 'telegraf' will only be created once measurements are received.

If you need to generate some test data you may use curl as explained above.

The Grafana UI is accessible on your host's port 3000:

    http://localhost:3000
    
Add a Data Source

- Name: InfluxDB
- Default: [Check]
- Url: http://localhost:8086
- Access: direct
- Database: telegraf
- User: root
- Password: root

Click "Save & Test" to confirm this is working

Create a Grafana dashboard, add a "Graph" and use a query:

    SELECT mean("data") FROM "particle" WHERE $timeFilter GROUP BY time($interval) fill(none)

The Grafana editor is very helpful and you can click e.g. "select measurement" for a dropdown of options (choose 'particle')

####Tidy-up

The following will stop-remove the containers and delete the network

```
for container in grafana telegraf influxdb
do
  docker stop $container | xargs docker rm
done
docker network rm influxdata
```
