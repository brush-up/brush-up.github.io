---
title:  "CocoaPods 사용법 정리."
excerpt: "CocoaPods 사용법 정리."

categories:
  - etc
tags:
  - [etc, Programming]

toc: true
toc_sticky: true
 
date: 2023-01-06
last_modified_at: 2023-01-06

published: true
---

# Intro
* 이제 조금은 더 편하게 살아보자라는 심경으로 한번 알아보게 됨.
* 애플 플랫폼에서 사용할 수 있는 의존성 관리 도구는 대표적으로 CocoaPods, Carthage, Swift Package Manager 등이 있고 이것들 중 CocoaPods를 정리해보자.
* https://cocoapods.org/ 를 보면 되긴한다.

# 사용법
* 코코아팟 설치하기

```
$ sudo gem install cocoapods

// 설치가 잘 됐는지 살펴 보자.
$ pod --version
```

* 코코아팟 라이브러리를 적용하고 싶은 프로젝트 경로로 이동하여 'pod init' 명령어를 입력합니다

```
// .xcodeproj 파알이 있는 곳이어야 할듯
$ cd {Xcode 프로젝트 위치}

// Profile file을 생성하자.
$ pod init

```

* 생성된 Profile 내용
	* TestProject 이 프로젝트 이름임.

```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'TestProject' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for TestProject

end

```

* Podfile을 Xcode로 실행하자. -- 이건 불필요.

```
$ open -a Xcode Podfile
```

* Podfile에 원하는 라이브러리를 추가 후 저장 하자 
	* 아래는 수정한 Profile 내용 
	* SDWebImageWebPCoder 를 추가함.

```

# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'TestProject' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!
  pod 'SDWebImageWebPCoder'
  # Pods for TestProject

end

```

* Podfile에 적은 라이브러리를 설치하자.

```
$ pod install

```

* 여기서 문제 발생
	* 사용중인 macbook 이 M1 pro인데 문제가 발생해서 아래처럼 해결함
	* cocoapods 를 제거하고 brew 를 통해서 재설치함...

```
> sudo gem uninstall cocoapods
> sudo gem uninstall ffi
> brew install cocoapods
//시간이 엄청 걸린다...
> pod install
```

* 이제 .xcworkspace 파일을 사용해서 작업하면 된다.


* 이제 .xcworkspace 를 열어보면 아래와 같은 파일들이 추가로 생성되어 있다.

```
Podfile.lock : Pods의 버전 픽스를 위한 파일
Pods : 라이브러리들이 다운로드되는 디렉토리
{프로젝트명}.xcworkspace : Pods를 사용할 수 있도록 포함된 워크스페이스 
```


* 마지막으로 삭제
	* cocoapods 을 삭제하려면 podfile에서 해당 라이브러리를 삭제 후 다시 pod install 해주시면 삭제됩니다


# 문제 해결 관련 링크
* https://stackoverflow.com/questions/68553842/error-installing-a-pod-bus-error-at-0x00000001045b8000
* https://developer.apple.com/forums/thread/652822
* https://github.com/CocoaPods/CocoaPods/issues/9907#issuecomment-655870749
