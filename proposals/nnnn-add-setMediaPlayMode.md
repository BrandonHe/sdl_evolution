# Add SetMediaPlayMode

* Proposal: [SDL-NNNN](NNNN-Add-setMediaPlayMode.md)
* Author: [xxx](https://github.com/)
* Status: **Awaiting review**
* Impacted Platforms: [Core / iOS / Android / RPC / HMI]

## Introduction

This proposal is about supporting media play mode by adding new RPC `SetMediaPlayMode`.

## Motivation

Currently, if a SDL connected `AppHMIType` is `MEDIA`, SDL dosen't support media play mode setting. Media play mode is widely supported in media player, almost all media devices (including SYNC) and music apps are supporting this feature, shuffle mode and repeat mode are mostly used in media playback. This will help improve user experience.

## Proposed solution

The proposed solution to this issue is to add new RPC `SetMediaPlayMode`. When a new Media type mobile app connected HU via SDL, it would send request RPC `setMediaPlayMode`, if HU support this function, it will response `SUCCESS`, otherwise, it will reponse `INVALID_DATA`.

### HMI API

Add new enum `repeatMode`, new enum `shuffleMode` and new function `setMediaPlayMode`to handle this two enums.
```xml
<enum name="RepeatMode">
  <description>Sets the repeat mode of the media player.</description>
  <element name=“ONE” internal_name=“REPEAT_ONE">
  </element>
  <element name=“LIST” internal_name="REPEAT_LIST">
  </element>
  <element name=“ALL” internal_name="REPEAT_ALL">
  </element>
  <element name=“NOT_SUPPORT” internal_name="REPEAT_NOT_SUPPORT">
  </element>
</enum>

<enum name=“ShuffleMode”>
  <description>Sets the shuffle mode of the media player.</description>
  <element name="OFF" internal_name=“SHUFFLE_OFF">
  </element>
  <element name="ON" internal_name="SHUFFLE_ON">
  </element>
  <element name="NOT_SUPPORT” internal_name="SHUFFLE_NOT_SUPPORT">
  </element>
</enum>

<function name="SetMediaPlayMode” messagetype="request">
    <description>Sets the play mode of the media player.</description>
    <param name=“repeatMode” type="Common.RepeatMode" mandatory="false">
      <description>Sets the repeat mode of the Media app connected via SDL. The available repeat mode is “ONE”, “LIST”, ALL” and “NOT_SUPPORT”.</description>
      <description>If this param is not provided, or is set to “NOT_SUPPORT”, the HU system may hide the button or set the button as unable.</description>
    </param>
    <param name=“shuffleMode" type="Common.ShuffleMode" mandatory=“false”>
      <description>Sets the shuffle mode of the Media app connected via SDL. The available shuffle mode is “OFF”, “ON” and “NOT_SUPPORT”.</description>
      <description>If this param is not provided, or is set to “NOT_SUPPORT”, the HU system may hide the button or set the button as unable.</description>
    </param>
    <param name="appID" type="Integer" mandatory="true">
      <description>ID of application that requested this RPC.</description>
    </param>
</function>
<function name="SetMediaPlayMode" messagetype="response">
</function>
```

### MOBILE API

Add funcitonID in base request/response RPCs and Base Notifications, also add corresponding RPCs has added in HMI API

```xml

<enum name="RepeatMode">
  <description>Sets the repeat mode of the media player.</description>
  <element name=“ONE” internal_name=“REPEAT_ONE">
  </element>
  <element name=“LIST” internal_name="REPEAT_LIST">
  </element>
  <element name=“ALL” internal_name="REPEAT_ALL">
  </element>
  <element name=“NOT_SUPPORT” internal_name="REPEAT_NOT_SUPPORT">
  </element>
</enum>

<enum name=“ShuffleMode”>
  <description>Sets the shuffle mode of the media player.</description>
  <element name="OFF" internal_name=“SHUFFLE_OFF">
  </element>
  <element name="ON" internal_name="SHUFFLE_ON">
  </element>
  <element name="NOT_SUPPORT” internal_name="SHUFFLE_NOT_SUPPORT">
  </element>
</enum>

<!--
Base Request / Response RPCs
Range = 0x 0000 0001 - 0x 0000 7FFF
--> 
<element name="SendHapticDataID" value="49" hexvalue="31" />
<element name="SetMediaPlayModeID" value="50" hexvalue="32" />

<function name="SetMediaPlayMode" functionID="SetMediaPlayModeID" messagetype="request">
  <param name=“repeatMode” type="Common.repeatMode" mandatory="false">
    <description>Sets the repeat mode of the Media app connected via SDL. The available repeat mode is “ONE”, “LIST”, ALL” and “NOT_SUPPORT”.</description>
    <description>If this param is not provided, or is set to “NOT_SUPPORT”, the HU system may hide the button or set the button as unable.</description>
  </param>
  <param name=“shuffleMode" type="Common.ShuffleMode" mandatory=“false”>
    <description>Sets the shuffle mode of the Media app connected via SDL. The available shuffle mode is “OFF”, “ON” and “NOT_SUPPORT”.</description>
    <description>If this param is not provided, or is set to “NOT_SUPPORT”, the HU system may hide the button or set the button as unable.
    </description>
  </param>
</function>

<function name="SetMediaPlayMode" functionID="SetMediaPlayModeID" messagetype="response">
  <param name="success" type="Boolean" platform="documentation" mandatory="true">
    <description> true if successful; false, if failed </description>
  </param>
  <param name="resultCode" type="Result" platform="documentation" mandatory="true">
    <description>See Result</description>
    <element name="SUCCESS"/>
    <element name="INVALID_DATA"/>
    <element name="OUT_OF_MEMORY"/>
    <element name="TOO_MANY_PENDING_REQUESTS"/>
    <element name="APPLICATION_NOT_REGISTERED"/>
    <element name="GENERIC_ERROR"/>
    <element name="REJECTED"/>
    <element name="IGNORED"/>
  </param>
  <param name="info" type="String" maxlength="1000" mandatory="false" platform="documentation">
    <description>Provides additional human readable info regarding the result.</description>
  </param>
</function>

```

## Potential downsides

The proposed change is backward compatible and will not cause a breaking change.

## Impact on existing code

This should not have an impact on the existing code. Existing Media apps are note affected as this proposal is adding an new function. If a newer proxy sends this request to an unsupported Core, Core would respond with `INVALID_DATA`.

## Alternatives considered

There have two alternative proposed solution to provide this function, one is to custom `SoftButton` and another is to add `AddSubMenu`.
- For `SoftButton` solution, developers would define two softButton with name **Shuffle** and **Repeat**. Based on current template Media - with softButton, this is not a clear API, most apps doesn't support this function.
- For `AddSubMenu` solution, developers would add a subMenu name **setMediaPlayMode** under **Menu** button, and then add **Shuffle Mod** and **Repeat Mode** under this subMenu. For this solution, users have to firstly goto the Menu button, then select subMenu **SetMediaPlayMode** and finally choose the play mode they want to change. This would not a good user experience design.
