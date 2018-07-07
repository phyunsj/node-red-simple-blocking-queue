# Node-RED Simple Blocking Queue

Simple blocking queue (size : 20 for this demo) using Function Node. 
**Node-RED version v0.18.6**

#### Dependencies 

> npm install node-red-contrib-mqtt-broker

## In Action

- MQTT connected

<p align="center">
<img src="https://github.com/phyunsj/node-red-simple-blocking-queue/blob/master/blockedqueue-connected.gif" width="700px"/>
</p>

- MQTT disconnected

<p align="center">
<img src="https://github.com/phyunsj/node-red-simple-blocking-queue/blob/master/blockedqueue-disconnected.gif" width="700px"/>
</p>

## Flow

```
[{"id":"a0a12d74.93df","type":"debug","z":"f8887b8f.b20468","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"payload","x":728.1666717529297,"y":114.66667461395264,"wires":[]},{"id":"4ea76ef4.0160c","type":"inject","z":"f8887b8f.b20468","name":"alarm trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":105.1666488647461,"y":149.8888897895813,"wires":[["6a2146c.b045eb8"]]},{"id":"786a05e1.341ebc","type":"function","z":"f8887b8f.b20468","name":"SimpleBlockingQueue","func":"context.queue = context.queue || [];\n\nvar totalAllowedQueueSize = 20;\nvar mqtt_connected = false;\nvar blocked = false;\n\n// mqtt out node\nvar nextNode = RED.nodes.getNode(\"ae7f8962.dd8b18\");\n// status check\nmqtt_connected = nextNode.brokerConn.connected;\nif (context.queue.length >= totalAllowedQueueSize) blocked = true;\n\nif (msg.payload.op == \"reset\"  ) {\n   context.queue = [];\n} else if ( msg.payload.op == \"trigger\" ) {\n    // dequeue\n    if ( context.queue.length === 0 ) msg = null; // no action\n    else {\n       if ( mqtt_connected ) msg.payload = context.queue.shift();\n    }\n} else {\n    // enqueue\n    if ( !blocked ) context.queue.push(msg.payload);\n}\n\nif (mqtt_connected) {\n    if( !blocked )\n        node.status({fill:\"green\",shape:\"ring\",text: '('+context.queue.length+') stored,connected' });\n    else \n        node.status({fill:\"red\",shape:\"ring\",text: 'blocked,('+context.queue.length+') stored,connected' });\n} else {\n    if( !blocked ) \n        node.status({fill:\"red\",shape:\"ring\",text: '('+context.queue.length+') stored,disconnected' });\n    else\n        node.status({fill:\"red\",shape:\"ring\",text: 'blocked,('+context.queue.length+') stored,disconnected' });\n}\n\nreturn msg;","outputs":1,"noerr":0,"x":528.1666717529297,"y":167.22222328186035,"wires":[["a0a12d74.93df","ae7f8962.dd8b18"]]},{"id":"614fc9a1.8f7bb8","type":"inject","z":"f8887b8f.b20468","name":"report trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":105.16664123535156,"y":192.88888931274414,"wires":[["a73b7a92.6b5058"]]},{"id":"6a2146c.b045eb8","type":"function","z":"f8887b8f.b20468","name":"alarm generator","func":"\nvar trigger  = Math.floor(Math.random() * 6);\n\nmsg.payload.timestamp = Date.now();\nswitch ( trigger ) {\ncase 0 :\ncase 2 :\ncase 4 :\n    msg.payload.op = 'report';\n    msg.payload.priority = 1;\n    msg.payload.task = 'Gas Leak';\n    break;  \ncase 1 :\ncase 3 :\ncase 5 :\n    msg.payload.op = 'report';\n    msg.payload.priority = 2;\n    msg.payload.task = 'Water Leak';\n    break;    \ndefault:\n    msg.payload.op = 'report';\n    msg.payload.priority = 'low';\n    msg.payload.task = 'Gas Task';\n    break;\n}\n\nreturn msg;","outputs":1,"noerr":0,"x":294.1666717529297,"y":151.11111164093018,"wires":[["786a05e1.341ebc"]]},{"id":"a73b7a92.6b5058","type":"function","z":"f8887b8f.b20468","name":"mqtt out trigger","func":"msg.payload = { op : 'trigger' } ;\nreturn msg;","outputs":1,"noerr":0,"x":294.1666679382324,"y":194.1111125946045,"wires":[["786a05e1.341ebc"]]},{"id":"ae7f8962.dd8b18","type":"mqtt out","z":"f8887b8f.b20468","name":"MQTT pub - node1/alarm","topic":"node1/alarm","qos":"","retain":"","broker":"d139ad3f.b28af","x":762.1666717529297,"y":215.77779293060303,"wires":[]},{"id":"2e679e40.1f3272","type":"mosca in","z":"f8887b8f.b20468","mqtt_port":"1883","mqtt_ws_port":"","name":"","username":"","password":"","dburl":"","x":121.16667175292969,"y":284.8888912200928,"wires":[["248405cc.406f8a"]]},{"id":"248405cc.406f8a","type":"debug","z":"f8887b8f.b20468","name":"","active":false,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":314.1666717529297,"y":284.7777805328369,"wires":[]},{"id":"ff2a51fd.27ce6","type":"debug","z":"f8887b8f.b20468","name":"","active":true,"tosidebar":true,"console":false,"tostatus":false,"complete":"false","x":342.1666679382324,"y":329.7777805328369,"wires":[]},{"id":"a1bd693d.21c098","type":"mqtt in","z":"f8887b8f.b20468","name":"MQTT sub - node1/alarm","topic":"node1/alarm","qos":"2","broker":"d139ad3f.b28af","x":131.1666717529297,"y":329.8888912200928,"wires":[["ff2a51fd.27ce6"]]},{"id":"bd1934b2.be82a8","type":"inject","z":"f8887b8f.b20468","name":"reset trigger","topic":"","payload":"","payloadType":"date","repeat":"","crontab":"","once":false,"onceDelay":0.1,"x":106.16667175292969,"y":233.88889122009277,"wires":[["c25a3456.5d7378"]]},{"id":"c25a3456.5d7378","type":"function","z":"f8887b8f.b20468","name":"reset trigger","func":"msg.payload = { op : 'reset' } ;\nreturn msg;\n","outputs":1,"noerr":0,"x":284.1666564941406,"y":234.31945991516113,"wires":[["786a05e1.341ebc"]]},{"id":"d139ad3f.b28af","type":"mqtt-broker","z":"","broker":"localhost","port":"1883","clientid":"","usetls":false,"compatmode":true,"keepalive":"60","cleansession":true,"birthTopic":"","birthQos":"0","birthPayload":"","willTopic":"","willQos":"0","willPayload":""}]
```

## MQTT Connection Status Detection

- `RED.nodes.getNode(` flow ID `)` 

```
// mqtt out node
var nextNode = RED.nodes.getNode("ae7f8962.dd8b18");
// status check
mqtt_connected = nextNode.brokerConn.connected;
```

<p align="center">
<img src="https://github.com/phyunsj/node-red-simple-blocking-queue/blob/master/mqtt-out-flow-id.png" width="500px"/>
</p>

- `nodes/core/core/80-function.js`

```
        RED: {
                util: RED.util,
+               nodes : RED.nodes
 
            },
+       require : require,  // If you wish to call "require()" in your function node. 
                            // Otherwise,  create a custome node 
                            // or adjust "functionGlobalContext" in settings.js
```

## Persistent Storage Options

- [node-red-contrib-key-value-store](https://github.com/boneskull/node-red-contrib-key-value-store) using lowdb
- [node-red-contrib-msq-queue](https://github.com/damoclark/node-red-contrib-msg-queue) using sqlite3


