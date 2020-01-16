---
title: Getting Started
---
# Getting Started

Build an Android app using the Amplify Framework which contains:

- CLI toolchain for creating and managing your serverless backend.
- Android, iOS, and JavaScript libraries to access your resources using a category based programming model.
- Framework-specific UI component libraries for React, React Native, Angular, Ionic and Vue.

This page guides you through setting up a backend and integration into your Android app. You will create a "Todo app" with a GraphQL API to store and retrieve items in a cloud database, as well as receive updates over a realtime subscription.

[GraphQL](http://graphql.org){:target="_blank"} is a data language that was developed to enable apps to fetch data from APIs. It has a declarative, self-documenting style. In a GraphQL operation, the client specifies how to structure the data when it is returned by the server. This makes it possible for the client to query only for the data it needs, in the format that it needs it in.

## Prerequisites

* [Install and configure the Amplify CLI](..)

* [Install Android Studio](https://developer.android.com/studio/index.html#downloads) version 3.1 or higher. 

* [Install Android SDK for API level 28 (Android 9.0).](https://developer.android.com/studio/releases/platforms)

* This guide assumes that you are familiar with Android development and tools. If you are new to Android development, you can follow [these steps](https://developer.android.com/training/basics/firstapp/creating-project){:target="_blank"} to create your first Android application using Java. 


## Step 1: Configure your app

You can use an existing Android app or create a new Android app using Java as per the steps in prerequisite section.

Modify your `project/build.gradle` with the following build dependency:

```groovy
classpath 'com.amazonaws:aws-android-sdk-appsync-gradle-plugin:2.9.+'
```

Next, add dependencies to your `app/build.gradle`, and then choose Sync Now on the upper-right side of Android Studio.

```groovy
apply plugin: 'com.amazonaws.appsync'

dependencies {
    //Base SDK
    implementation 'com.amazonaws:aws-android-sdk-core:2.15.+'
    //AppSync SDK
    implementation 'com.amazonaws:aws-android-sdk-appsync:2.8.+'
    implementation 'org.eclipse.paho:org.eclipse.paho.client.mqttv3:1.2.0'
    implementation 'org.eclipse.paho:org.eclipse.paho.android.service:1.1.1'
}
```

Finally, update your AndroidManifest.xml with the following:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE"/>
<uses-permission android:name="android.permission.WAKE_LOCK" />
<uses-permission android:name="android.permission.READ_PHONE_STATE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE"/>
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE"/>

        <!--other code-->

    <application
        android:allowBackup="true"
        android:icon="@mipmap/ic_launcher"
        android:label="@string/app_name"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/AppTheme">

        <service android:name="org.eclipse.paho.android.service.MqttService" />

        <!--other code-->
    </application>
```

**Build** your Android Studio project.

## Step 2: Initialize your project

In a terminal window, navigate to your project folder (the folder that typically contains your project level `build.gradle`) and run the following command (for this app, accepting all defaults is OK):

```bash
$ cd ./YOUR_PROJECT_FOLDER
$ amplify init        #accept defaults
```
An `awsconfiguration.json` file will be created with your configuration and updated as features get added to your project by the Amplify CLI. The file is placed in the `./app/src/main/res/raw` directory of your Android Studio project and automatically used by the SDKs at runtime.

**What is awsconfiguration.json?**

Rather than configuring each service through a constructor or constants file, the AWS SDKs for Android support configuration through a centralized file called `awsconfiguration.json` which defines all the regions and service endpoints to communicate. Whenever you run `amplify push`, this file is automatically created allowing you to focus on your application code. On Android projects, the `awsconfiguration.json` will be placed into the `./app/src/main/res/raw` directory.

## Step 3: Add API and Database

Add a GraphQL API to your app and automatically provision a database with the following command (accepting all defaults is OK):

```bash
$ amplify add api     #select GraphQL, API Key
```

The `add api` flow  will ask you simple questions. If this is your first time using the CLI select **No** for the question "Do you have an annotated GraphQL schema". The CLI then guides you through the default project **"Single object with fields (e.g., “Todo” with ID, name, description)"** as it will be used in the code generation examples below. You can always change the schema as needed. This process creates an AWS AppSync API and connects it to an Amazon DynamoDB database. The CLI flow will look like below:

```bash
$ amplify add api
? Please select from one of the below mentioned services: GraphQL
? Provide API name: todo
? Choose the default authorization type for the API: API key
? Enter a description for the API key: ToDo description
? After how many days from now the API key should expire (1-365): 180
? Do you want to configure advanced settings for the GraphQL API: No, I am done.
? Do you have an annotated GraphQL schema? No
? Do you want a guided schema creation? Yes
? What best describes your project: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? No
```

## Step 4: Push changes

Create required backend resources for your configured API with the following command:

```bash
$ amplify push
```

Since you added an API the `amplify push` process will automatically enter the codegen process and prompt you for configuration. Accept the defaults which generate a `./app/src/main/graphql` folder structure with your statements.

Run a **Gradle Sync** and **Build** your app. The generated packages are automatically added to your project.

## Step 5: Integrate into your app

Initialize the AppSync client inside your application code, such as the `onCreate()` lifecycle method of your activity class:

```java
private AWSAppSyncClient mAWSAppSyncClient;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_main);
    mAWSAppSyncClient = AWSAppSyncClient.builder()
        .context(getApplicationContext())
        .awsConfiguration(new AWSConfiguration(getApplicationContext()))
        .build();
}
```

You can now add data to your database with a mutation:

```java
public void runMutation(){
    CreateTodoInput createTodoInput = CreateTodoInput.builder().
        name("Use AppSync").
        description("Realtime and Offline").
        build();

    mAWSAppSyncClient.mutate(CreateTodoMutation.builder().input(createTodoInput).build())
        .enqueue(mutationCallback);
}

private GraphQLCall.Callback<CreateTodoMutation.Data> mutationCallback = new GraphQLCall.Callback<CreateTodoMutation.Data>() {
    @Override
    public void onResponse(@Nonnull Response<CreateTodoMutation.Data> response) {
        Log.i("Results", "Added Todo");
    }

    @Override
    public void onFailure(@Nonnull ApolloException e) {
        Log.e("Error", e.toString());
    }
};
```

Next, query the data:

```java
public void runQuery(){
    mAWSAppSyncClient.query(ListTodosQuery.builder().build())
        .responseFetcher(AppSyncResponseFetchers.CACHE_AND_NETWORK)
        .enqueue(todosCallback);
}

private GraphQLCall.Callback<ListTodosQuery.Data> todosCallback = new GraphQLCall.Callback<ListTodosQuery.Data>() {
    @Override
    public void onResponse(@Nonnull Response<ListTodosQuery.Data> response) {
        Log.i("Results", response.data().listTodos().items().toString());
    }

    @Override
    public void onFailure(@Nonnull ApolloException e) {
        Log.e("ERROR", e.toString());
    }
};
```

You can also setup realtime subscriptions to data:

```java
private AppSyncSubscriptionCall subscriptionWatcher;

private void subscribe(){
    OnCreateTodoSubscription subscription = OnCreateTodoSubscription.builder().build();
    subscriptionWatcher = mAWSAppSyncClient.subscribe(subscription);
    subscriptionWatcher.execute(subCallback);
}

private AppSyncSubscriptionCall.Callback subCallback = new AppSyncSubscriptionCall.Callback() {
    @Override
    public void onResponse(@Nonnull Response response) {
        Log.i("Response", response.data().toString());
    }

    @Override
    public void onFailure(@Nonnull ApolloException e) {
        Log.e("Error", e.toString());
    }

    @Override
    public void onCompleted() {
        Log.i("Completed", "Subscription completed");
    }
};
```

Call the `runMutation()`, `runQuery()`, and `subscribe()` methods from your app code, such as from a button click or when your app starts in `onCreate()`. You will see data being stored and retrieved in your backend from the Android Studio console. At any time you can open the AWS console for your new API directly by running the following command:

```terminal
$ amplify console api
> GraphQL               ##Select GraphQL
```

This will open the AWS AppSync console for you to run Queries, Mutations, or Subscriptions at the server and see the changes in your client app.

## Next Steps

🎉 Congratulations! Your app is built, with a realtime backend using AWS AppSync.

What next? Here are some things to add to your app:


* [Authentication](./authentication)
* [Storage](./storage)
* [Serverless APIs](./api)
* [Analytics](./analytics)
* [Push Notification](./push-notifications)
* [Messaging](./messaging)

**Existing AWS Resources**

If you want to use your existing AWS resources with your app you will need to **manually configure** your app with an `awsconfiguration.json` file in your code. For example, if you were using Amazon Cognito Identity, Cognito User Pools, AWS AppSync, or Amazon S3:

```xml
{
    "CredentialsProvider": {
        "CognitoIdentity": {
            "Default": {
                "PoolId": "XX-XXXX-X:XXXXXXXX-XXXX-1234-abcd-1234567890ab",
                "Region": "XX-XXXX-X"
            }
        }
    },
    "CognitoUserPool": {
        "Default": {
            "PoolId": "XX-XXXX-X_abcd1234",
            "AppClientId": "XXXXXXXX",
            "AppClientSecret": "XXXXXXXXX",
            "Region": "XX-XXXX-X"
        }
    },
    "AppSync": {
        "Default": {
            "ApiUrl": "https://XXXXXX.appsync-api.XX-XXXX-X.amazonaws.com/graphql",
            "Region": "XX-XXXX-X",
            "AuthMode": "AMAZON_COGNITO_USER_POOLS"
        }
    },
    "S3TransferUtility": {
        "Default": {
            "Bucket": "BUCKET_NAME",
            "Region": "XX-XXXX-X"
        }
    }
}
```

In the configuration above, you would need to set the appropriate values such as `Region`, `Bucket`, etc.

**AWS SDK Interfaces**

For working with other AWS services you can use service interface objects directly via the generated SDK clients. 

To work with service interface objects, your Amazon Cognito users' [IAM role](https://docs.aws.amazon.com/cognito/latest/developerguide/iam-roles.html) must have the appropriate permissions to call the requested services.
{: .callout .callout--warning}

You can call methods on any AWS Service interface object supported by the AWS Android SDK by passing your credentials from the AWSMobileClient to the service call constructor. See [SDK Setup Options](./manualsetup) for more information.
