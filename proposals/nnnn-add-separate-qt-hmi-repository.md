# Add Separate QT HMI Repository

* Proposal: [SDL-NNNN](NNNN-Add-Separate-QT-HMI-Repository.md)
* Author: [BrandonHe](https://github.com/BrandonHe)
* Status: **Awaiting review**
* Impacted Platforms: [HMI ]

## Introduction

This proposal is for adding separate QT HMI repository.

## Motivation

The Qt technology is widely used in automotive industry to match user expectation and market demands, there have massive class library and integrated tooling makes IVIs and Clusters software developers more productive, SDL also as a widely used open-source software, should support easy way to integrate the SDL in their IVIs. Another important reason is that we have decided to remove the component Qt HMI from SDL Core, because this Qt HMI has not been supported for a long time, for more information, please refer to proposal [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md) and [its discussion](https://github.com/smartdevicelink/sdl_evolution/issues/335) for details. 

Currently, we have maintained a version of Generic HMI, it can be run in any browser, but its not for every possible platform without browser supported, Qt HMI is more comfortable deploying to embedded hardware via using its open source Qt C++ cross platform development framework.

Another reason is that there have many developers stated that they wish to use the Qt HMI, this demands are also mentioned from proposal [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md).

## Proposed solution

The proposed solution to this issue is to add a new separate QT HMI repository, this solution is also mentioned as [alternative 2](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered) in proposal SDL-0110. This QT HMI should act the same way as any of the other sample HMI do, they can be cloned and run separately, and it can be connected with SDL Core to work together, no install dependency required from SDL Core, good cross platform supported. Also, the good supported documentations are also required for this repository.

### Solution in detail

There already have two QT HMI repository exist in github APCVSRepo repository, [HMI_SDK](https://github.com/APCVSRepo/HMI_SDK) and [HMI_SDK_LIB](https://github.com/APCVSRepo/HMI_SDK_LIB), both of them are works fine and meet the above requirements. Here in this proposal is to push the second repository as a sample HMI, because the second repository has encapsulated json data analysis, has realized a template management, which is easy to extend a new template, it also supports the latest version of Gstreamer for video streaming software and hardware codec. For demo pictures in QT HMI, please refer to discussion in [issue comment](https://github.com/smartdevicelink/sdl_evolution/issues/335#issuecomment-343079628).

This would not require quite a lot of work to accomplish from research and develop, Ford AP team can still help SDLC maintion this project in the future. The instruction document will be offered for developers to develop a customized QT HMI as well as integrate SDL Core with QT HMI. Potential work are:

- Create new QT HMI repository in SmartDeviceLink
- Make Test Case
- Finish Testing 
- Send Pull Request for Code reveiw
- Documentation for instruction

## Potential downsides

This new QT HMI repository is initially created for AP SDL partners reference, some commit record and code comment are in Chinese, this may cause difficult for English language developers to understand, will need to update this information before send PR.

## Impact on existing code

N/A.

## Alternatives considered

The alternative solution is to update and maintain the component QT HMI in a separate repository as proposed by [SDL-0110](https://github.com/smartdevicelink/sdl_evolution/blob/master/proposals/0110-remove-qt-hmi-from-sdl-core.md#alternatives-considered). But that's require quite a lot of more work to accomplish.