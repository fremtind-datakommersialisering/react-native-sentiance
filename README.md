
# react-native-sentiance

## Status
*`react-native-sentiance` is still in an early development stage and not ready for production use just yet. Please try it out, give feedback, and help fix bugs.


## Getting started

`$ npm install react-native-sentiance --save`

### Manual installation


#### iOS

1. In XCode, in the project navigator, right click `Libraries` ➜ `Add Files to [your project's name]`
2. Go to `node_modules` ➜ `react-native-sentiance-library` and add `RNSentianceLibrary.xcodeproj`
3. In XCode, in the project navigator, select `RNSentianceLibrary.xcodeproj`. Add the folder where `SENTTransportDetectionSDK.framework` is located to `Search Paths` ➜ `Framework Search Paths`
4. In XCode, in the project navigator, select your project. Add `libRNSentianceLibrary.a` to your project's `Build Phases` ➜ `Link Binary With Libraries`
5. Run your project (`Cmd+R`)<


__via Cocoapods__
1. Add `RNSentiance` Pod to your Podfile
	```
	pod 'RNSentiance', :path => '../node_modules/react-native-sentiance/ios/RNSentiance.podspec'
	```
2. Add `SENTSDK` Pod to your Podfile
	```
	pod 'SENTSDK', :podspec => '../node_modules/react-native-sentiance/ios/SENTSDK.podspec'
	```
3. Run `pod install` in your `ios` folder


#### Android

1. Open up `android/app/src/main/java/[...]/MainActivity.java`
  - Add `import com.reactlibrary.RNSentiancePackage;` to the imports at the top of the file
  - Add `new RNSentiancePackage()` to the list returned by the `getPackages()` method
2. Append the following lines to `android/settings.gradle`:
  	```
  	include ':react-native-sentiance'
  	project(':react-native-sentiance').projectDir = new File(rootProject.projectDir, 	'../node_modules/react-native-sentiance/android')
  	```
3. Insert the following lines inside the dependencies block in `android/app/build.gradle`:
  	```
      compile project(':react-native-sentiance')
  	```


__Android notification icon__

The SDK may need to start a foreground service every now and again. `RNSentiance` will therefore pass a notification that can be used by the service.

Creating a small icon with the name `notification_icon` located at `android/src/main/res/mipmap-[...]` is required in order for the notification to be successfully created. Android 5.0+ enforces your icon to only be white and transparent.


## Usage
```javascript
import RNSentiance from 'react-native-sentiance';
```

#### Initializing the Sentiance SDK
Initialization is a very important step; before initialization, almost none of the methods on the Sentiance SDK interface are allowed to be called (with the exception of `init`, `isInitialized` and `getVersion`).
```javascript
try {
	const initResponse = await RNSentiance.init('APPID', 'SECRET');
	// SDK init has successfully initialized
} catch (err) {
	// SDK init has failed initializing
}
```

_NOTE: Ideally, initializing the SDK is done from `Application.onCreate` as this will guarantee that the SDK is running as often as possible. If your application uses a login flow, you will want to start the SDK only if the user is logged in, at that point you could start the SDK through JavaScript. Once the user is logged in, the SDK should always start before the end of `onCreate`. Please refer to https://developers.sentiance.com/docs/sdk/android/integration for documentation on the Android SDK integration._


#### Starting the Sentiance SDK
Starting is only allowed after successful initialization. Resolves with an SDK status object.
```javascript
try {
	const startResponse = await RNSentiance.start();
	const { startStatus } = startResponse;
	if (startStatus === 'STARTED') {
		// SDK started properly.
	} else if (startStatus === 'PENDING') {
		// Something prevented the SDK to start properly. Once fixed, the SDK will start automatically.
	}
} catch (err) {
	// SDK did not start.
}
```

#### Stopping the Sentiance SDK
Stopping is only allowed after successful initialization. While it's possible to "pause" the detections modules of the Sentiance SDK's, it's not recommended.
```javascript
try {
	const stopResponse = await RNSentiance.stop();
	// SDK stopped properly.
} catch(err) {
	// An error prevented the SDK from stopping correctly
}
```

#### Init status
Checking if SDK is initialized
```javascript
const isInitialized = await RNSentiance.isInitialized();
```

#### SDK status
The status of the Sentiance SDK
```javascript
const sdkStatus = await RNSentiance.getSdkStatus();
```

The SDK can signal SDK status updates to JavaScript without being invoked directly. You can subscribe to these status updates by creating a new NativeEventEmitter instance around your module, and adding a listener for `SDKStatusUpdate`.
```javascript
import { NativeEventEmitter } from 'react-native'

const sentianceEmitter = new NativeEventEmitter(RNSentiance);
const subscription = sentianceEmitter.addListener(
	'SDKStatusUpdate',
	res => {
		// Returns SDK status
	}
);

// Don't forget to unsubscribe, typically in componentWillUnmount
subscription.remove();
```

#### Get SDK version
```javascript
const version = await RNSentiance.getVersion();
```

#### Get user id
If the SDK is initialized, you can get the user id as follows. This user id will allow you to interact with the API's from Sentiance. You need a token and user to authorize requests and query the right data.
```javascript
const userId = await RNSentiance.getUserId();
```

#### Get user access token
If the SDK is initialized, you can get a user access token as follows. This token will allow you to interact with the API's from Sentiance. You need a token and user to authorize requests and query the right data. If the token has expired, or will expire soon, the SDK will get a new bearer token before passing it to the callback. Generally, this operation will complete instantly by returning a cached bearer token, but if a new token has to be obtained from the Sentiance API, there is a possibility it will fail.
```javascript
const { tokenId } = await RNSentiance.getUserAccessToken();
```

#### Adding custom metadata
Custom metadata allows you to store text-based key-value user properties into the Sentiance platform.
Examples are custom user id's, application related properties you need after the processing, ...
```javascript
const label = 'correlation_id'
const value = '3a5276ec-b2b2-4636-b893-eb9a9f014938'

await RNSentiance.addUserMetadataField(label, value);
```

#### Remove custom metadata
You can remove previously added metadata fields by passing the metadata label to the removeUserMetadataField function.
```javascript
const label = 'correlation_id'

await RNSentiance.removeUserMetadataField(label);
```

#### Adding multiple custom metadata fields
You can add multiple custom metadata fields by passing an object to the addUserMetadataFields function.
```javascript
const metadata = { corrolation_id: '3a5276ec-b2b2-4636-b893-eb9a9f014938' }

await RNSentiance.addUserMetadataFields(metadata);
```

#### Starting trip
Whenever you call startTrip on the SDK, you override moving state detection and the SDK will track the trip until you call stopTrip or until the timeout (2 hours) is reached. `startTrip` accepts a metadata object and a transport mode hint (`number`) as parameters.

Transport mode hint:
```
SENTTransportModeUnknown = 1,
SENTTransportModeCar = 2,
SENTTransportModeBicycle = 3,
SENTTransportModeOnFoot = 4,
SENTTransportModeTrain = 5,
SENTTransportModeTram = 6,
SENTTransportModeBus = 7,
SENTTransportModePlane = 8,
SENTTransportModeBoat = 9,
SENTTransportModeMetro = 10,
SENTTransportModeRunning = 11
```

Example:
```javascript
const metadata = { corrolation_id: '3a5276ec-b2b2-4636-b893-eb9a9f014938' }
const transportModeHint = 1
try {
	await RNSentiance.startTrip(metadata, transportModeHint);
	// Trip is started
} catch (err) {
	// Unable to start trip
}
```

#### Stopping trip
```javascript
try {
	const trip = await RNSentiance.stopTrip();
	// Stopped trip
} catch (err) {
	// Unable to stop trip
}
```

The SDK can also signal trip timeouts to JavaScript. You can subscribe to these trip timeouts by creating a new NativeEventEmitter instance around your module, and adding a listener for `TripTimeout`.
```javascript
import { NativeEventEmitter } from 'react-native'

const sentianceEmitter = new NativeEventEmitter(RNSentianceLibrary)
const subscription = sentianceEmitter.addListener(
	'TripTimeout',
	() => {
		// Trip timeout received
	}
)
```

#### Trip status
Checking trip status
```javascript
const isTripOngoing = await RNSentiance.isTripOngoing();
```

#### Control sending data
If you want to override the default behavior, you can initiate a force submission of detections. Ideally, you use this method only after explaining to the user that your app will consume more bandwidth in case the device is not connected to Wi-Fi.

```javascript
try {
	await RNSentiance.submitDetections();
} catch (err) {
	// Something went wrong with submitting data, for more information, see the error variable
}
```

#### Disk, mobile network and Wi-Fi quotas
The actual usages and limits in bytes can be obtained using the getWiFiQuotaUsage, getWiFiQuotaLimit and similar methods on the Sentiance SDK interface.

```javascript
const limit = await RNSentiance.getWiFiQuotaLimit();
```

All quota functions:

* `getWiFiQuotaLimit`
* `getWiFiQuotaUsage`
* `getMobileQuotaLimit`
* `getMobileQuotaUsage`
* `getDiskQuotaLimit`
* `getDiskQuotaUsage`
