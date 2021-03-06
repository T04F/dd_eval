Your answers to the questions go here.

# Solutions Engineer - Jerome Klodzinski - jp.klodzinski (at) gmail (dot) com - Company "Datadog Recruiting Candidate"

# Level Zero - Prerequisites
Setup is a nested environment (based on non-free Vagrant Service-Provider for VMware Fusion)
    - MacOS HighSierra 10.13.6
        - Fusion 10.1.3
            - Ubuntu 18.04.1 LTS
                - Vagrant 2.0.2 with VirtualBox 5.2.10 
                    - precise64 with datadog agent 6.4.2
# Level One - Collecting Metrics
1) Tags got added into the dadadog.yaml see:

http://tmp.gdwb.de/SC01.png

Second screenshot shows the attached tags to the system in Host Map

http://tmp.gdwb.de/SC04.png

Additional the metric view on Host Map for the app apache:

http://tmp.gdwb.de/SC05.png

Question: Tags of Apps are not inherited towards agent tags. How to filter Host by app related tags inside host map?
Like for my "app-infra:apache2" inside of app "apache"? Apache integration got installed


2) Installed and configured mongodb on host and integration got installed in datadog
http://tmp.gdwb.de/SC08.png

See the installed integration:
http://tmp.gdwb.de/SC10.png

See the datadog-agent status output related to mongodb
http://tmp.gdwb.de/SC06.png

Site note in datadog.yaml the log collection got enabled and for both apps apache and mongodb configured.
http://tmp.gdwb.de/SC07.png


3) To creat a custom agent script you need to add to files into /etc/datadog.
In ../conf.d/ a my_metic.yaml which looks like
----
init_config:

instances:
    [{}]
----

In ../check.d/ a my_metric.py which looks like:
----
from random import randint
from checks import AgentCheck
class my_metric(AgentCheck):
    def check(self, instance):
        self.gauge('my_metric', randint(0,1000))
----
See: http://tmp.gdwb.de/SC09.png

4) To adjust the interval you first need to know the version of the agent, the responsible yaml in conf.d need the setting.
Since v6 the option "min_collection_interval : 45" is only possible on instance level with a v5 agent you could even set it globally.



5)Bonus Question: For sure I guess the collection period can get adjusted globally within datadog.yaml. 


# Level Two - Visualizing Data
First of all you have to create new application key to get the required full access to API for named user.
http://tmp.gdwb.de/SC13.png

Based on my script I did had a "parsing error" when the graph for anomaly was integrated. Honestly, didn't figured out what is the cause.
Commenting out this graph and the api called dashboard creation went well.
----
api_key=adf64ae91539d6f3260aed9f3db50b47
app_key=34ad9f0b15824c83b53b5445f691bc756bd5b672

curl  -X POST -H "Content-type: application/json" \
-d '{
        "title" : "JPK Timeboard",
        "description" : "JPK timeboard created by API.",
        "template_variables" : 
		[{
                "name": "host1",
                "prefix": "host",
                "default": "precise64"
        	}],
        "graphs" : 
		[{
                "title": "my_metric for host:precise64",
                "definition": 
			{
                        "events": [],
                        "requests": 
				[
            			{"q": "my_metric{host:precise64}"}
                        	],
                        "viz": "timeseries"
                	}
        	},
#                {
#                "title": "Anomaly for MongoDB",
#                "definition": 
#			{
#                        "events": [],
#                        "requests": 
#				[
#				{"q": "avg(last_4h):anomalies(avg:mongodb.backgroundflushing.last_ms{host:precise64}, 'basic', 3, direction='both', alert_window='last_15m', interval=60, count_default_zero='true') >= 1" }
#                        	],
#                        "viz": "timeseries"
#                 	}
#        	},
               {
                "title": "my_metric SUM-UP",
                "definition": 
			{
                        "events": [],
                        "requests": 
				[
                                {"q": "my_metric{host:precise64}.rollup(sum,3600)"}
                        	],
                        "viz": "query_value"
                	}
        	}]
}' \
"https://api.datadoghq.com/api/v1/dash?api_key=${api_key}&application_key=${app_key}"
----
I guess it is some minor typo but I didn't found the cause. Screenshot attached
http://tmp.gdwb.de/SC11.png

Thus current deshboard without anomaly graph (which of cause could get added through GUI, but I want to stay "clean")
http://tmp.gdwb.de/SC14.png

NExt step you can see the 5min graph in "fullscreen"
http://tmp.gdwb.de/SC15.png

And afterwards the 5min timeframe while taking a snapshot with @notation and the related event.
http://tmp.gdwb.de/SC16.png
http://tmp.gdwb.de/SC17.png

Bonus Question:
Based on the selected algorithm the anomaly graph is learning of historic behavior if an upcoming value is inside historic normal behavior or not.
One example for this would be a high load on AD service every day at 8-9 am based on shift change would get marked as normal.
A sudden high load on AD service at 2pm would be a anomaly.

# Level Three - Monitoring Data
1) Created a metric monitor to check the value of my_metric
http://tmp.gdwb.de/SC22.png

Additional you can see the notification for initial creation of monitor and for the alarm (or more correct for the warning as it looks like I'm not getting an alarm:
http://tmp.gdwb.de/SC23.png
http://tmp.gdwb.de/SC21.png

Bonus Question:
See the following screenshots of the maintenance windows created for this monitor:

http://tmp.gdwb.de/SC18.png
http://tmp.gdwb.de/SC19.png

# Level Four - Collecting APM Data
See the APM view and the Dashboard with an APM and an host graph
http://tmp.gdwb.de/SC24.png
http://tmp.gdwb.de/SC25.png
As the goal was to just have an app running and I had some issues with pip on precise related to https not supported and no newer version direct awailable, I set up a second vagrant VM with Xenial64. There the following flask app got started:
----
from flask import Flask, Response


app = Flask(__name__)

@app.route("/")
def index():
    return Response("It works!"), 200

if __name__ == "__main__":
    app.run(debug=True)
----

Bonus Question:
A Service is the sum of all process mandatory to provide the output/the data/files etc also called the "resources" towards the user of the app.

# Level Five - Final Question
There are two topics coming straight in my mind. First of all would be smarthome. Based on different smarthome products
like www.hom.ee collecting all sensor output/logs etc. And combining this with anomaly behavior to determin if this is working as expect
or if there are misbehaviors...might even as an smart way of alarm system
Second idea is using datadog for the telemetry of racing series like DTM, Formula E or Formula One. 
