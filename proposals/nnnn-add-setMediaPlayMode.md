# Add SetMediaPlayMode

* Proposal: [SDL-NNNN](NNNN-Add-setMediaPlayMode.md)
* Author: [BrandonHe](https://github.com/brandonhe)
* Status: **Awaiting review**
* Impacted Platforms: [Core / iOS / Android / RPC / HMI]

## Introduction

This proposal is about adding new function name SetMediaPlayMode.

## Motivation

Currently, if a SDL connected `AppHMIType` is `MEDIA`, SDL dosen't support media play mode setting. Media play mode is widely supported in media player, almost all devices (including SYNC) and apps are supporting this feature, shuffle mode and repeat mode are mostly used in media playback. This will help improve user experience.

## Proposed solution
When a new Media type mobile app connected HU via SDL, it sends RPC `setMediaPlayMode` request, if HU support this function, it will response `SUCCESS`, otherwise, it will reponse `INVALID_DATA`.

### HMI API

Add `repeatMode` and `shuffleMode` mode and `setMediaPlayMode`
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
      <description>Sets the repeat mode of the Media app connected via SDL. The available cycle mode is “ONE”, “LIST”, ALL” and “NOT_SUPPORT”.</description>
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

<function name="OnMediaPlayModeChange” messagetype="notification">
    <description>Must be sent by HU system when the user clicks on buttons of play mode. One or both param below should be provided.</description>
    <param name="repeatMode" type="Common.RepeatMode" mandatory=“false”>
      <description>The cycle mode which is supposed to be switched to.</description>
    </param>
    <param name="shuffleMode" type="Common.ShuffleMode" mandatory=“false”>
      <description>The shuffle mode which is supposed to be switched to.</description>
    </param>
  </function>
```

### MOBILE API

Add funcitonID in base request/response RPCs and Base Notifications, also add corresponding RPCs has added in HMI API

```xml
<!--
Base Request / Response RPCs
Range = 0x 0000 0001 - 0x 0000 7FFF
--> 
<element name="SendHapticDataID" value="49" hexvalue="31" />
<element name="SetMediaPlayModeID" value="50" hexvalue="32" />
:
<!--
 Base Notifications
 Range = 0x 0000 8000 - 0x 0000 FFFF
 -->

<element name="OnWayPointChangeID" value="32784" hexvalue="8010" />
<element name="OnMediaPlayModeChangeID" value="32785" hexvalue="8011" />

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

<function name="OnMediaPlayModeChange” functionID="OnMediaPlayModeChangeID" messagetype="notification">
  <description>Must be sent by HU system when the user clicks on buttons of play mode. One or both param below should be provided.</description>
  <param name="cycleMode" type="Common.RepeatMode" mandatory=“false”>
    <description>The cycle mode which is supposed to be switched to.</description>
  </param>
  <param name="shuffleMode" type="Common.ShuffleMode" mandatory=“false”>
    <description>The shuffle mode which is supposed to be switched to.</description>
  </param>
</function>
```

## Potential downsides

The proposed change is backward compatible and will not cause a breaking change.

## Impact on existing code

This should not have an impact on the existing code. Existing Media apps are note affected as this proposal is adding an new function. If a newer proxy sends this request to an unsupported Core, Core would respond with `INVALID_DATA`.

## Alternatives considered

The alternative proposed solution is to use existing `softButton` and `addCommand` to provide this function as a workaround. However, this would make a unclean API.
- For `softButton`, user sends notificaiton from HMI to connected Media app, the player mode will change according to Media app response.
- For `AddCommand`, user have to goto the Menu button and then choose the play mode.
