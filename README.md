# How to make Tuya Radiator Thermovalve TV02-ZG work with Tuyapi.

## Problem:
Eventhoguh Tuyapi project (https://codetheweb.github.io/tuyapi/) represents a robust solution on how to work with various device that use the Tuya ecosystem, each product is specific.

The thermovalve TV02-ZG looks like this:

<img width="415" alt="Screenshot 2021-10-22 at 12 57 36" src="https://user-images.githubusercontent.com/27240074/138442815-771b0cfc-7d77-4ff2-84b7-cd8c270d9e62.png">

When trying to work with Tuyapi, I discovered the behaviour of the valve isn't aligned with the standard documented steps for the API:
*	The valve is connected via Zigbee bridge, thus the subdevice logic is used. I figured out using the gwID and ID logic in the setup step doesn't have any impact. Only specifying CID in the get/set steps works:

`device.get({cid:'xxxxxxxxxxxxx', schema:true}).then(data => console.log(data));`
* The following setup worked for me:
```
const device = new TuyAPI({
  id: 'xxxxxxxxxx',
  key: 'xxxxxxxxxx',
  ip:'192.168.0.x',
  version: '3.3',
  nullPayloadOnJSONError:true})
```
