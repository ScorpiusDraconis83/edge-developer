---
title: Capabilities and EdgeOptions
description: A reference for WebDriver capabilities and Microsoft Edge-specific options supported by EdgeDriver.
author: MSEdgeTeam
ms.author: msedgedevrel
ms.topic: conceptual
ms.service: microsoft-edge
ms.date: 06/09/2022
---
# Capabilities and EdgeOptions

Capabilities are options that you can use to customize and configure an `EdgeDriver` session.  To learn about starting a new `EdgeDriver` session, see [Automating Microsoft Edge](index.md#automate-microsoft-edge).  This article describes all supported capabilities for Microsoft Edge and provides details about passing the capabilities to `EdgeDriver` sessions.

Capabilities are passed to a WebDriver session as a JSON map, but it is not necessary or recommended to set them this way.  WebDriver testing frameworks (such as [Selenium](https://www.selenium.dev)) provide language bindings which typically have convenience methods so you don't need to configure the JSON map yourself.  For example, Selenium configures capabilities through the `EdgeOptions` class.


To learn more about how to configure capabilities, consult the documentation for your preferred WebDriver testing framework.  For more information, see [Choose a WebDriver testing framework](index.md#choose-a-webdriver-testing-framework).


<!-- ====================================================================== -->
## Using the EdgeOptions class

Create an instance of `EdgeOptions`, which provides convenience methods to set Microsoft Edge-specific capabilities.  After you configure the `EdgeOptions` object, pass `EdgeOptions` into the `EdgeDriver` constructor.

```csharp
var options = new EdgeOptions();
options.AddExtensions("/path/to/extension.crx");
var driver = new EdgeDriver(options);
```

To use capabilities that don't have an associated convenience method, use the `AddAdditionalEdgeOption` method.  You must pass the full name of the capability and a value with the correct type.  For the full list of accepted capabilities and value types, see [EdgeOptions object](#edgeoptions-object).

```csharp
options.AddAdditionalEdgeOption("wdpAddress", "remotehost:50080");
```


<!-- ====================================================================== -->
## Recognized Capabilities

For standard capabilities that `EdgeDriver` accepts, see [Selenium documentation](https://www.selenium.dev/documentation/en/driver_idiosyncrasies/shared_capabilities/) and the [W3C WebDriver standard](https://www.w3.org/TR/webdriver#capabilities).  This article only lists capabilities specific to Microsoft Edge.


<!-- ====================================================================== -->
## EdgeOptions object

Most Microsoft Edge-specific capabilities are exposed through the `EdgeOptions` object.  In some languages, the capabilities are implemented by the `EdgeOptions` class.  In other languages, the capabilities are stored under the `ms:edgeOptions` dictionary in `DesiredCapabilities`.

| Capability | Type | Details |
|:--- |:--- |:--- |
| `args` | list of strings | List of command-line arguments to pass to the Microsoft Edge process on launch. Arguments with an associated value should be separated by an `=` sign (for example, `['start-maximized', 'user-data-dir=/tmp/temp_profile']`). If you're launching a WebView2 app, then these arguments are passed to your app instead of the underlying Microsoft Edge browser process. To pass arguments to the browser process when launching a WebView2 app, use [webviewOptions.additionalBrowserArguments](#webviewoptions-object) instead. |
| `binary` | string | Path to the Microsoft Edge binary to use (on macOS, the path should be the actual binary, not just the app.  for example, `/Applications/Microsoft Edge.app/Contents/MacOS/Microsoft Edge`). |
| `debuggerAddress` | string | An address of a debugger server to which to connect, in the form of `hostname/ip:port`, for example `127.0.0.1:38947`. |
| `detach` | boolean | Default value = `false`. If `false`, Microsoft Edge quits when the WebDriver service shuts down, even if the WebDriver local end hasn't closed the session.  If `true`, Microsoft Edge only quits if the WebDriver local end closes the session.  If `true`, and the WebDriver local end doesn't close the session, `EdgeDriver` doesn't clean up the temporary user data folder used by the Microsoft Edge instance. |
| `excludeSwitches` | list of strings | List of Microsoft Edge command line switches to exclude that EdgeDriver by default passes when starting Microsoft Edge.  Avoid the `--` prefix for switches. |
| `extensions` | list of strings | A list of extensions to install on startup.  Each item in the list should be a base-64 encoded packed extension (`.crx`). |
| `localState` | dictionary | A dictionary with each entry consisting of the name of the preference and the value.  The preferences are applied to the Local State file in the user data folder. |
| `minidumpPath` | string | Directory to store Microsoft Edge minidumps.  (Supported only on Linux.) |
| `mobileEmulation` | dictionary | A dictionary with either a value for `deviceName`, or values for `deviceMetrics` and `userAgent`. |
| `perfLoggingPrefs` | dictionary | An optional dictionary that specifies performance logging preferences.  For more information, see [perfLoggingPrefs object](#perfloggingprefs-object). |
| `prefs` | dictionary | A dictionary with each entry consisting of the name of the preference and the value.  The preferences are only applied to the user profile in use.  For examples, see the `Preferences` file in the user data folder of Microsoft Edge. |
| `wdpAddress` | string | An address of a Windows Device Portal server to connect to, in the form of `hostname/ip:port`, for example  `127.0.0.1:50080`.  For more information, see [Remotely debug Windows devices](../devtools/remote-debugging/windows.md). |
| `wdpPassword` | string | Optional password to use when connecting to a Windows Device Portal server.  Required if the server has authentication enabled. |
| `wdpUsername` | string | Optional user name to use when connecting to a Windows Device Portal server.  Required if the server has authentication enabled. |
| `wdpProcessId` | integer | The required process ID to use if attaching to a running WebView2 UWP app, for example `36590`. This information can be found in `browserProcessId` on `http://<Device Portal URL>/msedge`. |
| `webviewOptions` | dictionary | An optional dictionary that can be used to configure the WebView2 environment when launching a WebView2 app. For more information, see [webviewOptions object](#webviewoptions-object). |
| `windowsApp` | string | Application user model ID of a Microsoft Edge app package to launch, for example `Microsoft.MicrosoftEdge.Stable_8wekyb3d8bbwe!MSEDGE`.  Use `windowsApp` instead of `binary` when connecting to a Windows 10X device or emulator using Windows Device Portal. |
| `windowTypes` | list of strings | A list of window types that are displayed in the list of window handles.  For access to Android webview elements, include `webview` in the list. |


<!-- ====================================================================== -->
## perfLoggingPrefs object

The `perfLoggingPrefs` dictionary has the following format. All keys are optional.

| Key | Type | Default value | Details |
|:--- |:--- |:--- |:--- |
| `bufferUsageReportingInterval` | positive integer | 1000 | The requested number of milliseconds between DevTools trace buffer usage events.  For example, if 1000, then once per second, DevTools reports how full the trace buffer is.  If a report indicates the buffer usage is 100%, a warning is issued. |
| `enableNetwork` | boolean | true | To collect (or not collect) events from Network domain. |
| `enablePage` | boolean | true | To collect (or not collect) events from Page domain. |
| `traceCategories` | string | (empty) | A comma-separated string of Microsoft Edge tracing categories for which trace events should be collected.  An unspecified or empty string disables tracing. |


<!-- ====================================================================== -->
## webviewOptions object

The `webviewOptions` dictionary is used to configure the WebView2 environment when launching a WebView2 app. It has the following format. All keys are optional.

| Key | Type | Default value | Details |
|:--- |:--- |:--- |:--- |
| `browserExecutableFolder` | string | (empty) | Path to a folder containing a fixed version WebView2 runtime to use. For more information about using a fixed version runtime distribution with WebView2, see [Distribute a WebView2 app and the WebView2 Runtime](../webview2/concepts/distribution.md#the-fixed-version-runtime-distribution-mode). |
| `userDataFolder` | string | (empty) | Path to the user data folder that WebView2 will use. If `userDataFolder` isn't specified, Microsoft Edge WebDriver will create a temporary user data folder. For more information about managing the user data folder with WebView2, see [Manage the user data folder](../webview2/concepts/user-data-folder.md). |
| `additionalBrowserArguments` | list of strings |  | List of command-line arguments that WebView2 will pass to the browser process on launch. Arguments with an associated value should be separated by an `=` sign (for example, `['start-maximized', 'log-level=0']`). |
| `releaseChannelPreference` | string |  | Preferred WebView2 evergreen runtime distribution to use. Can be `"stable"` or `"canary"`. |

<!-- ====================================================================== -->
## Returned capabilities

The following list contains all of the Microsoft Edge-specific capabilities that `EdgeDriver` returns when you create a new session.

| Capability | Type | Details |
|:--- |:--- |:--- |
| `msedge.msedgedriverVersion` | string | The version of EdgeDriver. |
| `msedge.userDataDir` | string | The path to the user data folder used by the Microsoft Edge instance. |
