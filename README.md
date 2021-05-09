# Raspberry-Pi-Google-Cloud-Temperature-Monitor
## Introduction
This document describes the steps required to setup a Raspberry Pi temperature and humidity monitor IoT device using Google Cloud as the backbone of the data processing and handling. 

The first section goes into details on the initial setup of Google Cloud while the next section describes the steps on how to setup the Raspberry Pi device. The last section then shows how data can be collected and aggregated in Google Cloud. 

This is pureley a demonstration of how to use statitical information processing from an IoT device with Google Cloud and is not meant for any form of critical weather analysis.

## Required Hardware
* One Raspberry Pi 3: https://www.amazon.com/Raspberry-Pi-Desktop-Starter-White/dp/B01CI58722
* One breadboard: https://www.amazon.com/Solderless-Breadboard-Circuit-Circboard-Prototyping/dp/B01DDI54II/
* One DHT11 sensor: https://www.amazon.com/HiLetgo-Temperature-Humidity-Arduino-* Raspberry/dp/B01DKC2GQ0
* Three male-to-female jumper cables: https://www.amazon.com/RGBZONE-120pcs-Multicolored-Dupont-Breadboard/dp/B01M1IEUAF/

# Setting up a project
To get started with Google IoT Core, a Google account is required in order to access Google Cloud. If you do not have a Google account, you can create one by navigating to this URL: https://accounts.google.com/SignUp?hl=en. Once you have created your account, you can login and navigate to Google Cloud Console: https://console.cloud.google.com. 

As of 5/9/2021, Google Cloud Platform has a free trial for 12 months with $300 to try out the service before paying for any of their services. None of the services utilized in this demonstration require you to pay any amount of money unless you excessively send and process data on Google's service.

1. Once you have signed up, you can start by creating a new project. From the top menu bar, select the Select a Project dropdown and click on the plus icon to create a new project. You can fill in the details as illustrated in the following screenshot:

![New Project](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/0cf2051c-3814-46d6-a206-2d72bd506fea.png)

2. Click on the Create button. Once the project is created, navigate to the Project and you should land on the Home page.
# Enabling APIs
1. From the menu on the left-hand side, select APIs & Services | Library as shown in the following screenshot:

![Library](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/cd04f6e8-f9a3-40b5-a42f-1966c0464947.png)

2. On the following screen, search for pubsub and select the Pub/Sub API from the results and you should land on a page similar to the following:

![Pub Sub](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/6823e4dd-1212-4add-81bb-604354f9be9b.png)

3. Click on the ENABLE button and you should now be able to use these APIs in your project.
4. Next, you need to enable the real-time API; search for realtime and you should find something similar to the following:

![RealTime](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/542f3b58-b597-4d26-b2f5-f2ea75c1b1bf.png)

5. Click on the ENABLE & button.
# Enabling device registry and devices
The following steps should be used for enabling device registry and devices:
1. From the left-hand side menu, select IoT Core and you should land on the IoT Core home page:

![IoTCore](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/f2426d1e-f235-475d-ab9c-45b86445219f.png)

Instead of the previous screen, if you see a screen to enable APIs, please enable the required APIs from here.
2. Click on the & Create device registry button. On the Create device registry screen, fill the details as shown in the following table:

| Field | Value |
| --- | --- |
| Registry ID | Pi4-DHT11-Nodes |
| Cloud region | us-central1 |
| Protocol | MQTT, HTTP |
| Default telemetry topic | device-events |
| Default state topic | dht11 |
3. After completing all the details, our form should look like the following:

![Device Registry](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/c2cbf9cd-8765-4df0-9791-e8e3954f7c81.png)

4. Click on the Create button and a new device registry will be created.
5. From the Pi3-DHT11-Nodes registry page, click on the Add device button and set the Device ID as Pi3-DHT11-Node or any other suitable name.
6. Leave everything as the defaults and make sure the Device communication is set to Allowed and create a new device.
7. On the device page, you should see a warning as highlighted in the following screenshot:

![Device Details](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/d835136a-e769-488c-a994-1ecabf7736eb.png)

8. Now, in order to secure the connection between the Raspberry Pi and Google Cloud, you must create a public and private SSL Key to authenitcate and establish the connection. To generate a public/private key pair, you will need an OpenSSL command line available. You can download and set up OpenSSL from here: https://www.openssl.org/source/.
9. You can use the following command to generate a certificate pair at the default location on your machine:
```
openssl req -x509 -newkey rsa:2048 -keyout rsa_private.pem -nodes -out rsa_cert.pem -subj "/CN=unused"
```
10. If the command ran successfully, you should see an output as shown here:

![openSSL](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/b2bb3941-646a-484a-9cf0-76f5f5e75c8e.png)

Do not share these certificates anywhere; anyone with these certificates can connect to Google IoT Core as a device and start publishing data.
11. Now, once the certificates are created, you will attach them to the device you have created in IoT Core.
12. Head back to the device page of the Google IoT Core service and under Authentication click on Add public key. On the following screen, fill it in as illustrated:

![Auth Key](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/fa205710-d61b-4b44-98f9-57893cd3705c.png)

13. The public key value is the contents of rsa_cert.pem that you generated earlier. Click on the ADD button.
Now that the public key has been successfully added, you can connect to the cloud using the private key.
# Setting up Raspberry Pi 3 with DHT11 node
Now that you have our device set up in Google IoT Core, you are going to complete the remaining operation on Raspberry Pi 3 to send data.

If you are new to the world of Raspberry Pi GPIO’s interfacing, take a look at this Raspberry Pi GPIO Tutorial: The Basics Explained on YouTube: https://www.youtube.com/watch?v=6PuK9fh3aL8.
The following steps are to be used for the setup process:
1. Connect the DHT11 sensor to Raspberry Pi 3 as shown in the following diagram:

![Pi Wires](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/43b69442-103a-4fbd-9b43-9717e06c9187.png)

3. Next, power up Raspberry Pi 4 and log in to it. On the desktop, create a new folder named Google-IoT-Device. Open a new Terminal and cd into this folder.
## Setting up Node.js
Refer to the following steps to install Node.js:
1. Open a new Terminal and run the following commands:
```
$ sudo apt update
```
```
$ sudo apt full-upgrade
```
2. This will upgrade all the packages that need upgrades. Next, you will install the latest version of Node.js. you will be using the Node 7.x version:
```
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
```
```
$ sudo apt install nodejs
```
3. This will take a moment to install, and once your installation is done, you should be able to run the following commands to see the version of Node.js and npm:
```
$ node -v
```
```
$ npm -v
```
## Developing the Node.js device app
Now, you will set up the app and write the required code:
1. From the Terminal, once you are inside the Google-IoT-Device folder, run the following command:
> $ npm init -y
2. Next, you will install jsonwebtoken (https://www.npmjs.com/package/jsonwebtoken) and mqtt (https://www.npmjs.com/package/mqtt) from npm. Execute the following command:
> $ npm install jsonwebtoken mqtt--save
3. Next, you will install rpi-dht-sensor (https://www.npmjs.com/package/rpi-dht-sensor) from npm. This module will help in reading the DHT11 temperature and humidity values:
> $ npm install rpi-dht-sensor --save
4. Your final package.json file should look similar to the following code snippet:
```
{ 
 "name": "Google-IoT-Device", 
 "version": "1.0.0", 
 "description": "", 
 "main": "index.js", 
 "scripts": { 
  "test": "echo "Error: no test specified" && exit 1" 
 }, 
 "keywords": [], 
 "author": "", 
 "license": "ISC", 
 "dependencies": { 
  "jsonwebtoken": "^8.1.1", 
  "mqtt": "^2.15.3", 
  "rpi-dht-sensor": "^0.1.1" 
 } 
}
```
5. Now that you have the required dependencies installed, let’s continue. Create a new file named index.js at the root of the Google-IoT-Device folder. Next, create a folder named certs at the root of the Google-IoT-Device folder and move the two certificates you created using OpenSSL there.
6. Your final folder structure should look something like this:

![Pi tree](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/cb7b7d25-2768-4aa4-bb78-85168c2fba1b.png)

7. Open index.js in any text editor and update it as shown here:
```
var fs = require('fs'); 
var jwt = require('jsonwebtoken'); 
var mqtt = require('mqtt'); 
var rpiDhtSensor = require('rpi-dht-sensor'); 
 
var dht = new rpiDhtSensor.DHT11(2); // `2` => GPIO2 
 
var projectId = 'pi-iot-project'; 
var cloudRegion = 'us-central1'; 
var registryId = 'Pi3-DHT11-Nodes'; 
var deviceId = 'Pi3-DHT11-Node'; 
 
var mqttHost = 'mqtt.googleapis.com'; 
var mqttPort = 8883; 
var privateKeyFile = '../certs/rsa_private.pem'; 
var algorithm = 'RS256'; 
var messageType = 'state'; // or event 
 
var mqttClientId = 'projects/' + projectId + '/locations/' + cloudRegion + '/registries/' + registryId + '/devices/' + deviceId; 
var mqttTopic = '/devices/' + deviceId + '/' + messageType; 
 
var connectionArgs = { 
  host: mqttHost, 
  port: mqttPort, 
  clientId: mqttClientId, 
  username: 'unused', 
  password: createJwt(projectId, privateKeyFile, algorithm), 
  protocol: 'mqtts', 
  secureProtocol: 'TLSv1_2_method' 
}; 
 
console.log('connecting...'); 
var client = mqtt.connect(connectionArgs); 
 
// Subscribe to the /devices/{device-id}/config topic to receive config updates. 
client.subscribe('/devices/' + deviceId + '/config'); 
 
client.on('connect', function(success) { 
  if (success) { 
    console.log('Client connected...'); 
    sendData(); 
  } else { 
    console.log('Client not connected...'); 
  } 
}); 
 
client.on('close', function() { 
  console.log('close'); 
}); 
 
client.on('error', function(err) { 
  console.log('error', err); 
}); 
 
client.on('message', function(topic, message, packet) { 
  console.log(topic, 'message received: ', Buffer.from(message, 'base64').toString('ascii')); 
}); 
 
function createJwt(projectId, privateKeyFile, algorithm) { 
  var token = { 
    'iat': parseInt(Date.now() / 1000), 
    'exp': parseInt(Date.now() / 1000) + 86400 * 60, // 1 day 
    'aud': projectId 
  }; 
  var privateKey = fs.readFileSync(privateKeyFile); 
  return jwt.sign(token, privateKey, { 
    algorithm: algorithm 
  }); 
} 
 
function fetchData() { 
  var readout = dht.read(); 
  var temp = readout.temperature.toFixed(2); 
  var humd = readout.humidity.toFixed(2); 
 
  return { 
    'temp': temp, 
    'humd': humd, 
    'time': new Date().toISOString().slice(0, 19).replace('T', ' ') // https://stackoverflow.com/a/11150727/1015046 
  }; 
} 
 
function sendData() { 
  var payload = fetchData(); 
 
  payload = JSON.stringify(payload); 
  console.log(mqttTopic, ': Publishing message:', payload); 
  client.publish(mqttTopic, payload, { qos: 1 }); 
 
  console.log('Transmitting in 30 seconds'); 
  setTimeout(sendData, 30000); 
}
```
In the previous code, you first define the projectId, cloudRegion, registryId, and deviceId based on what you have created. Next, you build the connectionArgs object, using which you are going to connect to Google IoT Core using MQTT-SN. Do note that the password property is a JSON Web Token (JWT), based on the projectId and privateKeyFile algorithm.

The token that is created by this function is valid only for one day. After one day, the cloud will refuse connection to this device if the same token is used.
The username value is the Common Name (CN) of the certificate you have created, which is unused.

Using mqtt.connect(), you are going to connect to the Google IoT Core. And you are subscribing to the device config topic, which can be used to send device configurations when connected.

Once the connection is successful, you callsendData() every 30 seconds to send data to the state topic.

Save the previous file and run the following command:
```
$ sudo node index.js
```
And you should see something like this:

![Pi data](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/e6a11f42-5813-43ab-a4ad-1c4f865193ac.png)

As you can see from the previous Terminal logs, the device first gets connected then starts transmitting the temperature and humidity along with time. You are sending time as well, so you can save it in the BigQuery table and then build a time series chart quite easily.

If you head back to the Device page of Google IoT Core and navigate to the Configuration & state history tab, you should see the data that you are sending to the state topic here:

![Pi node](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/9c49aab3-3514-4322-8705-7b603e30d539.png)

## Setting up subscriptions
The data from the device is being sent to Google IoT Core using the state topic. If you recall, you have named that topic dht11. Now, you are going to create a subscription for this topic:
1. From the menu on the left side, select Pub/Sub | Topics. Now, click on New subscription for the dht11 topic, as shown in the following screenshot:

![Pi sub](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/97e5f3f6-1e20-40dc-bcb4-4d4a2be8365e.png)

2. Create a new subscription by setting up the options selected in this screenshot:

![Pi create sub](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/02ee3473-c0dc-4d54-ab98-26166e7aa604.png)

# Building a dashboard
Now that you have seen how a client can read the data from your device on demand, you will move on to building a dashboard, where you display data in real time.

For this, you are going to use Google Cloud Functions, Google BigQuery, and Google Data Studio.
## Google Cloud Functions
Cloud Functions are solution for serverless services. Cloud Functions is a lightweight solution for creating standalone and single-purpose functions that respond to cloud events.

You can read more about Google Cloud Functions at https://cloud.google.com/functions/.
## Google Bigquery
Google BigQuery is an enterprise data warehouse that solves this problem by enabling super-fast SQL queries using the processing power of Google’s infrastructure.

You can read more about Google BigQuery at https://cloud.google.com/bigquery/.
## Setting up BigQuery
The first thing you are going to do is set up BigQuery:
1. From the side menu of the Google Cloud Platform Console, your project page, click on the BigQuery URL and you should be taken to the Google BigQuery home page. Select Create new dataset, as shown in the following screenshot:

![BigQuery](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/c9bc2a73-e3f8-47ac-b41e-710db363e201.png)

2. Create a new dataset with the values illustrated in the following screenshot:

![dataset](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/d5a910e3-664f-4079-a2b7-4b11d918b76d.png)

3. Once the dataset is created, click on the plus sign next to the dataset and create an empty table. You are going to name the table dht11_data and you are going have three fields in it, as shown here:

![big compose](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/9664c3d5-8440-4978-af54-9083b47474d2.png)

4. Click on the Create Table button to create the table.
Now that you have your table ready, you will write a cloud function to insert the incoming data from Pub/Sub into this table.
## Setting up Google Cloud Function
Now, you are going to set up a cloud function that will be triggered by the incoming data:
1. From the Google Cloud Console’s left-hand-side menu, select Cloud Functions under Compute. Once you land on the Google Cloud Functions homepage, you will be asked to enable the cloud functions API. Click on Enable API
2. Once the API is enabled, you will be on the Create function page. Fill in the form as shown here:

![cloud functions](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/3dd97d36-b389-4d22-868e-3a0f90e175ad.png)

The Trigger is set to Cloud Pub/Sub topic and you have selected dht11 as the Topic.
3. Under the Source code section; make sure you are in the index.js tab and update it as shown here:
```
var BigQuery = require('@google-cloud/bigquery'); 
var projectId = 'pi-iot-project'; 
 
var bigquery = new BigQuery({ 
  projectId: projectId, 
}); 
 
var datasetName = 'pi3_dht11_dataset'; 
var tableName = 'dht11_data'; 
 
exports.pubsubToBQ = function(event, callback) { 
  var msg = event.data; 
  var data = JSON.parse(Buffer.from(msg.data, 'base64').toString()); 
  // console.log(data); 
  bigquery 
    .dataset(datasetName) 
    .table(tableName) 
    .insert(data) 
    .then(function() { 
      console.log('Inserted rows'); 
      callback(); // task done 
    }) 
    .catch(function(err) { 
      if (err && err.name === 'PartialFailureError') { 
        if (err.errors && err.errors.length > 0) { 
          console.log('Insert errors:'); 
          err.errors.forEach(function(err) { 
            console.error(err); 
          }); 
        } 
      } else { 
        console.error('ERROR:', err); 
      } 
 
      callback(); // task done 
    }); 
};
```
4. In the previous code, you were using the BigQuery Node.js module to insert data into your BigQuery table. Update projectId, datasetName, and tableName as applicable in the code.
5. Finally, for the Function to execute field, enter pubsubToBQ. pubsubToBQ is the name of the function that has your logic and this function will be called when the data event occurs.
6. Click on the Create button and your function should be deployed in a minute.
## Running the device
Now that the entire setup is done, you will start pumping data into BigQuery:
1. Head back to Raspberry Pi 4 which was sending the DHT11 temperature and humidity data, and run the application. You should see the data being published to the state topic:

![Pi code run](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/7459f5ad-5712-4951-a9f6-9bc0f0b50c1a.png)

2. Now, if you head back to the Cloud Functions page, you should see the requests coming into the cloud function:

![trigger](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/7e0877fe-d906-4864-aa10-4a7558d52ef9.png)

3. You can click on VIEW LOGS to view the logs of each function execution:

![Logs](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/e588c941-6d04-4cff-b64c-31bbe8e6c010.png)

4. Now, head over to your table in BigQuery and click on the RUN QUERY button; run the query as shown in the following screenshot:

![New Query](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/fac45e3f-c9c1-41da-9d18-b9fdc75d15ed.png)

5. Now, all the data that was generated by the DHT11 sensor is timestamped and stored in BigQuery.
6. You can use the Save to Google Sheets button to save this data to Google Sheets and analyze the data there or plot graphs, as shown here:

![sheet data](https://packt-type-cloud.s3.amazonaws.com/uploads/sites/2518/2018/05/7b6f5f11-6ea1-4421-a66f-e54d0b697eef.png)

## Summary
This document showed how to setup a basic temperature and humiditiy IoT tracking service using Google Cloud and a Raspberry Pi 3 device. These instructions are relativily flexible with the device you use (Newer Raspberry Pi B models or Raspberry Pi zero nodes) but any changes to Google Cloud may require minor changes to the functions JSON code or BigQuery data collector. Further information about setting up other Google Cloud IoT projects with Raspberry Pi devices can be found in the following links:
