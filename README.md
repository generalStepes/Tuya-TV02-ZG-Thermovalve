# How to make Tuya Radiator Thermovalve TV02-ZG work with Tuyapi

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

* Still, when quering Tuyapi, the valve was returning cached values and `DP_REFRESH` packet was always sent empty. None of the steps available in Tuyapi 7.2.0 helped here (I tried sending `refresh()`, `refresh(dp:xx)`, `issueRefreshOnConnect: true` but non of these triggered data refresh on the valve).

## Solution:
* When debugging through Tuya Cloud I noticed there's a rather unusual command `Check01` sent that seemingly refreshes the data.

<img width="1005" alt="Screenshot 2021-10-22 at 13 07 54" src="https://user-images.githubusercontent.com/27240074/138444063-b5a3d393-0628-42a2-894c-bde39b2858d6.png">

* Running out of options, I tried re-setting various DPSs and getting their values to understand how the refreshes work as `Check01` seems to be absolutely undocumented. I figured out setting the DPS 115 to `true` or `1` helps. The value is that reset after each query automatically. 

`device.set({cid: 'xxxxxxx', dps:115,set:true}`

This seems to do the trick as DPS 115 likely corresponds to `Check01` as it is recognised as such in Tuya Cloud logs. When querying Tuyapi with the `DPS 115:true`, the `DP_REFRESH` is sent with current values. However, it seems that `DPS 24`, which corresponds to the current temperature is sent only when the value changed.

```
  TuyAPI Received DP_REFRESH packet. +3ms
DP_REFRESH data from device:  { dps: { '24': 211 }, cid: 'xxxxxxx', t: 1634899900 }
  TuyAPI Received data: 000055aa00000000000000080000005b00000000332e33000000000000c0950000000170579292b9c947f35bcaa10ab7e8487456e4de1574acc271e058f6d283c164177da0e52e7b8734a7c1430157c56ee524f98a32fef737d6cb12acf373b842939ceaa9c5750000aa55 +2ms
  TuyAPI Parsed: +2ms
  TuyAPI { payload:
  TuyAPI    { dps: { '24': 211 }, cid: 'xxxxxxx', t: 1634899900 },
  TuyAPI   leftover: false,
  TuyAPI   commandByte: 8,
  TuyAPI   sequenceN: 0 } +1ms
  ```
This refreshes the data completely.

* There might be a better way to do it. So please note these steps were composed rather ad-hoc just in case they help with your projects :-).



