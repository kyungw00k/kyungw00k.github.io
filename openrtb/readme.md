# OpenRTB

[openrtb/openrtb2x](https://github.com/openrtb/openrtb2x) 프로젝트 사용 로그를 남겨봅니다.

## Implementation

### DSP

#### org.openrtb.dsp.intf.service.AdvertiserService.java

#### org.openrtb.dsp.intf.service.IdentificationService.java

```java
package org.openrtb.dsp.intf.service;

import java.util.Collection;

import org.openrtb.common.model.Identification;
import org.openrtb.dsp.intf.model.SupplySidePlatform;

/**
 * Single interface for interacting with identification data, specifically:
 * <ol>
 * <li>identify your organization for making communication
 * {@link #getOrganizationIdentifier()}, and</li>
 * <li>identifying who you want to send requests/response to, along with their
 * service urls and shared secrets</li>.
 * </ol>
 *
 * @since 1.0
 */
public interface IdentificationService {

    public static final String SPRING_NAME = "dsp.intf.IdentificationService";

    /**
     * Returns the non-<tt>null</tt> identification for the organization making
     * the request.  This value will be used to uniquely identify who the
     * request and/or responses are coming from.
     *
     * @return a non-<tt>null</tt> organization identification token. The
     *         {@link Identification#getOrganization()} value should represent
     *         the value the request or response recipients will use to identify
     *         you.
     */
    String getOrganizationIdentifier();

    /**
     * Returns a non-<tt>null</tt> list of supply-side platforms that requests
     * will be sent to. dsp-core will be iterating through these inventory
     * suppliers, building a list of appropriate information for the request
     * from other service calls.
     *
     * Information about the SSPs organization identifier, shared secret, and
     * batch service url are populated in the {@link SupplySidePlatform}. For
     * more information about that information, please see the javadoc for
     * {@link SupplySidePlatform}.
     *
     * @return a non-<tt>null</tt> list of supply-side platforms to request
     *         information from.
     */
    Collection<SupplySidePlatform> getServiceEndpoints();

}

```
### SSP

## Build

```sh
$ git clone https://github.com/openrtb/openrtb2x.git && cd openrtb2x
$ mvn package
```