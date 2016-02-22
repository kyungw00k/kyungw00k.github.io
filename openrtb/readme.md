# OpenRTB

[openrtb/openrtb2x](https://github.com/openrtb/openrtb2x) 프로젝트 사용 로그를 남겨봅니다.

## Implementation

### DSP
#### org.openrtb.dsp.intf.service.AdvertiserService.java
```java
package org.openrtb.dsp.intf.service;

import java.util.Collection;
import java.util.Collections;
import java.util.List;

import org.openrtb.common.model.Advertiser;
import org.openrtb.common.model.Blocklist;
import org.openrtb.dsp.intf.model.SupplySidePlatform;

/**
 * This service is responsible for retrieving and storing data necessary to
 * support the synchronization of advertiser data between the DSP and SSP.
 * 
 * In order to request {@link Blocklist}s from the SSP, a list of
 * {@link Advertiser}s is required. The only required field is the
 * {@link Advertiser#getLandingPage()}. For more information about populating
 * that object, please refer to the {@link Advertiser} javadoc.
 * 
 * Responses from the SSP need to be persisted. Complete replacements of data
 * will make a call to the
 * {@link #replaceBlocklists(SupplySidePlatform, Collection)}. If the blocklist
 * values being returned are an incremental update to the advertiser, then
 * {@link #updateAdvertiserBlocklists(List)} will be called.
 * 
 * @since 1.0
 */
public interface AdvertiserService {

    public static final String SPRING_NAME = "dsp.intf.AdvertiserService";

    /**
     * Return the list of advertisers you would like to request
     * {@link Blocklist} for. If the desire-ment is to not request any
     * blocklists, then return {@link Collections#emptyList()}.
     *
     * @return return a non-<tt>null</tt> value list of {@link Advertiser}s to
     *         request blocklists for. If the demand-side platform wishes to not
     *         synchronize any advertisers, an empty list will suffice.
     */
    public Collection<Advertiser> getAdvertiserList();

    /**
     * {@link Advertiser}s supplied in this call will have their entire
     * {@link Blocklist} entry replaced in the demand-side store.
     * 
     * Implementers should be aware that {@link Advertiser}s can have no
     * {@link Blocklist}s being returned from the supply-side platform. If this
     * situation arises, then any associated blocklists in the DSP system should
     * be removed. Please see those javadocs for more information.
     * 
     * @param ssp
     *            a non-<tt>null</tt> supply side platform. This SSP is the same
     *            entity that was returned from
     *            {@link IdentificationService#getServiceEndpoints()}.
     * @param advertisers
     *            a non-<tt>null</tt> list of advertisers whose blocklist
     *            contents need to be replaced in the demand-side platform's
     *            persistent store.
     */
    public void replaceBlocklists(SupplySidePlatform ssp, 
                                  Collection<Advertiser> advertisers);

}
```
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