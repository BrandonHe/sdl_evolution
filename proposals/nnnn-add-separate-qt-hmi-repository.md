# Add Separate QT HMI Repository

* Proposal: [SDL-NNNN](NNNN-Add-Separate-QT-HMI-Repository.md)
* Author: [BrandonHe](https://github.com/BrandonHe)
* Status: **Awaiting review**
* Impacted Platforms: [HMI ]

## Introduction

This proposal is for adding separate QT HMI repository.

## Motivation

The Qt technology is widely used in automotive industry to meet users' expectation and market demands. There have massive class library and integrated tooling makes IVIs and Clusters software developers more productive. As a widely using open-source software, SDL connects IVI system to smartphone applications, it should support an easier way to integrate the SDL in their IVIs, such as to integrate a Qt-based HMI rather than a Web-based HMI in their embedded system. Currently, SDLC have maintaining a version of Generic HMI, it can be run in any browser, but its not for every possible platform without browser supported, Qt HMI is more comfortable deploying to embedded hardware via using its open source Qt C++ cross platform development framework.

Another important reason is that SDLC have decided to remove the component Qt HMI from SDL Core, since this Qt HMI has not been supported for a long time. However, there still have many developers out of SDLC stated that they wish to use the Qt HMI. For more information, please refer to proposal [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md) and [its discussion](https://github.com/smartdevicelink/sdl_evolution/issues/335) for details. 

## Proposed solution

The proposed solution to this issue is to add a new separate Qt HMI repository, and this solution was also mentioned as [alternative 2](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered) in proposal SDL-0110. This Qt HMI should act the same way as any of the other sample HMI do, they can be cloned and run separately, and they can be connected with SDL Core to work together. No installation dependency library required from SDL Core, good cross platform supported, and good supported documentations are also required for this repository.

### Solution in detail

There already have two Qt HMI repositories existed in github APCVSRepo repository ([HMI_SDK](https://github.com/APCVSRepo/HMI_SDK) and [HMI_SDK_LIB](https://github.com/APCVSRepo/HMI_SDK_LIB)), both of them are works fine and meet the above requirements. Here in this proposal is to push the HMI_SDK_LIB repository as a sample HMI, because the second repository has encapsulated json data analysis, has realized a template management, which is very easy for a developer to extend a new template, it also supports the latest version of Gstreamer for software and hardware codec in video streaming. For demo pictures in Qt HMI, please refer to discussion in [issue comment](https://github.com/smartdevicelink/sdl_evolution/issues/335#issuecomment-343079628).

This would not require quite a lot of work to accomplish from research and develop, Ford AP team can still help SDLC maintain this project in the future. The instruction document will be offered for developers to develop a customized Qt HMI as well as to integrate SDL Core with Qt HMI. Potential works are:

- Create a new Qt HMI repository in SmartDeviceLink
- Make test case
- Have a test 
- Send pull request for code reveiw
- Write document for instruction

## Potential downsides

This new Qt HMI repository is initially created for reference to AP SDL partners, some commit record and code comment are in Chinese language, this may cause difficult for English language developers to understand, will need to update this information before send PR.

## Impact on existing code

N/A.

## Alternatives considered

The alternative solution is to update and maintain the component Qt HMI in a separate repository as proposed by [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered). But that's require quite a lot of more work to accomplish.