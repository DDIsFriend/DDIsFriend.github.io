---
layout: default
title:  "Cocoapods"
date:   2022-12-20 10:55:07 +0800
categories: jekyll update
---
## 新建一个cocopods lib后需要对项目做出一些调整:
<p>     1.在.podspec文件中将<mark>s.source的https改为git@</mark>。</p>
<p>2.在.podspec文件中添加<code>s.pod_target_xcconfig = { 'VALID_ARCHS' => 'x86_64 armv7 arm64' , 'EXCLUDED_ARCHS[sdk=iphonesimulator*]' => 'arm64' , 'OTHER_LINKER_FLAGS' => '$(inherited)'}</code>，当然以上只是一个示例，添加你需要的一些配置，这里只会将当前.podspec中的spec及subspec修改为以上配置,而不会影响spec和subspec的dependency的其他库，dependency的其他库由它们库本身的.podspec决定。</p>
<p>3.在podfile中添加<mark>source "https://github.com/CocoaPods/Specs.git"</mark>及<mark>source 'git@github.com:DDIsFriend/DDSpecs.git'</mark>（此为你自己的specs）。</p>
<p>4.podfile中可以添加:</p>
    post_install do |installer|
      installer.pods_project.build_configurations.each do |config|
        config.build_settings["EXCLUDED_ARCHS[sdk=iphonesimulator*]"] = "arm64"
        config.build_settings["OTHER_LINKER_FLAGS"] = "$(inherited)"
      end
    end
<p>来修改pods的配置,podfile中添加以上语句会将podfile中所有的pod库都改为以上的配置。</p>
<p> 5.在xcworkspace的xcshareddata中添加<mark>IDETemplateMacros.plist</mark>，就是文件的创建信息，例如:</p>
    //    Cocoapods.h
    //    DDDiary
    //    Created by DDIsFriend on 2022/10/31.
    
<p>6.准备提交版本前先<mark>pod lib lint</mark>（此时可能需要跟--allow-warnings --sources --use-library等依赖）。</p>
<p>7.pod lib lint成功后，需要git tag当前版本至远程，然后再<mark>pod repo push</mark>（此时可能需要跟--allow-warnings --sources --use-library等依赖）。</p>
<p>8..pod库有些是红圈，有些是房子，有些是公文包，其中房子和公文包是确定的，一个是静态库，一个是动态库，但是红圈的原因暂不清楚，唯一能发现的是红圈都是在<mark>build setting</mark>中未包含<mark>apple clang</mark>的。</p>

## Podspec
### File patterns  
1. public_header_files: 界面已经完成，打算供您产品的客户使用。公共标头作为可读源代码包含在产品中，不受限制。
2. private_header_files:该界面不适用于您的客户，或者处于开发的早期阶段。产品中包含私有标头，但标记为“私有”。因此这些符号对所有客户端都是可见的，但客户端应该明白他们不应该使用它们。
3. project_header_files:该接口仅供当前项目中的实现文件使用。项目标头不包含在目标中，目标代码中除外。客户根本看不到这些符号，只有您可以看到。
