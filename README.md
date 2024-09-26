# dfinity-candid-lite
This is the lighter version of the [ic4j-agent](https://github.com/ic4j/ic4j-agent)

From ic4j-agent version [0.7.4](https://repo1.maven.org/maven2/org/ic4j/ic4j-agent/0.7.4/), mobile users can now start building Android applications on top of Internet Computer (ICP) using the Java SDK. The libraries are optimized for Android performance, enabling direct communication with canisters using ic4j-candid.

This README provides step-by-step instructions to build an Android application that interacts with ICP using ic4j-agent and ic4j-candid.
For a Demo Application, visit [here](https://github.com/nikhil5642/ICP-TweetApp).

---

## Table of Contents
- [Introduction](#introduction)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Key Components](#key-components)
- [How to Build an Android App on ICP](#how-to-build-an-android-app-on-icp)
  - [Step 1: Set up the Android Project](#step-1-set-up-the-android-project)
  - [Step 2: Add Dependencies](#step-2-add-dependencies)
  - [Step 3: Create an ICP Client](#step-3-create-an-icp-client)
  - [Step 4: Implement Canister Communication](#step-4-implement-canister-communication)
- [Example Usage](#example-usage)

---

## Introduction

With the release of **ic4j-agent 0.7.4**, the **Internet Computer (ICP)** is now accessible to Android app developers through a streamlined and optimized Java SDK. **ic4j-candid** is the core library that enables serialization and deserialization of Candid data types, essential for communication between Android apps and ICP canisters.

This lightweight library allows Android developers to:
- Query canisters on the ICP.
- Send update requests (e.g., posting data).
- Use Candid serialization efficiently within Android's performance constraints.

---

## Prerequisites

- **Android Studio** 4.x or later
- **Java 8+** (or Kotlin)
- **Gradle** (for dependency management)
- Basic knowledge of **ICP**, **ic4j-candid**, and **Candid data serialization**
  
---

## Installation

Add the following dependencies to your Android project’s **`build.gradle`** file to include **ic4j-agent** and **ic4j-candid** libraries.

```gradle
dependencies {
    implementation 'org.ic4j:ic4j-agent:0.7.4'  // Core Agent library
    implementation 'org.ic4j:ic4j-candid:0.7.4' // Candid serialization
    implementation 'org.slf4j:slf4j-android:1.7.36' // Logging library designed for Android
    implementation 'com.squareup.okhttp3:okhttp:4.12.0' //Networking
}
```

---

## Key Components

- **`ic4j-agent`**: Manages communication with ICP canisters. Handles the transport layer (HTTP, OkHttp).
- **`ic4j-candid`**: Provides Candid serialization and deserialization of data types (used to structure requests and responses from ICP canisters).
- **Anonymous Identity**: Used to interact with canisters that do not require authentication.

---

## How to Build an Android App on ICP

### Step 1: Set up the Android Project

1. **Create a new Android project** in Android Studio or open your existing project.
2. **Sync Gradle** to ensure that dependencies are installed correctly.

### Step 2: Add Dependencies

In the **app-level `build.gradle`** file, add the required dependencies for **ic4j-agent** and **ic4j-candid**.

```gradle
dependencies {
    implementation 'org.ic4j:ic4j-agent:0.7.4'
    implementation 'org.ic4j:ic4j-candid:0.7.4'
    implementation 'org.slf4j:slf4j-android:1.7.36' 
    implementation 'com.squareup.okhttp3:okhttp:4.12.0' 
}
```

### Step 3: Create an ICP Client

Create a singleton class **`ICClient`** to handle communication with the ICP canister. This class initializes the **ic4j-agent** using **AnonymousIdentity** and manages the canister interaction.

```java
import org.ic4j.agent.Agent;
import org.ic4j.agent.AgentBuilder;
import org.ic4j.agent.ProxyBuilder;
import org.ic4j.agent.identity.AnonymousIdentity;
import org.ic4j.agent.http.ReplicaOkHttpTransport;
import org.ic4j.types.Principal;

public class ICClient {
    private static ICClient instance;
    private final TweetCanisterService tweetService;

    private ICClient(String canisterId, String icEndpoint) throws Exception {
        // Initialize the agent with anonymous identity
        Agent agent = new AgentBuilder()
            .identity(new AnonymousIdentity())
            .transport(ReplicaOkHttpTransport.create(icEndpoint))
            .build();

        // Create proxy for TweetCanisterService
        tweetService = ProxyBuilder.create(agent, Principal.fromString(canisterId))
            .getProxy(TweetCanisterService.class);
    }

    public static ICClient getInstance(Context context) throws Exception {
        if (instance == null) {
            instance = new ICClient(
                context.getString(R.string.canister_id),
                context.getString(R.string.ic_endpoint));
        }
        return instance;
    }

    public TweetCanisterService getTweetService() {
        return tweetService;
    }
}
```

### Step 4: Implement Canister Communication

Define the interface **`TweetCanisterService`**, which will be used to send queries and updates to the canister. The `@QUERY` annotation is used for read-only operations, while the `@UPDATE` annotation is used for making updates to the canister.

```java
import org.ic4j.agent.annotations.QUERY;
import org.ic4j.agent.annotations.UPDATE;

public interface TweetCanisterService {

    @QUERY
    TweetModel[] getTweets();

    @UPDATE
    Long postTweet(TweetModel tweetModel);
}
```

This interface is used to send and retrieve data from the canister. The **`ic4j-candid`** library handles serialization and deserialization of the data to and from the canister.

---

## Example Usage

### Posting a Tweet

Here’s an example of how to post a tweet from an Android fragment:

```kotlin
val tweetModel = TweetModel(
    id = 0,  // ID is auto-generated on the canister
    creatorID = "User123",
    applicationID = "Principal-ID",
    created = System.currentTimeMillis(),
    content = "This is my first tweet!"
)

val tweetService = ICClient.getInstance(context).getTweetService()
val tweetId = tweetService.postTweet(tweetModel)
```

### Fetching Tweets

You can fetch the list of tweets from the canister as follows:

```kotlin
val tweetService = ICClient.getInstance(context).getTweetService()
val tweets = tweetService.getTweets()
recyclerView.adapter = TweetsAdapter(tweets)
```
