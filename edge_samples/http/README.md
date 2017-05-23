# Application level protocols: HTTP
This example shows an application which runs on Raspberry Pi and sends data from DHT-12 Temparature and Humidity Sensor module.

## Prepare hardware components
* Raspberry Pi 3 (Model B)
* Power Adapter (2A/5V)
* MicroSD Card (8GB+) and SD Adapter
* DHT-12 Temparature and Humidity Sensor module
* Breadboard and a set of dupont cables
* Ethernet cable for wired network connection

## Assemble the device
<img src="./_images/pinout.png" height="400">

## Prepare SD card
* Burn Raspbian image
* Create folder `/home/pi/sensor`
* Create file `/home/pi/sensor/package.json` with the following contents:
  ```
  {
    "name": "dht_12",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC",
    "dependencies": {
      "request": "^2.81.0",
      "rpi-dht-sensor": "^0.1.1"
    }
  }
  ```
* Create file `/home/pi/sensor/index.js` with the following contents:
  ```
  var rpiDhtSensor = require('rpi-dht-sensor');
  var request = require('request');

  var receiver = 'http://REMOTE-SERVER-ADDRESS.com:8080';
  var dht = new rpiDhtSensor.DHT11(2);

  function read () {
    var readout = dht.read();
    var data = {
      temperature: readout.temperature.toFixed(2),
      humidity: readout.humidity.toFixed(2)
    };
    console.log(data);
    request.post({url: receiver, form: data}, function(err) {
      if(err) console.log('Failed to send to ' + receiver);
    });
    setTimeout(read, 1000);
  }

  read();
  ```

## Run the sensor application
* Insert SD card into the RPi
* Connect Ethernet cable and open SSH connection
* Navigate to `/home/pi/sensor` and install dependencies:
  ```
  # Install Node.js
  curl -sL https://deb.nodesource.com/setup_6.x | sudo -E bash - sudo apt-get install -y nodejs
  # Install dependencies
  npm install
  ```
* Finally, launch the application with `npm start`:
  <img src="./_images/sensor_output.png" height="400">

## Run the receiver application
* Create folder `receiver`
* Create file `./receiver/package.json` with the following contents:
   ```
  {
    "name": "receiver",
    "version": "1.0.0",
    "description": "",
    "main": "index.js",
    "scripts": {
      "test": "echo \"Error: no test specified\" && exit 1"
    },
    "author": "",
    "license": "ISC"
  }
  ```
* Create file `./receiver/index.js` with the following contents:
   ```
  var http = require('http');
  var querystring = require('querystring');

  http.createServer(function (req, res) {
    req.on('data', chunk => {
      var data = querystring.parse(chunk.toString());
      console.log(data);
    });
    req.on('end', () => {
      res.writeHead(200, 'OK', {'Content-Type': 'text/html'})
      res.end('Data received.')
    });
  }).listen(8080);
   ```
* Install Nodejs and dependencies:
  ```
  # Consult your PC platform docs for Node.js install
  npm install
  ```
* Finally, launch the application with `npm start`:
  <img src="./_images/receiver_output.png" height="400">
