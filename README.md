## Edge-Rpi-GPIO-Access
[](assets/m2m-pub-sub.png)
[](https://raw.githubusercontent.com/EdoLabs/src2/master/quicktour.svg?sanitize=true)

#### Install *m2m* and *array-gpio* on your endpoints.

```js
$ npm install m2m array-gpio
```

### Rpi Server 1 
#### 1. Connect a momentary switch button on pin's 11 and 13.
#### 2. Save the code below as *app.js* in your project directory. 

```js
const m2m = require('m2m')
const r = require('array-gpio')

let edge = new m2m.Edge({name:'rpi server 1'})

let sw1 = r.in(11)
let sw2 = r.in(13)

let led1 = r.out(33)
let led2 = r.out(35)

m2m.connect()
.then(console.log)
.then(() => {

  let server = edge.createServer({port:8125, host:'192.168.0.178', allowInsecure:true})

  server.publish('gpio-11', (tcp) => {
    sw1.watch((state) => {
     if(state){
       led1.on();
       tcp.send({pin:sw1.pin, state:state})     
     }
     else{
       led1.off();
       tcp.send({pin:sw1.pin, state:state})  
     }
    })
  })

  server.publish('gpio-15', (tcp) => {
    sw2.watch((state) => {
     if(state){
       led2.on();
       tcp.send({pin:sw2.pin, state:true})     
     }
     else{
       led2.off();
       tcp.send({pin:sw2.pin, state:false})  
     }
    })
  })

})
.catch(console.log)
```

#### 3. Start rpi server 1 application.

```js
$ node app.js
```

### Rpi Server 2

#### 1. Connect an led on pin 33.

#### 2. Save the code below as *app.js* in your project directory.

```js
const m2m = require('m2m')
const r = require('array-gpio')

let led = r.out(33)

let edge = new m2m.Edge({name:'rpi server 2'})

m2m.connect()
.then(console.log) 
.then(() => {
  
    let port = 8125	
    let host = '192.168.0.158'
    
    edge.createServer({port:port, host:host, allowInsecure:true}, (server) => {
      server.dataSource('gpio-33', (tcp) => {
	      led.pulse(500)
      })	
    })
})
```

#### 3. Start rpi server 2 application.

```js
$ node app.js
```
### Rpi Server 3

#### 1. Connect an led on pin 33.
#### 2. Save the code below as *app.js* in your project directory. <br> The code is similar with rpi server 2 but using using a different host ip. 

```js
const m2m = require('m2m')
const r = require('array-gpio')

let led = r.out(33)

let edge = new m2m.Edge({name:'rpi server 3'})

m2m.connect()
.then(console.log)
.then(() => {
  
    let port = 8125	
    let host = '192.168.0.121'
    
    edge.createServer({port:port, host:host, allowInsecure:true}, (server) => {
      server.dataSource('gpio-33', (tcp) => {
	      led.pulse(500)
      })	
    })
})
```

#### 3. Start rpi server 3 application.

```js
$ node app.js
```

### Rpi Client

The rpi client will serve as the central distributed control application. It will monitor the gpio input pins (11, 13 and 15) from rpi server 1 and appropriately send a pulse signal to the rpi servers' (2 and 3) output led (pin 33) in real-time.  
#### 1. Save the code below as *app.js* in your client project directory.

```js
const m2m = require('m2m') 

let edge = new m2m.Edge()

m2m.connect()
.then(console.log)
.then(() => {

  let ec1 = new edge.client({port:8125, ip:'192.168.0.178', secure:false}) // access rpi 1 gpio input pin 11 and 13 
  let ec2 = new edge.client({port:8125, ip:'192.168.0.158', secure:false}) // access rpi 2 gpio output pin 33
	let ec3 = new edge.client({port:8125, ip:'192.168.0.121', secure:false}) // access rpi 3 gpio output pin 33

  ec1.sub('gpio-11', async (data) => {
		console.log('gpio-11', data)
    if(data.state){
      ec2.read('gpio-33')
		}
	})

  ec1.sub('gpio-13', async (data) => {
		console.log('gpio-13', data)
    if(data.state){
			ec3.read('gpio-33')
		}
	})

})
.catch(console.log)
```

#### 2. Start client application.
```js
$ node app.js
```
You should get a similar output result as shown below.
```js
...

```




