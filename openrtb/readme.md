# OpenRTB

[openrtb/openrtb2x](https://github.com/openrtb/openrtb2x) 프로젝트 사용 로그를 남겨봅니다.

## Specification

## Implementation
Need to implement your own client

### DSP
This package is an example implementation of the 'dsp-intf' to satisfy the following: 1) Integration testing between the 'dsp-web' and 'ssp-web' projects. 2) Provide a framework for other DSPs to leverage this specification more quickly by allowing the DSP to focus on the integration aspects of the specification with their own specific platform.

As a result of requirement (1) above, this package must be capable of working in concert with the 'ssp-client' described below.



* [`org.openrtb.dsp.intf.service.AdvertiserService.java`](https://github.com/openrtb/openrtb2x/blob/2.0/demand-side/dsp-intf/src/main/java/org/openrtb/dsp/intf/service/AdvertiserService.java)
* [`org.openrtb.dsp.intf.service.IdentificationService.java`](https://github.com/openrtb/openrtb2x/blob/2.0/demand-side/dsp-intf/src/main/java/org/openrtb/dsp/intf/service/IdentificationService.java)

### SSP
This package is an example implementation of the 'ssp-intf' to satisfy the following: 1. Integration testing between the 'ssp-web' and 'dsp-web' projects. 2. Provide a framework for other SSPs to leverage this specification more quickly by allowing the SSP to focus on the integration aspects of the specification with their own specific platform.

As a result of requirement (1) above, this package must be capable of working in concert with the 'dsp-client' described above.


* [`org.openrtb.ssp.SupplySideService.java`](https://github.com/openrtb/openrtb2x/blob/2.0/supply-side/ssp-intf/src/main/java/org/openrtb/ssp/SupplySideService.java)

## Build

```sh
$ git clone https://github.com/openrtb/openrtb2x.git && cd openrtb2x
$ mvn package
```