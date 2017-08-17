![](./media/image06.png)

#Lab 2 - Traffic Management for APIs

![](./media/image18.png)

*Duration : 30 mins*

*Persona : API Team / Security*
# Use case

You have a requirement to protect your target servers (backend) from traffic spikes. You would like to protect your APIs from denial of service attacks that might lead to performance lags or downtime of your backend.

# How can Apigee Edge help?

By exposing an API through Apigee Edge, you gain the ability to modify and monitor its behavior using out-of-the-box policies. Edge's out-of-the-box policies enable you to enhance your API with sophisticated features to control traffic, enhance performance, enforce security, and increase the utility of your APIs, without requiring you to write any code or to modify any backend services. Extension policies enable you to implement custom logic in the form of JavaScript, Python, Java, and XSLT.

In this lab we will see how to use an out of the box traffic management policy, "Spike Arrest", to protect against traffic spikes. A Spike Arrest policy can throttle the number of requests processed by an API proxy and sent to a backend, protecting against performance lags and downtime.

# Pre-requisites

Apigee Edge API Proxy created in earlier lab exercise. If not, jump back to "API Design - Create a Reverse Proxy with OpenAPI specification" lab.


## Background

To maintain performance and availability across a diverse base of client
apps, it's critical to maintain app traffic within the limits of the
capacity of your APIs and backend services. It's also important to
ensure that apps don't consume more resources than permitted.

Apigee Edge provides three policies that enable you to optimize
traffic management to minimize latency for apps while maintaining the
health of backend services.

> NB, a policy in Apigee Edge is a unit of re-usable logic that performs
a certain task. There are policy types for transforming messages,
caching data, checking tokens or keys, creating tokens, and so on.

Each of the traffic management policy types addresses a distinct aspect
of traffic management. In some cases, you might use the three policy
types in a single API proxy.

In addition, Apigee Edge includes a cache, that can also be accessed via
policies, to manage traffic and handling of requests. 


### Spike Arrest 

This policy smoothes traffic spikes by dividing a limit that you define
into smaller intervals. For example, if you define a limit of 100
messages per second, the Spike Arrest policy enforces a limit of about 1
request every 10 milliseconds (1000 / 100); and 30 messages per minute
is smoothed into about 1 request every 2 seconds (60 / 30). The Spike
Arrest limit should be close to capacity calculated for either your
backend service or the API proxy itself. The limit should also be
configured for shorter time intervals, such as seconds or minutes. This
policy should be used to prevent sudden traffic bursts caused by
malicious attackers attempting to disrupt a service using a
denial-of-service (DOS) attack or by buggy client applications.  See
[SpikeArrest](http://docs.apigee.com/api-services/reference/spike-arrest-policy).

### Quota 

This policy enforces consumption limits on client apps by maintaining a
distributed 'counter' that tallies incoming requests. The counter can
tally API calls for any identifiable entity, including apps, developers,
API keys, access tokens, and so on. Usually, API keys are used to
identify client apps. This policy is computationally expensive so, for
high-traffic APIs, it should configured for longer time intervals, such
as a day or month. This policy should be used to enforce business
contracts or SLAs with developers and partners, rather than for
operational traffic management.  See [Quota](http://docs.apigee.com/api-services/reference/quota-policy).

### Concurrent Rate Limit

This policy enables traffic management between API Services and your
backend services. Some backend services, such as legacy applications,
may have strict limits on the number of simultaneous connections they
can support. This policy enforces a limit on the number of requests that
can be sent at any given time from API services to your backend
service. This number is counted across all of the distributed instances
of API Services that may be calling your backend service. Policy limits
and time duration should be configured to match the capacity available
for your backend service.  See [Concurrent Rate Limit](http://docs.apigee.com/api-services/reference/concurrent-rate-limit-policy).


### Caching Policies

Apigee Edge supports different caching policies enabling you to:

* Reduce latency: A request satisfied from the cache gets the
  representation and displays it in a shorter time. The server is more
  responsive with requests satisfied from the cache, which is closer to
  the client than the origin server.

* Reduce network traffic: Representations are reused, reducing the
  impact of processing duplicate or redundant requests. Using a cache
  also reduces the amount of bandwidth you use.

* Persist data across transactions: You can store session data for reuse
  across HTTP transactions.

* Support security:You may need "scope" access to the contents of a
  cache so it can only be accessed in a particular environment or by a
  specific API proxy.

The various caching policies supported by Apigee Edge are:

| Policy name    | Description |
| :------------- | :---------- |
| ResponseCache | Uses a cache to cache the response  from the backend resource. This reduces the number of requests to the resource. When the response is already available in cache, it is returned directly by Edge, without contacting the backend.  [ResponseCache policy](http://apigee.com/docs/api-services/reference/response-cache-policy). |
| PopulateCache | Write arbitrary data, not necessarily response content, to the cache. You can use composite keys for the cache, so that you could cache user information, client information, Nonces, and so on. Each cache item can be tagged with an individual Time-to-Live, so data can be available for 10 seconds or 10 hours or 10 days, etc. [PopulateCache](http://apigee.com/docs/api-services/reference/populate-cache-policy). |
| LookupCache | Retrieve values that have been cached with the PopulateCache policy. [LookupCache](http://apigee.com/docs/api-services/reference/lookup-cache-policy). |
| InvalidateCache | Purges values that have been cached by PopulateCache. [Invalidate Cache policy](http://apigee.com/docs/api-services/reference/invalidate-cache-policy). |


## Part 1: Adding a Spike Arrest Policy

**Estimated Time: 6 minutes**

* Go to the [Apigee Edge Management UI](https://apigee.com/edge) and log in. This is the Edge management UI.

* Select **Develop → API Proxies** in the side navigation menu.

![image alt text](./media/image_2.png)

* Click on **{your_initials}**\_payments\_api that you have created in earlier lab exercise.

![image alt text](./media/image_3.png)

* Click on **Develop** tab to access API Proxy development dashboard.

![image alt text](./media/image_4.png)

![image alt text](./media/image_5.png)

* Click on **PreFlow** under Proxy Endpoint default.

![image alt text](./media/image_6.png)

  Each proxy has a number of different places at which developers can
  attach policies. You can specify that your policies will run during
  request processing, or during response processing - *Request flow* or
  *Response flow* in Apigee parlance. For each of those, you can set up
  conditions to apply policies to a subset of cases - for example a
  condition will typically match a url path and verb.  `GET /payments`
  might have a conditional flow. There is also a "preflow" and
  "postflow" which act like wildcards - the policies attached here will
  run all requests on this proxy, before or after any conditional flow,
  respectively.

* Click on **+ Step** on top of Request Flow to attach a spike arrest policy.  
![](./media/image_6a.png)

  By doing this, you are adding a policy step that will run in the
  Request Pre-flow. It will run for all inbound requests. You will see a
  modal dialog that allows you to select a particular policy:   
  ![](./media/image_6b.png)  

  Feel free to scroll to examine the list of available policies. 

* Select the **Spike Arrest Policy**. Click on **Add** button to add spike arrest policy to proxy endpoint reflow request.  
![](./media/image_7.png)

 You have now added a policy, and you will be presented with this view:  
![](./media/image_8.png)

* Modify the configuration of the policy, specifying "12pm" - 12 per minute - for the Rate.   
![](./media/image_8a.png)

  This shows you the basics related to configuring policies in Apigee
  Edge. There are numerous policy types, and for each one, you may
  configure certain properties to affect their behavior.

  Your configuration should look
  something like the following. The DisplayName and name are not of critical importance:

```
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<SpikeArrest async="false" continueOnError="false" enabled="true" name="Spike-Arrest-1">
    <DisplayName>Spike Arrest-1</DisplayName>
    <Properties/>
    <Rate>12pm</Rate>
</SpikeArrest>
```


Think of Spike Arrest as a way to generally protect against traffic spikes (system wide) rather than as a way to limit traffic to a specific number of requests *per user*. Your APIs and backend can handle a certain amount of traffic, and the Spike Arrest policy helps you smooth traffic to the general amounts you want, or to set safety limits.

The runtime Spike Arrest behavior differs from what you might expect to see from the literal per-minute or per-second values you enter.

For example, say you enter a rate of 30pm (30 requests per minute). In testing, you might think you could send 30 requests in 1 second, as long as they came within a minute. But that's not how the policy enforces the setting. Thirty requests inside a 1-second period could be considered a mini spike in some environments.

Instead, Spike Arrest smooths the allowed traffic by dividing your settings into smaller intervals:
* Interval Options:
  * **Per-minute** rates get smoothed into requests allowed intervals of
    **seconds**. For example, 30pm gets smoothed like this: 60 seconds
    (1 minute) / 30pm = 2-second intervals, or about 1 request allowed
    every 2 seconds. A second request inside of 2 seconds will fail.
    Also, a 31st request within a minute will fail.

  * **Per-second** rates get smoothed into requests allowed in intervals
    of **milliseconds**. For example, 10ps gets smoothed like this: 1000
    milliseconds (1 second) / 10ps = 100-millisecond intervals, or about
    1 request allowed every 100 milliseconds . A second request inside
    of 100ms will fail. Also, an 11th request within a second will fail.

* Click on **Save** button to save the proxy and ensure that it is deployed successfully to the **test** environment.
![](./media/image_9.png)

  * *Congratulations!*...You have now secured your backend against denial of service attacks, performance lags or downtime of target servers.

* Let us test the updated API proxy using the Trace Console. Click on **Trace** tab.
![](./media/image_10.png)

* Click on **Start Trace Session** to see API Proxy with spike arrest in action.
![](./media/image_11.png)

* Click on **Send** button multiple times, You will see 500 response code when spike arrest policy kicks in to protect target servers from spike in traffic. Note to add the **/payments** to the url to execute the return function of all payments.

	![](./media/image_12.png)

   * You can use the [Apigee Rest Client](https://apigee-rest-client.appspot.com/) to test Spike Arrest Policy.
![image alt text](./media/image_13.png)


   * Or you can use Postman to quickly send more than 2 requests in 6 seconds and observe that certain requests will receive an error with the errorCode `policies.ratelimit.SpikeArrestViolation`.

* Let's use the Trace UI to examine what's happening. Return to the Edge UI, and select the Trace tab again. 
![](./media/image_14.png)

* Return to testing tool, and send a few more requests. Now back to the Edge UI.  Examine the trace. You should see some requests returning 500 errors, indicating that the SpikeArrest has sent back the call. 
![](./media/image_12.png)

* Stop the Trace session.


## Part 2: Adding a Response Cache

**Estimated Time: 10 minutes**

Now we'll introduce the ResponseCache policy.  

* Go to the [Apigee Edge Management UI](https://apigee.com/edge) and log in. This is the Edge management UI.

* Select **Develop → API Proxies** in the side navigation menu.

![image alt text](./media/image_2.png)

* Click on **{your_initials}**\_payments\_api that you have created in earlier lab exercise.

![image alt text](./media/image_3.png)

* Click on **Develop** tab to access API Proxy development dashboard.

![image alt text](./media/image_4.png)

![image alt text](./media/image_5.png)

* Click on **PreFlow** under Proxy Endpoint default, Click on “+ Step” on the Request Flow

![](./media/image_19.png)

* From the resulting modal dialog, Select the **Response Cache** policy, and click Add.
You should see something like the following: 
![](./media/image_20.png)

  You have now added a second policy to the Request Pre-Flow. Remember,
  these are policies that will run for every inbound request handled by
  this proxy. They run in sequence. You get to specify the order. 

  The configuration for this policy looks like the following:
  ```
  <ResponseCache async="false" continueOnError="false" enabled="true" name="Response-Cache-1">
    <DisplayName>Response Cache-1</DisplayName>
    <Properties/>
    <CacheKey>
        <Prefix/>
        <KeyFragment ref="request.uri" type="string"/>
    </CacheKey>
    <Scope>Exclusive</Scope>
    <ExpirySettings>
        <ExpiryDate/>
        <TimeOfDay/>
        <TimeoutInSec ref="">3600</TimeoutInSec>
    </ExpirySettings>
    <SkipCacheLookup/>
    <SkipCachePopulation/>
  </ResponseCache>
  ```

  This stipulates that the cache should timeout after 3600 seconds, or
  one hour. You can modify this to whatever setting you like. As implied
  by the configuration options, you can also set the expiry to a
  particular time of day - say 3:00am Pacific time each day. This allows
  you to hold a cache all day, and refresh it at the beginning of each
  day, for example. For more information, see [the
  documentation](http://apigee.com/docs/api-services/reference/response-cache-policy).

  Apigee Edge provides a default cache resource that can be used for
  quick testing, which is what is being used in this lesson. Cache
  policies like ResponseCache can also used named cache resources.  A
  Named cache resource can be manipulated administratively, outside of
  policy control. For examine, if you would like to clear a cache
  administratively, you can do that with a named cache resource. It
  takes just a moment.

  For more information on Cache Resources, see [Manage Caches for an
  Environment](http://apigee.com/docs/api-services/content/manage-caches-environment).

* Your Proxy Endpoints → Default → PreFlow should now look as follows:
![](./media/image_21.png)

* Click the Target Endpoints → default → PostFlow
![](./media/image_22.png)

* Verify that it looks as follows:
![](./media/image_23.png)

* **Save** the changes to the API Proxy, and wait for it to successfully deploy.

* Now, let's test the modifications you've made.  Click the **Trace** tab and press **Start Trace Session**.

* Send a test **/GET payment** request.

  * You can use the [Apigee Rest Client](https://apigee-rest-client.appspot.com/) to test Response Cache Policy.
![](./media/image_24.png)

  * From Postman, send a test ‘/GET payment’ request from Postman. In the Postman UI, you should see a success response, and a timing. Wait for 6 to 10 seconds, to avoid the Spike Arrest policy from stopping your requests) and send the same request again from Postman. You should see a faster response this time.
![](./media/image_25.png)

* Go back to the Edge UI, in the Trace view and review the transaction
  map of both the requests including the overall elapsed time to process
  both requests.

  The first transaction map will show you that the request was proxied
  to the backend. It should look as follows:
  ![](./media/image_26.png)

  The view for the second transaction will show you that the response
  was served from cache. It will look as follows:
  ![](./media/image_27.png)

  After configuring the Response Cache policy, as expected, after the
  initial request, the second and all other requests for the next 3600
  seconds will be served from the cache and hence avoid executing any
  other policies. Since no request is sent to the target service, the
  overall transaction time has also dropped significantly. This can also
  increase concurrency at high transaction rates.


# Lab Video

If you like to learn by watching, here is a short video on using Spike Arrest policy. [https://youtu.be/3Gi-GGTqllg](https://youtu.be/3Gi-GGTqllg)

# Earn Extra-points

Now that you have protected your backend against spike in traffic, Explore more about spike arrest policy using docs here, [http://docs.apigee.com/api-services/reference/spike-arrest-policy](http://docs.apigee.com/api-services/reference/spike-arrest-policy) & update the policy with identifer like queryparam on which spike arrest counter is updated. Use message weight property to assign a weight to the counter.

# Quiz

1. Assume there is a single message processor, Spike Arrest rate limit is 2 pm, What is the interval of time in which a single API Request is allowed to reach target server ?

2. Is it possible to implement spike arrest based on run time variables like per developer / per app ?

3. Suppose you modified the ResponseCache policy to have a TTL of 10 seconds.  Then, suppose you sent new requests in.  At this point, Would you expect the cache to be re-filled every 10 seconds?  Explain.

4. What would you need to do in order to allow an administrator the ability to clear the cache?

# Summary

That completes this hands-on lesson. You learned how to use the Spike
Arrest to protect the environment from traffic spikes and to use the
Response Cache policy to provide a better overall experience for the API
consumer while reducing network traffic. Obviously like any other
policy, these policies must be used appropriately based upon your use
cases.

# References

* Useful Apigee documentation links on Traffic Management & Spike Arrest Policy  - 

    * Spike Arrest Policy - http://docs.apigee.com/api-services/reference/spike-arrest-policy

    * Rate Limiting - [http://docs.apigee.com/api-services/content/rate-limiting](http://docs.apigee.com/api-services/content/rate-limiting)

    * Comparing Rate Limiting Policies - [http://docs.apigee.com/api-services/content/comparing-quota-spike-arrest-and-concurrent-rate-limit-policies](http://docs.apigee.com/api-services/content/comparing-quota-spike-arrest-and-concurrent-rate-limit-policies) 
    
    * Response Cache Policy - [http://docs.apigee.com/api-services/reference/response-cache-policy](http://docs.apigee.com/api-services/reference/response-cache-policy)

# Rate this lab

How did you like this lab? Rate [here](https://goo.gl/forms/O1CpSFvPohgRJuSj1).

Now go to [Lab-3](./../Lab%203%20API%20Diagnostics%20-%20Trace%20tool)

