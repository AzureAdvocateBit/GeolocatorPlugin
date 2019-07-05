# Getting Started

## Setup
* NuGet: [Xam.Plugin.Geolocator](http://www.nuget.org/packages/Xam.Plugin.Geolocator) [![NuGet](https://img.shields.io/nuget/v/Xam.Plugin.Geolocator.svg?label=NuGet)](https://www.nuget.org/packages/Xam.Plugin.Geolocator/)
* `PM> Install-Package Xam.Plugin.Geolocator`
* Install into ALL of your projects, include client projects.
* namespace: `using Plugin.Geolocator;`


## Using Geolocator APIs
It is drop dead simple to gain access to the Geolocator APIs in any project. All you need to do is get a reference to the current instance of IGeolocator via `CrossGeolocator.Current`:

```csharp
public bool IsLocationAvailable()
{
    return CrossGeolocator.Current.IsGeolocationAvailable;
}
```

There may be instances where you install a plugin into a platform that it isn't supported yet. This means you will have access to the interface, but no implementation exists. You can make a simple check before calling any API to see if it is supported on the platform where the code is running. This is nifty when unit testing:

```csharp
public bool IsLocationAvailable()
{
    if(!CrossGeolocator.IsSupported)
        return false;

    return CrossGeolocator.Current.IsGeolocationAvailable;
}
```



## Permissions & Additional Setup Considerations
Before making any calls to the geolocator that requires the permissions, you should consider checking that the user has granted proper permission. The geolocator plugin will attempt to ask for permission, but it is not guaranteed.

### Android:

This plugin uses the [Current Activity Plugin](https://github.com/jamesmontemagno/CurrentActivityPlugin/blob/master/README.md) to get access to the current Android Activity. Be sure to complete the full setup if a MainApplication.cs file was not automatically added to your application. Please fully read through the [Current Activity Plugin Documentation](https://github.com/jamesmontemagno/CurrentActivityPlugin/blob/master/README.md). At an absolute minimum you must set the following in your Activity's OnCreate method:

```csharp
CrossCurrentActivity.Current.Init(this, bundle);
```

It is highly recommended that you use a custom Application that are outlined in the Current Activity Plugin Documentation](https://github.com/jamesmontemagno/CurrentActivityPlugin/blob/master/README.md)

#### Permissions:
The `ACCESS_COARSE_LOCATION` and `ACCESS_FINE_LOCATION` permissions are required and are automatically added to your Android Manifest when you compile. No need to add them manually!

By adding these permissions [Google Play will automatically filter out devices](http://developer.android.com/guide/topics/manifest/uses-feature-element.html#permissions-features) without specific hardware. You can get around this by adding the following to your AssemblyInfo.cs file in your Android project:

```csharp
[assembly: UsesFeature("android.hardware.location", Required = false)]
[assembly: UsesFeature("android.hardware.location.gps", Required = false)]
[assembly: UsesFeature("android.hardware.location.network", Required = false)]
```

This plugin leverages the [Permission Plugin](http://github.com/jamesmontemagno/permissionsplugin), which means you must add the following code to your BaseActivity or MainActivity in Xamarin.Forms:

Add in Activity:
```csharp
public override void OnRequestPermissionsResult(int requestCode, string[] permissions, Android.Content.PM.Permission[] grantResults)
{
    Plugin.Permissions.PermissionsImplementation.Current.OnRequestPermissionsResult(requestCode, permissions, grantResults);
	base.OnRequestPermissionsResult(requestCode, permissions, grantResults);
}
```

### iOS/tvOS/macOS
Your app is required to have keys in your Info.plist for `NSLocationWhenInUseUsageDescription` in order to access the device's location. 

Simply Add:
```xml
<key>NSLocationWhenInUseUsageDescription</key>
<string>This app needs access to location when open.</string>
```

Ensure the `CrossGeolocator.Current` instance is first accessed on the main thread to prevent any deadlocks when first accessing it on a background thread. If you're unsure about the first access simply call `CrossGeolocator.Current` in your `AppDelegate`.

#### Background Updates
Only implement this and add these properites if you need background updates for your application. Most likely you will not. Adding this also has direct impact on permissions and prompts to the user. Please be very careful when adding this information.

Inside of your info.plist you must enable **Background Modes/UIBackgroundModes** for location updates. Here is a [full guide](https://developer.xamarin.com/guides/ios/application_fundamentals/backgrounding/ios_backgrounding_walkthroughs/location_walkthrough/). Your info.plist should contain something like this:

```
<key>UIBackgroundModes</key>
<array>
	<string>location</string>
</array>
```

In addition to `NSLocationWhenInUseUsageDescription` you are required to add `NSLocationAlwaysAndWhenInUseUsageDescription` keys in your app's Info.plist file. (If your app supports iOS 10 and earlier, the `NSLocationAlwaysUsageDescription` key is also required.) If those keys are not present, authorization requests fail immediately.

```xml
<key>NSLocationAlwaysUsageDescription</key>
<string>This app needs access to location when in the background.</string>
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>This app needs access to location when open and in the background.</string>
```

If you are targeting devices running older than iOS 8 you must add **NSLocationUsageDescription**.

If you want the dialogs to be translated you must support the specific languages in your app. Read the [iOS Localization Guide](https://developer.xamarin.com/guides/ios/advanced_topics/localization_and_internationalization/)

Be sure to read the [Background Updates](BackgroundUpdates.md) section for additional instructions.

**When using background updates on iOS it is highly recommended that you manage your own permissions inside of iOS. Due to changes in iOS 11 with Always and When In Use deadlocks may happen if you are not careful and have the correct permissions. It is recommended to create a location manager and monitor your own changes for correct permissions**


Please see the [Apple Documentation](https://devstreaming-cdn.apple.com/videos/wwdc/2017/713tkef4yl0sv3k/713/713_whats_new_in_location_technologies.pdf)

### UWP
You must set the `ID_CAP_LOCATION` permission.





<= Back to [Table of Contents](README.md)
