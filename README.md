# WebBluetooth With CC2541

![](https://raw.githubusercontent.com/DeqingSun/webBluetoothWithCC2541/master/images/ChromeWebBluetooth.gif)

This project shares my experience with WebBluetooth with Bluetooth hardware (CC2541 - Texas Instruments) 

## Hardware Platform

I used a Texas Instruments CC2541 dev board with original "SimpleBLEPeripheral" example. The example come with TI's BLE-Stack package. This example should work on all CC254x board as the example don't need any mandatory hardware to work.

![](https://raw.githubusercontent.com/DeqingSun/webBluetoothWithCC2541/master/images/cc2541project.png)

"SimpleBLEPeripheral" example works as a BLE peripheral. By default, it will be either advertising or connected, never turn off. The example has only one service with UUID 0xFFF0.

The 0xFFF0 service has 5 characteristics:

| UUID   | User Description | Properties | Data type |
| ------ |:----------------:| -----------| --------- |
| 0xFFF1 | Characteristic 1 | Read Write | uint8     |
| 0xFFF2 | Characteristic 2 | Read       | uint8     |
| 0xFFF3 | Characteristic 3 | Write      | uint8     |
| 0xFFF4 | Characteristic 4 | Notify     | uint8     |
| 0xFFF5 | Characteristic 5 | Read       | uint8[5]  |

I used Characteristic 1 only. Also I can read it's value on dev board's LCD screen.

## WebBluetooth API 

There is a good explanation of webBluetooth API on:

<https://developers.google.com/web/updates/2015/07/interact-with-ble-devices-on-the-web>

Also there is a tool to generate code to access bluetooth devices. You can type the filter condition, service and characteristic's UUID. This webpage will generate all core code immediately.

<https://beaufortfrancois.github.io/sandbox/web-bluetooth/generator/>

![](https://raw.githubusercontent.com/DeqingSun/webBluetoothWithCC2541/master/images/generator.png)

I will share what I learned for developers without much Javascript knowledge like me. All accesses to WebBluetooth API are asynchronous. You can not expect to get what you want immediately. Rather that registering all functions with messy nested callbacks, WebBluetooth API used [Promises](https://developers.google.com/web/fundamentals/getting-started/primers/promises). So you can write async requests in a linear way rather than nested.

I'll use my code as example.

```
document.getElementById('buttonConnect').addEventListener('click', function() {
  simpleBLEPeripheral.request()
  .then(_ => simpleBLEPeripheral.connect())
  .then(_ => {return simpleBLEPeripheral.readCharacteristicOne();})
  .then(value => {return simpleBLEPeripheral.writeCharacteristicOne(new Uint8Array([value.getUint8() + 1]));})
  .then(_ => {return simpleBLEPeripheral.readCharacteristicOne()})
  .catch(error => {
     console.log(error)
  });
});
```
When a "then" block returns another Promise, next "then" block will hold on until the previous async request is finished.

Data you read from BLE device is in a DataView object. You can get it's value using getUint8() or other similar methods. You can use a Uint8Array object to write to BLE device.

## Get Connected

To use WebBluetooth API, you need either enable WebBluetooth flag in <chrome://flags/#enable-web-bluetooth>

![](https://raw.githubusercontent.com/DeqingSun/webBluetoothWithCC2541/master/images/flags.png)

Or you can use Origin Trials on your webpage. You can apply for one on <http://bit.ly/WebBluetoothOriginTrial>. Once you get an email with the token you can add \<meta\> tag into your page's \<head\>. Then you will be able to connect to WebBluetooth in that page on any compatible device.

Finally you can host your page on localhost or a https server. You can also try mine on 

<https://deqingsun.github.io/webBluetoothWithCC2541/webBluetoothSimpleBLEPeripheral.html>
