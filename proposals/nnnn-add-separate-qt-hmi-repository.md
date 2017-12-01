# Add Separate QT HMI Repository

* Proposal: [SDL-NNNN](NNNN-Add-Separate-QT-HMI-Repository.md)
* Author: [BrandonHe](https://github.com/BrandonHe)
* Status: **Awaiting review**
* Impacted Platforms: [HMI]

## Introduction

This proposal is for adding separate QT HMI repository in SmartDeviceLink.

## Motivation

The Qt technology is widely used in automotive industry to meet users' expectation and market demands. There have massive class library and integrated tooling makes IVIs and Clusters software developers more productive. As a widely used open-source software, SDL connects IVI system to smartphone applications, it should support an easier way to integrate the SDL in their IVIs, such as to integrate a Qt-based HMI rather than a Web-based HMI into their embedded systems. Currently, SDL has been maintaining a version of Generic HMI, it can be run in any browser, However, it is not for every possible platform without browser support, Qt HMI is more comfortable deploying to embedded hardware via using its open source Qt C++ cross platform development framework.

Another important reason is that SDLC have decided to remove the component Qt HMI from SDL Core, since this Qt HMI has not been supported for a long time. However, there still have many developers out of SDLC stated that they wish to use the Qt HMI. For more information, please refer to proposal [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md) and [its discussion](https://github.com/smartdevicelink/sdl_evolution/issues/335) for details. 

## Proposed solution

The proposed solution to this issue is to add a new separate Qt HMI repository, and this solution was also mentioned as [alternative 2](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered) in proposal SDL-0110. This Qt HMI should act the same way as any of the other samples HMI do, they can be cloned and run separately, and can be connected with SDL Core to work together. No installation dependency library is required from SDL Core, good cross platform are supported, and instructed documentations are also required for this repository.

### Solution in detail

There already have two Qt HMI repositories existed in github APCVSRepo repository ([HMI_SDK](https://github.com/APCVSRepo/HMI_SDK) and [HMI_SDK_LIB](https://github.com/APCVSRepo/HMI_SDK_LIB)), both of them work fine and meet the above requirements. Here in this proposal is to push the HMI_SDK_LIB repository as a sample HMI. Because the second repository has encapsulated json data analysis and has realized a template management, which is very easy for a developer to extend a new template. It also supports the latest version of Gstreamer for software and hardware codec in video streaming. For demo pictures in Qt HMI, please refer to discussion in [issue comment](https://github.com/smartdevicelink/sdl_evolution/issues/335#issuecomment-343079628).

This would not require quite a lot of work to accomplish research and development, Ford AP team can still help SDLC maintain this project in the future. The instruction document will also be offered for developers to develop a customized Qt HMI as well as integrating SDL Core with Qt HMI. Potential work include:

- Create a new Qt HMI repository in SmartDeviceLink
- Make test case
- Have a test 
- Send pull request for code reveiw
- Write document for instruction
- Anything required for new repository

Ford AP team has document of test case and test report during developing the Qt HMI, this may be a reference for SDLC.

## Potential downsides

This new Qt HMI repository is initially created for reference to AP SDL partners, some commit record and code comment are in Chinese. This may cause difficulties for English language developers to understand, it will need to update this information before sending PR.

## Impact on existing code

N/A.

## Alternatives considered
Alternative 1:</br>
Create a Qt HMI linkage outside of SmartDeviceLink repository. This would make development work simple, just add a linkage in current SmartDeviceLink repository, but it's hard to have a good project maintaining, and also lack of good document support for English developers, questions and answers are not instant response for developers.

Alternative 2:</br>
Update and maintain the component Qt HMI in a separate repository as proposed by [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered). But that's require quite a lot of more work to accomplish.
