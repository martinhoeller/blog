---
title:  "Instantiating Images with Swift Property Wrappers"
date:   2020-05-10 10:00:00 +0200
categories: Development
tags: swift ios
---

When instatiating a `UIImage` in code we usually write something like this:

{% highlight swift %}
let myImage = UIImage(named: "myImage")
{% endhighlight %}

Easy enough, but often times we need to reuse the same image multiple times in our code base. Copying and pasting the same initializer over and over again is painful and error prone. This post explores a way to improve this by using a Swift property wrapper.

<!--more-->

How about wrapping it in a struct and making nice contants?

{% highlight swift %}
struct Images {
  static let myImage = UIImage(named: "myImage")
}
{% endhighlight %}

This is already much better. We now have a safe way to retreive the image and we even get autocomplete support. For simple code bases this is enough and good to go.

## Frameworks and Bundles

But what if we are working in a modularized code base where the image assets are embedden in a frameworks? Instatiating an image now becomes this:

{% highlight swift %}
let bundle = Bundle(for: MyClass.self)
let myImage = UIImage(named: "myImage", in: bundle, compatibleWith: nil)
{% endhighlight %}

It gets even more involved when the images are embedded in an asset bundle within the framework:

{% highlight swift %}
let frameworkBundle = Bundle(for: MyClass.self)
let assetUrl = frameworkBundle.url(forResource: "Assets", withExtension: "bundle")!
let assetBundle = Bundle(url: assetUrl)!
let myImage = UIImage(named: "myImage", in: assetBundle, compatibleWith: nil)
{% endhighlight %}

## ImageAssetRetreivable Property Wrapper

Copy/pasting the above block of code each time you need that image is an absolute no-go. And although it is absolutley possible to create a nice struct with traditional Swift, I find it so much more elegant to solve this with a property wrapper.

*For a great introduction to Swift property wrappers have a look at John Sundell's article [Property Wrappers in Swift](https://swiftbysundell.com/articles/property-wrappers-in-swift/).*

{% highlight swift %}
// 1
extension Bundle {
  static var assetsBundle: Bundle {
    let frameworkBundle = Bundle(for: MyClass.self)  // use any class in your framework
    let assetUrl = frameworkBundle.url(forResource: "Assets", withExtension: "bundle")!
    return Bundle(url: assetUrl)!
  }
}

// 2
@propertyWrapper
struct ImageAssetRetreivable<UIImage> {
  let imageName: String

  init(_ imageName: String) {
    self.imageName = imageName
  }

  var wrappedValue: UIImage {
    return UIKit.UIImage(named: imageName, in: .assetsBundle, compatibleWith: nil) as! UIImage
  }
}

// 3
struct Images {
  @ImageAssetRetreivable("myImage")
  static var myImage: UIImage

  @ImageAssetRetreivable("myOtherImage")
  static var myOtherImage: UIImage
}
{% endhighlight %}

Let's break this down a bit.

1. We need a way to access the asset bundle. This extension to `Bundle` does just that:
  * Get the framework bundle
  * get the url of the asset bundle
  * get the asset bundle

2. The actual property wrapper. It is initialized with the name of the image in the asset catalog.
The wrapped value is the `UIImage` instantiated from the image name and the asset bundle.
I am not entirely sure why prefixing with `UIKit.` is necessary here, but the compiler otherwise complains.

3. A struct containing all the images we want to use. Adding a new image is as simple as creating a new
static var and annotating it with the property wrapper, passing the image name.

Usage:
{% highlight swift %}
let myImage = Images.myImage
{% endhighlight %}

## Conclusion

Property wrappers are a great addition to Swift and, if used carefully, can make your code easier to read, safer and more elegant.
The approach presented in this post can also be translated to other assets in asset catalogs like colors.

Let me know what you think. You can contact me either via <a href="https://twitter.com/{{site.twitter_username}}"
    target="_blank">Twitter</a> or [email](mailto:{{site.email}}).