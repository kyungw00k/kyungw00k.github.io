# OpenRTB

[openrtb/openrtb2x](https://github.com/openrtb/openrtb2x) 프로젝트 사용 로그를 남겨봅니다.

## Implementation
Need to implement your own client
### DSP
This package is the contract and set of interfaces between the message handling code located in the 'dsp-core' and a DSP's specific internal representation of that data.

* [`org.openrtb.dsp.intf.service.AdvertiserService.java`](https://github.com/openrtb/openrtb2x/blob/2.0/demand-side/dsp-intf/src/main/java/org/openrtb/dsp/intf/service/AdvertiserService.java)
* [`org.openrtb.dsp.intf.service.IdentificationService.java`](https://github.com/openrtb/openrtb2x/blob/2.0/demand-side/dsp-intf/src/main/java/org/openrtb/dsp/intf/service/IdentificationService.java)

### SSP
* []()
* []()
## Build

```sh
$ git clone https://github.com/openrtb/openrtb2x.git && cd openrtb2x
$ mvn package
```