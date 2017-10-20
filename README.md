<p align="center">
	<img src="https://imgur.com/HChegN5.png">
</p>

<p align="center">
	<a href="https://cocoapods.org/pods/Lumina">
		<img src="https://img.shields.io/cocoapods/v/Lumina.svg" alt="Pods Version">
	</a>
	<a href="https://github.com/Carthage/Carthage">
		<img src="https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat" alt="Carthage Compatible">
	</a>
	<a href="https://swift.org/package-manager/">
		<img src="https://img.shields.io/badge/SwiftPM-futureReady-FC3324.svg?style=flat" alt="Swift PM Compatible">
	</a>
	<a href="https://swift.org/package-manager/">
		<img src="https://img.shields.io/badge/license-MIT-blue.svg" alt="MIT License">
	</a>
	<a href="https://github.com/RichardLitt/standard-readme">
		<img src="https://img.shields.io/badge/standard--readme-OK-green.svg" alt="Standard README Compliant">
	</a>
	<a href="https://img.shields.io/cocoapods/p/Lumina.svg?style=flat">
		<img src="https://img.shields.io/cocoapods/p/Lumina.svg?style=flat" alt="Platforms">
	</a>
</p>

----------------

Cameras are used frequently in iOS applications, and the addition of `CoreML` and `Vision` to iOS 11 has precipitated a rash of applications that perform live object recognition from images - whether from a still image or via a camera feed.

Writing `AVFoundation` code can be fun, if not sometimes interesting. `Lumina` gives you an opportunity to skip having to write `AVFoundation` code, and gives you the tools you need to do anything you need with a camera you've already built.

Lumina can:

- capture still images
- capture videos
- stream video frames to a delegate
- scan any QR or barcode and output its metadata
- detect the presence of a face and its location
- use any CoreML compatible model to stream object predictions from the camera feed

## Table of Contents

- [Requirements](#requirements)
- [Background](#background)
- [Install](#install)
- [Usage](#usage)
- [Maintainers](#maintainers)
- [Contribute](#contribute)
- [License](#license)

## Requirements

- Xcode 8.0+ (by loading Swift 4 Toolchain)
- iOS 10.0
- Swift 4.0

## Background

[David Okun](https://twitter.com/dokun24) has experience working with image processing, and he thought it would be a nice thing to have a camera module that allows you to stream images, capture photos and videos, and have a module that lets you plug in a CoreML model, and it streams the object predictions back to you alongside the video frames.

## Install

### CocoaPods

You can use [CocoaPods](https://cocoapods.org) to install `Lumina` by adding it to your `Podfile`:

```ruby
platform :ios, '10.0'
use_frameworks!

target 'MyApp' do
    pod 'Lumina'
end
```

### Carthage

You can use [Carthage](https://github.com/Carthage/Carthage) to install `Lumina` by adding it to your `Cartfile`:

```bash
github "dokun1/Lumina"
```

### Swift Package Manager

#### NB: Lumina does not currently build with Swift Package Manager due to a lack of support for frameworks that require UIKit. Nonetheless, as SPM evolves, Lumina will be ready to support it!

You can use [Swift Package Manager](https://swift.org/package-manager/) to install `Lumina` by adding the proper description to your `Package.swift` file:

```swift
import PackageDescription

let package = Package(
    name: "YOUR_PROJECT_NAME",
    targets: [],
    dependencies: [
        .Package(url: "https://github.com/dokun1/Lumina.git", majorVersion: 0)
    ]
)
```

**NB**: As the Swift Package Manager continues to grow, please view its documentation [here](https://swift.org/package-manager/#example-usage).

### Manually

Clone or download this repository, and use the provided workspace to build a version of the library for your own use in any application.

## Usage

**NB**: This repository contains a sample application. This application is designed to demonstrate the entire feature set of the library. We recommend trying this application out.

### Initialization

Consider that the main use of `Lumina` is to present a `ViewController`. Here is an example of what to add inside a boilerplate `ViewController`:

```swift
import Lumina

```

We recommend creating a single instance of the camera in your ViewController as early in your lifecycle as possible with:

```swift
let camera = LuminaViewController()
```

Presenting `Lumina` goes like so:

```swift
present(camera, animated: true, completion:nil)
```

Remember to add a description for `Privacy - Camera Usage Description` in your `Info.plist` file, so that system permissions are handled properly.

### Functionality

There are a number of properties you can set before presenting `Lumina`. You can set them before presentation, or during use, like so:

```swift
camera.position = .front // could also be .back
camera.streamFrames = true // could also be false
camera.textPrompt = "This is how to test the text prompt view" // assigning an empty string will make the view fade away
camera.trackMetadata = true // could also be false
camera.resolution = .highest // follows an enum
camera.frameRate = 60 // can be any number, defaults to 30 if selection cannot be loaded
camera.maxZoomRate = 5.0 // not setting this defaults to the highest zoom rate for any given camera device
```

### Object Recognition

**NB:** This only works for iOS 11.0 and up.

You must have a `CoreML` compatible model to try this. Ensure that you drag the model file into your project file, and add it to your current application target.

The sample in this repository comes with the `MobileNet` image recognition model, but again, any `CoreML` compatible model will work with this framework. Assign your model to the framework like so:

```swift
camera.streamingModel = MobileNet().model
```

You are now set up to perform live video object recognition.

### Handling output

To handle any output, such as still images, video frames, or scanned metadata, you will need to make your controller adhere to `LuminaDelegate` and assign it like so:

```swift
camera.delegate = self
```

Because the functionality of the camera can be updated at runtime, all delegate functions are required.

To handle the `Cancel` button being pushed, which is likely used to dismiss the camera in most use cases, implement:

```swift
func dismissed(controller: LuminaViewController) {
    // here you can call controller.dismiss(animated: true, completion:nil)
}
```

To handle a still image being captured with the photo shutter button, implement:

```swift
func captured(stillImage: UIImage, from controller: LuminaViewController) {
    // here you can take the image called stillImage and handle it however you'd like
}
```

To handle a video being captured with the photo shutter button being held down, implement:

```swift
func captured(videoAtURL: URL, from controller: LuminaViewController) {
    // here you can load the video file from the URL, which is located in NSTemporaryDirectory()
}
```

To handle a video frame being streamed from the camera, implement:

```swift
func streamed(videoFrame: UIImage, from controller: LuminaViewController) {
    // here you can take the image called videoFrame and handle it however you'd like
}
```

To handle metadata being detected and streamed from the camera, implement: 

```swift
func detected(metadata: [Any], from controller: LuminaViewController) {
    // here you can take the metadata and handle it however you'd like
    // you must find the right kind of data to downcast from, whether it is of a barcode, qr code, or face detection
}
```

To handle a `CoreML` model and its predictions being streamed with each video frame, implement:

```swift
func streamed(videoFrame: UIImage, with predictions: [LuminaPrediction]?, from controller: LuminaViewController) {
    guard let predicted = predictions else {
        return
    }
    guard let bestPrediction = predicted.first else {
        return
    }
    controller.textPrompt = "Object: \(bestPrediction.name), Confidence: \(bestPrediction.confidence * 100)%"
    }
```

The example above also makes use of the built-in text prompt mechanism for Lumina.

## Maintainers

David Okun [![Twitter Follow](https://img.shields.io/twitter/follow/dokun24.svg?style=social&label=Follow)](https://twitter.com/dokun24) [![GitHub followers](https://img.shields.io/github/followers/dokun1.svg?style=social&label=Follow)](https://github.com/dokun1)

## Contribute

See [the contribute file](CONTRIBUTING.md)!

PRs accepted.

Small note: If editing the README, please conform to the [standard-readme](https://github.com/RichardLitt/standard-readme) specification.

## License

[MIT](LICENSE) © 2017 David Okun
