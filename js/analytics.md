---
---

# Analytics

The Analytics category enables you to collect analytics data for your app. The Analytics category comes with built-in support for [Amazon Pinpoint](#using-amazon-pinpoint) and [Amazon Kinesis](#using-amazon-kinesis).

<b>Prerequisite:</b> [Install and configure the Amplify CLI](..)<br>
<b>Recommendation:</b> [Complete the Getting Started guide](./start?platform=purejs)
{: .callout .callout--info}

#### Automated Setup

Run the following command in your project's root folder:

```bash
$ amplify add analytics
```

The CLI will prompt configuration options for the Analytics category such as Amazon Pinpoint resource name and analytics event settings.

{The Analytics category utilizes the Authentication category behind the scenes to authorize your app to send analytics events.}
{: .callout .callout--info}

The `add` command automatically creates a backend configuration locally. To update your backend run:

```bash
$ amplify push
```

A configuration file called `aws-exports.js` will be copied to your configured source directory, for example `./src`. The CLI will also print the URL for Amazon Pinpoint console to track your app events.  

**NOTE**: If your Analytics resources were created with Amplify CLI version 1.6.4 and below, you will need to manually update your project to avoid Node.js runtime issues with AWS Lambda. [Read more]({%if jekyll.environment == 'production'%}{{site.amplify.docs_baseurl}}{%endif%}/cli/lambda-node-version-update)

##### Configure Your App

Import and load the configuration file in your app. It's recommended you add the Amplify configuration step to your app's root entry point. For example `App.js` in React or `main.ts` in Angular.

```javascript
import Amplify, { Analytics } from 'aws-amplify';
import awsconfig from './aws-exports';

Amplify.configure(awsconfig);
```

#### Manual Setup

The manual setup enables you to use your existing Amazon Pinpoint resource in your app.

```javascript
import Amplify from 'aws-amplify';

Amplify.configure({
    // To get the AWS Credentials, you need to configure 
    // the Auth module with your Cognito Federated Identity Pool
    Auth: {
        identityPoolId: 'us-east-1:xxx-xxx-xxx-xxx-xxx',
        region: 'us-east-1'
    },
    Analytics: {
        // OPTIONAL - disable Analytics if true
        disabled: false,
        // OPTIONAL - Allow recording session events. Default is true.
        autoSessionRecord: true,

        AWSPinpoint: {
            // OPTIONAL -  Amazon Pinpoint App Client ID
            appId: 'XXXXXXXXXXabcdefghij1234567890ab',
            // OPTIONAL -  Amazon service region
            region: 'XX-XXXX-X',
            // OPTIONAL -  Customized endpoint
            endpointId: 'XXXXXXXXXXXX',
            // OPTIONAL - Default Endpoint Information
            endpoint: {
                address: 'xxxxxxx', // The unique identifier for the recipient. For example, an address could be a device token, email address, or mobile phone number.
                attributes: {
                    // Custom attributes that your app reports to Amazon Pinpoint. You can use these attributes as selection criteria when you create a segment.
                    hobbies: ['piano', 'hiking'],
                },
                channelType: 'APNS', // The channel type. Valid values: APNS, GCM
                demographic: {
                    appVersion: 'xxxxxxx', // The version of the application associated with the endpoint.
                    locale: 'xxxxxx', // The endpoint locale in the following format: The ISO 639-1 alpha-2 code, followed by an underscore, followed by an ISO 3166-1 alpha-2 value
                    make: 'xxxxxx', // The manufacturer of the endpoint device, such as Apple or Samsung.
                    model: 'xxxxxx', // The model name or number of the endpoint device, such as iPhone.
                    modelVersion: 'xxxxxx', // The model version of the endpoint device.
                    platform: 'xxxxxx', // The platform of the endpoint device, such as iOS or Android.
                    platformVersion: 'xxxxxx', // The platform version of the endpoint device.
                    timezone: 'xxxxxx' // The timezone of the endpoint. Specified as a tz database value, such as Americas/Los_Angeles.
                },
                location: {
                    city: 'xxxxxx', // The city where the endpoint is located.
                    country: 'xxxxxx', // The two-letter code for the country or region of the endpoint. Specified as an ISO 3166-1 alpha-2 code, such as "US" for the United States.
                    latitude: 0, // The latitude of the endpoint location, rounded to one decimal place.
                    longitude: 0, // The longitude of the endpoint location, rounded to one decimal place.
                    postalCode: 'xxxxxx', // The postal code or zip code of the endpoint.
                    region: 'xxxxxx' // The region of the endpoint location. For example, in the United States, this corresponds to a state.
                },
                metrics: {
                    // Custom metrics that your app reports to Amazon Pinpoint.
                },
                /** Indicates whether a user has opted out of receiving messages with one of the following values:
                 * ALL - User has opted out of all messages.
                 * NONE - Users has not opted out and receives all messages.
                 */
                optOut: 'ALL',
                // Customized userId
                userId: 'XXXXXXXXXXXX',
                // User attributes
                userAttributes: {
                    interests: ['football', 'basketball', 'AWS']
                    // ...
                }
            },

            // Buffer settings used for reporting analytics events.
            // OPTIONAL - The buffer size for events in number of items.
            bufferSize: 1000,

            // OPTIONAL - The interval in milliseconds to perform a buffer check and flush if necessary.
            flushInterval: 5000, // 5s 

            // OPTIONAL - The number of events to be deleted from the buffer when flushed.
            flushSize: 100,

            // OPTIONAL - The limit for failed recording retries.
            resendLimit: 5
        }
    }
});
```

User session data is automatically collected unless you disabled analytics. To see the results visit the [Amazon Pinpoint console](https://console.aws.amazon.com/pinpoint/home/).
{: .callout .callout--info}

#### Update your IAM Policy:

Amazon Pinpoint service requires an IAM policy in order to use the `record` API:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "mobiletargeting:UpdateEndpoint",
                "mobiletargeting:PutEvents"
            ],
            "Resource": [
                "arn:aws:mobiletargeting:*:${accountID}:apps/${appId}*"
            ]
        }
    ]
}
```

If you get the error message: `Exceeded maximum endpoint per user count 10` when updating the endpoints, you can update the Policy with the Action: `mobiletargeting:GetUserEndpoints` which will allow the Analytics module to get the endpoints info and remove unused endpoints automatically.

### Working with the API 

#### Recording Custom Events

To record custom events call the `record` method:

```javascript
Analytics.record({ name: 'albumVisit' });
```

#### Record a Custom Event with Attributes

The `record` method lets you add additional attributes to an event. For example, to record *artist* information with an *albumVisit* event:

```javascript
Analytics.record({
    name: 'albumVisit', 
    // Attribute values must be strings
    attributes: { genre: '', artist: '' }
});
```

Attribute values must have the type `String` or be an array of strings.

#### Record Engagement Metrics

Data can also be added to an event:

```javascript
Analytics.record({
    name: 'albumVisit', 
    attributes: {}, 
    metrics: { minutesListened: 30 }
});
```

Metric values must be a `Number` type such as a float or integer.

#### Disable Analytics

You can also disable or re-enable Analytics:
```javascript
// to disable Analytics
Analytics.disable();

// to enable Analytics
Analytics.enable();
```

#### Update Endpoint

An endpoint uniquely identifies your app within Pinpoint. In order to update your <a href="https://docs.aws.amazon.com/pinpoint/latest/apireference/rest-api-endpoints.html" target="_blank">endpoint</a> use the `updateEndpoint()` method:

```javascript
import Analytics from '@aws-amplify/analytics';

Analytics.updateEndpoint({
    address: 'xxxxxxx', // The unique identifier for the recipient. For example, an address could be a device token, email address, or mobile phone number.
    attributes: {
        // Custom attributes that your app reports to Amazon Pinpoint. You can use these attributes as selection criteria when you create a segment.
        hobbies: ['piano', 'hiking'],
    },
    channelType: 'APNS', // The channel type. Valid values: APNS, GCM
    demographic: {
        appVersion: 'xxxxxxx', // The version of the application associated with the endpoint.
        locale: 'xxxxxx', // The endpoint locale in the following format: The ISO 639-1 alpha-2 code, followed by an underscore, followed by an ISO 3166-1 alpha-2 value
        make: 'xxxxxx', // The manufacturer of the endpoint device, such as Apple or Samsung.
        model: 'xxxxxx', // The model name or number of the endpoint device, such as iPhone.
        modelVersion: 'xxxxxx', // The model version of the endpoint device.
        platform: 'xxxxxx', // The platform of the endpoint device, such as iOS or Android.
        platformVersion: 'xxxxxx', // The platform version of the endpoint device.
        timezone: 'xxxxxx' // The timezone of the endpoint. Specified as a tz database value, such as Americas/Los_Angeles.
    },
    location: {
        city: 'xxxxxx', // The city where the endpoint is located.
        country: 'xxxxxx', // The two-letter code for the country or region of the endpoint. Specified as an ISO 3166-1 alpha-2 code, such as "US" for the United States.
        latitude: 0, // The latitude of the endpoint location, rounded to one decimal place.
        longitude: 0, // The longitude of the endpoint location, rounded to one decimal place.
        postalCode: 'xxxxxx', // The postal code or zip code of the endpoint.
        region: 'xxxxxx' // The region of the endpoint location. For example, in the United States, this corresponds to a state.
    },
    metrics: {
        // Custom metrics that your app reports to Amazon Pinpoint.
    },
    /** Indicates whether a user has opted out of receiving messages with one of the following values:
        * ALL - User has opted out of all messages.
        * NONE - Users has not opted out and receives all messages.
        */
    optOut: 'ALL',
    // Customized userId
    userId: 'XXXXXXXXXXXX',
    // User attributes
    userAttributes: {
        interests: ['football', 'basketball', 'AWS']
        // ...
    }
}).then(() => {
});
```

<a href="https://docs.aws.amazon.com/pinpoint/latest/developerguide/audience-define-user.html" target="_blank">Learn more</a> about Amazon Pinpoint and Endpoints.

#### API Reference

For a complete API reference visit the [API Reference](https://aws-amplify.github.io/amplify-js/api/classes/analyticsclass.html)
{: .callout .callout--info}

## Using Amazon Kinesis

The Amazon Kinesis analytics provider allows you to send analytics data to an [Amazon Kinesis](https://aws.amazon.com/kinesis) stream for real-time processing.

### Installation and Configuration

Register the *AWSKinesisProvider* with the Analytics category: 

```javascript
import { Analytics, AWSKinesisProvider } from 'aws-amplify';
Analytics.addPluggable(new AWSKinesisProvider());

```

If you did not use the CLI, ensure you have <a href="https://docs.aws.amazon.com/streams/latest/dev/learning-kinesis-module-one-iam.html" target="_blank">setup IAM permissions</a> for `PutRecords`.

Example IAM policy for Amazon Kinesis:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "kinesis:PutRecord",
                "kinesis:PutRecords"
            ],
            "Resource": "*"
        }
    ]
}
```

For more information visit [Amazon Kinesis Developer Documentation](https://docs.aws.amazon.com/streams/latest/dev/learning-kinesis-module-one-iam.html).

Configure Kinesis:

```javascript

// Configure the plugin after adding it to the Analytics module
Analytics.configure({
    AWSKinesis: {

        // OPTIONAL -  Amazon Kinesis service region
        region: 'XX-XXXX-X',
        
        // OPTIONAL - The buffer size for events in number of items.
        bufferSize: 1000,
        
        // OPTIONAL - The number of events to be deleted from the buffer when flushed.
        flushSize: 100,
        
        // OPTIONAL - The interval in milliseconds to perform a buffer check and flush if necessary.
        flushInterval: 5000, // 5s
        
        // OPTIONAL - The limit for failed recording retries.
        resendLimit: 5
    } 
});

```

### Working with the API

You can send a data to a Kinesis stream with the standard *record()* method:

```javascript
Analytics.record({
    data: { 
        // The data blob to put into the record
    },
    // OPTIONAL
    partitionKey: 'myPartitionKey', 
    streamName: 'myKinesisStream'
}, 'AWSKinesis');
```
## Using Amazon Personalize

Amazon Personalize can create recommendations by using event data, historical data,  or a combination of both.   
AWS Amplify includes an Amazon Personalize analytics provider that you can use to send event data to Amazon Personalize. The event data can then be used to create recommendations. 


To record event data, you need the following:

* [A dataset group](https://docs.aws.amazon.com/personalize/latest/dg/recording-events.html#event-dataset-group)
* [An event tracker](https://docs.aws.amazon.com/personalize/latest/dg/recording-events.html#event-get-tracker).

For more information, see [Record Events](https://docs.aws.amazon.com/personalize/latest/dg/recording-events.html).


### Installation and Configuration

Register the *AmazonPersonalizeProvider* with the Analytics category:
You need the tracking ID of your event tracker. For more information, see [Get a Tracking ID](https://docs.aws.amazon.com/personalize/latest/dg/recording-events.html#event-get-tracker).

```javascript
import { Analytics, AmazonPersonalizeProvider } from 'aws-amplify';
Analytics.addPluggable(new AmazonPersonalizeProvider());

```

Configure Amazon Personalize:

```javascript

// Configure the plugin after adding it to the Analytics module
Analytics.configure({
    AmazonPersonalize: {
    
        // REQUIRED - The trackingId to track the events 
        trackingId: '<TRACKING_ID>',
        
        // OPTIONAL -  Amazon Personalize service region
        region: 'XX-XXXX-X',

        // OPTIONAL - The number of events to be deleted from the buffer when flushed.
        flushSize: 10,

        // OPTIONAL - The interval in milliseconds to perform a buffer check and flush if necessary.
        flushInterval: 5000, // 5s
    }
});

```
### Working with the API

You can use the `Identify` event type to track a user identity. This lets you connect a user to their actions and record traits about them. To identify a user, specify a unique identifier for the userId property. 
 Consider the following user interactions when choosing when and how often to call record with the  Identify eventType:

* After a user registers.
* After a user logs in.
* When a user updates their information (For example, changing or adding or adding a new address).
* Upon loading any pages that are accessible by a logged in user (optional).

```javascript
Analytics.record({
        eventType: "Identify",
        properties: {
           "userId": "<USER_ID>"
        }
    }, 'AmazonPersonalize');
```
You can send events to Amazon personalize by calling the `record` operation. If you already use `Identify` tracking end-user data, you can skip the userId, the SDK will fetch the userId based on current browser session.
For information about the properties field, see [Put Events](https://docs.aws.amazon.com/personalize/latest/dg/API_UBS_PutEvents.html).

```javascript
Analytics.record({
        eventType: "<EVENT_TYPE>",
        userId: "<USER_ID>", (optional)
        properties: {
          "itemId": "<ITEM_ID>",
          "eventValue": "<EVENT_VALUE>"}
    },
    "AmazonPersonalize");
```
You can track iframe and HTML5 media types by using the MediaAutoTrack event type. MediaAutoTrack tracks all media events of the media DOM element that you bind to. `MediaAutoTracker` will automatically track *Play*, *Pause*, *Ended*, *TimeWatched*, and *Resume* in eventType. The duration of the event compared to the total length of the media is stored as a percentage value in eventValue.

```javascript
Analytics.record({
    eventType: "MediaAutoTrack",
    userId: "<USER_ID>", (optional)
    properties: {
        "domElementId": "MEDIA DOM ELEMENT ID",
        "itemId": "<ITEM_ID>"
    }
}, "AmazonPersonalize");
```

## Using Amazon Kinesis Firehose

The Amazon Kinesis Firehose analytics provider allows you to send analytics data to an [Amazon Kinesis Firehose](https://aws.amazon.com/kinesis/data-firehose) stream for reliably storing data.

### Installation and Configuration

Register the *AWSKinesisFirehoseProvider* with the Analytics category: 

```javascript
import { Analytics, AWSKinesisFirehoseProvider } from 'aws-amplify';
Analytics.addPluggable(new AWSKinesisFirehoseProvider());

```

Ensure you have <a href="https://docs.aws.amazon.com/firehose/latest/dev/controlling-access.html" target="_blank">setup IAM permissions</a> for `PutRecordBatch`.

Example IAM policy for Amazon Kinesis Firehose:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "firehose:PutRecord",
                "firehose:PutRecordBatch"
            ],
            "Resource": "*"
        }
    ]
}
```

Configure Kinesis Firehose:

```javascript

// Configure the plugin after adding it to the Analytics module
Analytics.configure({
    AWSKinesisFirehose: {

        // OPTIONAL -  Amazon Kinesis Firehose service region
        region: 'XX-XXXX-X',
        
        // OPTIONAL - The buffer size for events in number of items.
        bufferSize: 1000,
        
        // OPTIONAL - The number of events to be deleted from the buffer when flushed.
        flushSize: 100,
        
        // OPTIONAL - The interval in milliseconds to perform a buffer check and flush if necessary.
        flushInterval: 5000, // 5s
        
        // OPTIONAL - The limit for failed recording retries.
        resendLimit: 5
    } 
});

```

### Working with the API

You can send a data to a Kinesis Firehose stream with the standard *record()* method. Any data is acceptable and `streamName` is required:

```javascript
Analytics.record({
    data: { 
        // The data blob to put into the record
    },
    streamName: 'myKinesisStream'
}, 'AWSKinesisFirehose');
```

## Using a Custom Plugin

You can create your custom pluggable for Analytics. This may be helpful if you want to integrate your app with a custom analytics backend.

To create a plugin implement the `AnalyticsProvider` interface:

```typescript
import { Analytics, AnalyticsProvider } from 'aws-amplify';

export default class MyAnalyticsProvider implements AnalyticsProvider {
    // category and provider name
    static category = 'Analytics';
    static providerName = 'MyAnalytics';

    // you need to implement these four methods
    // configure your provider
    configure(config: object): object;

    // record events and returns true if succeeds
    record(params: object): Promise<boolean>;

    // return 'Analytics';
    getCategory(): string;

    // return the name of you provider
    getProviderName(): string;
}
```

You can now register your pluggable:

```javascript
// add the plugin
Analytics.addPluggable(new MyAnalyticsProvider());

// get the plugin
Analytics.getPluggable(MyAnalyticsProvider.providerName);

// remove the plugin
Analytics.removePluggable(MyAnalyticsProvider.providerName);

// send configuration into Amplify
Analytics.configure({
    MyAnalyticsProvider: { 
        // My Analytics provider configuration 
    }
});

```

The default provider (Amazon Pinpoint) is in use when you call `Analytics.record()` unless you specify a different provider: `Analytics.record({..},'MyAnalyticsProvider')`. 
{: .callout .callout--info}

## Using Modular Imports

You can import only specific categories into your app if you are only using specific features, analytics for example: `npm install @aws-amplify/analytics` which will only install the Analytics category. For working with AWS services you will also need to install and configure `@aws-amplify/auth`.

Import only Analytics:

```javascript
import Analytics from '@aws-amplify/analytics';

Analytics.configure();

```

## Using Analytics Auto Tracking

Analytics Auto Tracking helps you to automatically track user behaviors like sessions start/stop, page view change and web events like clicking, mouseover.

### Session Tracking

You can track the session both in a web app or a React Native app by using Analytics.
A web session can be defined in different ways. To keep it simple we define that the web session is active when the page is not hidden and inactive when the page is hidden. 
A session in the React Native app is active when the app is in the foreground and inactive when the app is in the background.

For example: 
```javascript
Analytics.autoTrack('session', {
    // REQUIRED, turn on/off the auto tracking
    enable: true,
    // OPTIONAL, the attributes of the event, you can either pass an object or a function 
    // which allows you to define dynamic attributes
    attributes: {
        attr: 'attr'
    },
    // when using function
    // attributes: () => {
    //    const attr = somewhere();
    //    return {
    //        myAttr: attr
    //    }
    // },
    // OPTIONAL, the service provider, by default is the AWS Pinpoint
    provider: 'AWSPinpoint'
});
```

When the page is loaded, the Analytics module will send an event with:
```javascript
{ 
    eventType: '_session_start', 
    attributes: { 
        attr: 'attr' 
    }
}
```
to the AWS Pinpoint Service. 

To keep backward compatibility, the auto tracking of the session is enabled by default. You can turn it off by:
```javascript
Analytics.configure({
    // OPTIONAL - Allow recording session events. Default is true.
    autoSessionRecord: false,
});
```
or 
```javascript
Analytics.autoTrack('session', {
    enable: false
});

// Note: this must be called before Amplify.configure() or Analytics.configure() to cancel the session_start event
```

### Page View Tracking

If you want to track which page/url in your webapp is the most frequently viewed one, you can use this feature. It will automatically send events containing url information when the page is visited.

To turn it on:
```javascript
Analytics.autoTrack('pageView', {
    // REQUIRED, turn on/off the auto tracking
    enable: true,
    // OPTIONAL, the event name, by default is 'pageView'
    eventName: 'pageView',
    // OPTIONAL, the attributes of the event, you can either pass an object or a function 
    // which allows you to define dynamic attributes
    attributes: {
        attr: 'attr'
    },
    // when using function
    // attributes: () => {
    //    const attr = somewhere();
    //    return {
    //        myAttr: attr
    //    }
    // },
    // OPTIONAL, by default is 'multiPageApp'
    // you need to change it to 'SPA' if your app is a single-page app like React
    type: 'multiPageApp',
    // OPTIONAL, the service provider, by default is the AWS Pinpoint
    provider: 'AWSPinpoint',
    // OPTIONAL, to get the current page url
    getUrl: () => {
        // the default function
        return window.location.origin + window.location.pathname;
    }
});
```
Note: This is not supported in React Native.

### Page Event Tracking

If you want to track user interactions with elements on the page, you can use this feature. All you need to do is attach the specified selectors to your dom element and turn on the auto tracking.

To turn it on:
```javascript
Analytics.autoTrack('event', {
    // REQUIRED, turn on/off the auto tracking
    enable: true,
    // OPTIONAL, events you want to track, by default is 'click'
    events: ['click'],
    // OPTIONAL, the prefix of the selectors, by default is 'data-amplify-analytics-'
    // in order to avoid collision with the user agent, according to https://www.w3schools.com/tags/att_global_data.asp
    // always put 'data' as the first prefix
    selectorPrefix: 'data-amplify-analytics-',
    // OPTIONAL, the service provider, by default is the AWS Pinpoint
    provider: 'AWSPinpoint',
    // OPTIONAL, the default attributes of the event, you can either pass an object or a function 
    // which allows you to define dynamic attributes
    attributes: {
        attr: 'attr'
    }
    // when using function
    // attributes: () => {
    //    const attr = somewhere();
    //    return {
    //        myAttr: attr
    //    }
    // }
});
```

For example:
```html
<!-- you want to track this button and send an event when it is clicked -->
<button
    data-amplify-analytics-on='click'
    data-amplify-analytics-name='click'
    data-amplify-analytics-attrs='attr1:attr1_value,attr2:attr2_value'
/>
```
When the button above is clicked, an event will be sent automatically and this is equivalent to do:
```html
<script>
    var sendEvent = function() {
        Analytics.record({
            name: 'click',
            attributes: {
                attr: 'attr', // the default ones
                attr1: attr1_value, // defined in the button component
                attr2: attr2_value, // defined in the button component
            }
        });
    }
</script>
<button onclick="sendEvent()"/>
```
