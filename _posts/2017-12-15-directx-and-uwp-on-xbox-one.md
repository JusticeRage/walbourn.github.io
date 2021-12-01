---
layout: msdnpost
title: DirectX and UWP on Xbox One
date: 2017-12-15 02:25
author: Chuck Walbourn
comments: true
categories: [uwp, xbox]
---
With the release of the Fall Creators Update (October 2017) for Xbox One, UWP apps can now opt into expanded resources as was announced in <a href="https://blogs.windows.com/buildingapps/2017/09/15/resources-universal-windows-platform-games-fall-xbox-one-update/">this Windows Blog post</a>. Details about UWP on Xbox One can be found on <a href="https://docs.microsoft.com/en-us/windows/uwp/xbox-apps/">Microsoft Docs</a>, but in this blog post I'll be talking about a few technical issues and specifics I encountered while updating the UWP samples on the <a href="https://github.com/Microsoft/Xbox-ATG-Samples">Xbox-ATG-Samples</a> GitHub and working with the UWP versions of my <a href="https://github.com/walbourn/directx-vs-templates/wiki">Direct3D Game template</a>.
<!--more-->

<h1>Game mode</h1>

By default all UWP apps run on the Xbox One (which must support either the "Universal" or "Xbox" device family and be built for the x64 architecture) run in a shared system mode with a <a href="https://docs.microsoft.com/en-us/windows/uwp/xbox-apps/system-resource-allocation">subset of system resources</a> with the remainder of the system resources reserved for games. With the latest version of the OS, a UWP app can declare itself a 'Game' which will give it access to additional memory, CPU, and GPU resources.

When an application is published, this is handled by Windows Store metadata that indicates it is a 'Game' rather than an 'App'. During development, you manually set the package's type to 'Game' via the DevHome interface.

First, build & deploy the application package to the Xbox One. Then using the Xbox One game controller, highlight the package in DevHome's Games & apps list:

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox1.png"><img width="300" height="234" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox1.png"/></a>

Then press the View button to bring up the context menu:

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox2.png"><img width="300" height="157" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox2.png" /></a>

Go down to the bottom to <em>View details and</em> press the A button:

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox3.png"><img width="300" height="157" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox3.png" /></a>

This brings up the App details page and the <em>App type</em> field should be selected. Use the combo box to set it to "Game" by pressing A, down on the D-pad, and then pressing A again. Then press B to return to the main page.

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox4.png"><img width="300" height="169" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox4.png" /></a>

<strong>When you start the application, it will be running in Game mode.</strong>

<em>Note that the "</em><a href="https://docs.microsoft.com/en-us/previous-versions/windows/desktop/gamemode/game-mode-portal">expandedResources</a><em>" restricted capability in the Appx manifest still exists, but it's use is deprecated. While I make use of it in some of the ATG samples to simplify deployment, any UWP submitted to the Windows Store will fail submission if it makes use of this restricted capability.</em>

<strong>Note: </strong>You can also use the <a href="https://docs.microsoft.com/en-us/windows/uwp/debug-test-perf/device-portal-xbox">Xbox Device Portal (Windows Device Portal on Xbox)</a> to configure your Xbox One to default to "Game" rather than "App" for newly deployed development apps as of the November 2017 update. If you change this setting, keep in mind that if you also want to develop a non-game application, you should explicitly set it to 'App' using the instructions above.

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox5.png"><img width="300" height="122" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox5.png" /></a>

<h1>DirectX 11</h1>

For UWP on Xbox One, the DirectX 11 device is supported using <a href="https://walbourn.github.io/direct3d-feature-levels/">Direct3D Feature Level</a> 10.0. This is still true in Game mode, although you have significantly more GPU resources available to you than when you run as an 'App'.

The default DirectX 11 device creation for a UWP app typically includes the 9.3 and later Direct3D feature levels:

```cpp
D3D_FEATURE_LEVEL lvl[] =
{
    D3D_FEATURE_LEVEL_11_1,
    D3D_FEATURE_LEVEL_11_0,
    D3D_FEATURE_LEVEL_10_1,
    D3D_FEATURE_LEVEL_10_0,
    D3D_FEATURE_LEVEL_9_3
};
```

In practice Direct3D Hardware Feature Level 10.0 provides a significant set of capabilities suitable for many games:

<table>
<tbody>
<tr>
<td width="312"><strong>Supported for DirectX 11 (UWP on Xbox One)</strong></td>
<td width="312"><strong>Not supported</strong></td>
</tr>
<tr>
<td width="312">Shader Model 4.0</td>
<td width="312">Shader Model 5

DirectCompute</td>
</tr>
<tr>
<td width="312">Geometry Shader and Stream Out</td>
<td width="312">Tessellation/Hull Shaders</td>
</tr>
<tr>
<td width="312">BC1/BC2/BC3/BC4/BC5</td>
<td width="312">BC6H/BC7</td>
</tr>
<tr>
<td width="312">1D, 2D, 3D Textures

1D and 2D Texture Arrays

Cubemaps</td>
<td width="312">Cubemap arrays</td>
</tr>
<tr>
<td width="312">Instancing, Alpha-to-coverage, Event queries</td>
<td width="312">MSAA texture per-sample shaders</td>
</tr>
<tr>
<td width="312">8092 maximum texture size</td>
<td width="312"></td>
</tr>
</tbody>
</table>

> Note that in development mode, if you try to create a DirectX 11 device and do not provide the option of using ``D3D_FEATURE_LEVEL_10_0``, then you will end up with a WARP software device rather than the much faster hardware device. Remember also that WARP it not supported for retail Xbox One machines.

**Xbox Series X\|S**: When using UWP on Xbox Series X\|S, Direct3D Hardware Feature Level 11.0 is supported for DirectX 11.

See <a href="https://walbourn.github.io/anatomy-of-direct3d-11-create-device/">Anatomy of Direct3D 11 Create Device</a> for more details on DirectX 11 device creation.

<h1>DirectX 12</h1>

For UWP on Xbox One, DirectX12 is supported as <strong>Direct3D Hardware Feature Level 11.0</strong> while in Game mode. It supports Shader Model 5.1 and Shader Model 6.4 shaders.

> Note that in development mode, if you try to create a DirectX 12 device while not in Game mode, you will end up with a WARP software device rather than the much faster hardware device. Remember also that WARP it not supported for retail Xbox One machines.

See <a href="https://walbourn.github.io/anatomy-of-direct3d-12-create-device/">Anatomy of Direct3D 12 Create Device</a> for more details on DirectX 12 device creation.

<h1>Game controller usage</h1>

If you are using the Direct3D app model for your game rather than XAML or Direct3D+XAML, an important thing to note is that you need to handle the BackRequested navigation event. This event is generated whenever the B button is pressed on the Xbox One game controller, and if not handled it will result in the application being suspended as control is returned to DevHome (or the previous app). Typically Direct3D games would use <a href="https://docs.microsoft.com/en-us/uwp/api/windows.gaming.input">Windows.Gaming.Input</a> (or the XINPUT emulator library xinput_uap.lib) to react directly to the use of the B button as appropriate to the game control scheme rather than rely on the notion of a 'page stack'.

See also <a href="https://walbourn.github.io/directx-tool-kit-now-with-gamepads/">GamePad</a> in the <em>DirectX Tool Kit</em> (<a href="https://github.com/Microsoft/DirectXTK">DX11</a> / <a href="https://github.com/Microsoft/DirectXTK12/">DX12</a>).

<h2>C++/CX</h2>

If using C++/CX (a.k.a. <code>/ZW</code>) then you can implement this as follows:

```cpp
virtual void SetWindow(CoreWindow^ window)
{
    ...
    auto navigation =
        Windows::UI::Core::SystemNavigationManager::GetForCurrentView();
    navigation->BackRequested +=
    ref new EventHandler<BackRequestedEventArgs^>(this,
        &ViewProvider::OnBackRequested);
}

void OnBackRequested(Platform::Object^,
    Windows::UI::Core::BackRequestedEventArgs^ args)
{
    // UWP on Xbox One triggers a back request whenever the B
    // button is pressed which can result in the app being
    // suspended if unhandled
    args->Handled = true;
}
```

<h2>C++/WinRT</h2>

If using the <a href="https://walbourn.github.io/directx-tool-kit-and-cwinrt/">C++/WinRT projections</a>, then you can implement this as:

```cpp
void SetWindow(CoreWindow const & window)
{
    …
    auto navigation = SystemNavigationManager::GetForCurrentView();
    // UWP on Xbox One triggers a back request whenever the B
    // button is pressed which can result in the app being
    // suspended if unhandled
    navigation.BackRequested([](const winrt::Windows::Foundation::IInspectable&,
        const BackRequestedEventArgs& args)
    {
        args.Handled(true);
    });
}
```

<h1>4K</h1>

For UWP on Xbox One apps, the system always indicates a window size of 1920 x 1080 (i.e. 1080p). If the TV is running as 4K on an Xbox One S or Xbox One X, the swapchain content is automatically scaled up by the display hardware.

It is also possible to create a native 4k swapchain, but it's recommended you only do this while running on an Xbox One X. You can check which device the UWP is running on via a new <a href="https://walbourn.github.io/windows-10-fall-creators-update-sdk/">Windows 10 Fall Creators Update SDK (16299)</a> API:

```cpp
#include "Gamingdeviceinformation.h"
…
GAMING_DEVICE_MODEL_INFORMATION info = {};
GetGamingDeviceModelInformation(&info);
if (info.vendorId == GAMING_DEVICE_VENDOR_ID_MICROSOFT)
{
    switch (info.deviceId)
    {
    case GAMING_DEVICE_DEVICE_ID_XBOX_ONE:
    case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_S:
        // Keep swapchain at 1920 x 1080
        break;

    case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_X:
    case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_X_DEVKIT:
    default: // Forward compatibility
        m_outputWidth = 3840;
        m_outputHeight = 2160;
        break;
    }
}
```

Note that if your UWP app has a Target Platform Minimum Version before 10.0.0.16299, then you need to guard the use of ``GetGamingDeviceModelInformation`` as follows:

```cpp
#include <libloaderapi2.h>
extern "C" IMAGE_DOS_HEADER __ImageBase;
…
// Requires the linker settings to include
// /DELAYLOAD:api-ms-win-gaming-deviceinformation-l1-1-0.dll

if (QueryOptionalDelayLoadedAPI(
    reinterpret_cast(&__ImageBase),
    "api-ms-win-gaming-deviceinformation-l1-1-0.dll",
    "GetGamingDeviceModelInformation",
    0))
{
    GAMING_DEVICE_MODEL_INFORMATION info = {};
    GetGamingDeviceModelInformation(&info);
    if (info.vendorId == GAMING_DEVICE_VENDOR_ID_MICROSOFT)
    {
        switch (info.deviceId)
        {
        case GAMING_DEVICE_DEVICE_ID_XBOX_ONE:
        case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_S:
            // Keep swapchain at 1920 x 1080
            break;

        case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_X:
        case GAMING_DEVICE_DEVICE_ID_XBOX_ONE_X_DEVKIT:
        default: // Forward compatibility
            m_outputWidth = 3840;
            m_outputHeight = 2160;
            break;
        }
    }
}
```

You set the delay load via the Visual C++ project settings:

<a href="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox6.png"><img width="300" height="205" src="https://raw.githubusercontent.com/walbourn/walbourn.github.io/master/images/uwpxbox6.png" /></a>

If you use a native 4k swapchain on Xbox One X and the TV is in a 1080p mode, then image is downscaled automatically by the display hardware which does have the effect of 'super-sampling' for better image quality. In other words, if you decide to render at native 4k, you should do so regardless of the TV setting.

<h1>HDR</h1>

UWP on Xbox One apps have limited access to the High-Dynamic Range (HDR) capabilities of the Xbox One S and the Xbox One X. For games looking to implement HDR, contact the <a href="https://www.windowscentral.com/idxbox">ID@Xbox program</a>.
<h1>Tools</h1>
The Visual Studio Graphics Diagnostics (a.k.a. VSPIX) supports UWP on Xbox One.

> Note that neither the Xbox PIX nor the <a href="https://blogs.msdn.microsoft.com/pix/">"new" PIX</a> tool currently support UWP on Xbox One.

<strong>Related:</strong> <a href="https://walbourn.github.io/windows-10-fall-creators-update-sdk/">Windows 10 Fall Creators Update SDK</a>
