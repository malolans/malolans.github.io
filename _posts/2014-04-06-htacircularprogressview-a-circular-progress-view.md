---
layout: post
title: "HTACircularProgressView: A Circular Progress View"
categories:
tags:
---
I was downloading an app using the iOS App Store app. I noticed that Apple had its own version of the Circular Progress Bar. I have implemented it and this is the screen shot of the control in action.

![Screen Shot](https://raw.githubusercontent.com/malolans/HTACircularProgressView/master/ScreenCap.gif)

### Installation

#### CocoaPods

I have submitted a pull request for the CocoaPods spec. It will be available shortly.

{% highlight ruby %}
pod "HTACircularProgressView", "~> 1.0"
{% endhighlight %}

#### Direct Copy

The repo is available on github.

* Clone the repo from [this link](https://github.com/malolans/HTACircularProgressView).
* Copy the contents of the folder `HTACircularProgressView`.
* Import `HTACircularProgressView.h`
* Use it as any other `UIView`

### Usage

Instantiate a `HTACircularProgressView` like a regular `UIView`. A frame for the view should be specified. Then this view can be added to any `UIView`.

{% highlight objective-c %}
- (void)viewDidLoad
{
    [super viewDidLoad];

    CGRect frameRect = CGRectMake(self.view.center.x - 50.0f, self.view.center.y - 50.0f, 100.0f, 100.0f);

    self.circularProgressBar = [[HTACircularProgressView alloc] initWithFrame:frameRect];
    [self.view addSubview:self.circularProgressBar];
}
{% endhighlight %}

The interface to change the progress of the view is very similar to `UIProgressView`. The method to be called is `setProgress:animated:`. Here is an example of the usage:

{% highlight objective-c %}
[self.circularProgressBar setProgress:1.0f animated:YES];
{% endhighlight %}

Please refer to the header file for more information on using `HTACircularProgressView`.

### Customization

`HTACircularProgressView`'s color and thickness properties conform to the `UIApperance` protocol. So these properties can be customized by sending appearance modification messages to the class’s appearance proxy.

To change the `progressTintColor` of all `HTACircularProgressView`, the following message can be sent before the invocation of the view.

{% highlight objective-c %}
[[HTACircularProgressView appearance] setProgressTintColor:[UIColor redColor]];
{% endhighlight %}

### Requirements

* ARC Support
* iOS 7.0 or higher

### Feedback

Please let me know if you have any feedback by leaving a comment.
