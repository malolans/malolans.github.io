---
title: "UIAppearance and Default Styles"
layout: post
---
[`UIAppearance`](https://developer.apple.com/library/ios/documentation/uikit/reference/UIAppearance_Protocol/Reference/Reference.html) is one of the easiest way to stylize all the UI controls in the app. This protocol has been shipping since iOS 5.0. `UIAppearance` can be used to enforce a common style guide across the app. The closest analogy for the `UIAppearance` is CSS. In the case a website HTML (Model) code does not specify how the Presentation (View) is being laid out. However, it is controlled by CSS. Similarly `UIAppearance` enables customizing the presentation without modify the model class.

### How to Customize
The basic way to setup `UIAppearance` for a UI element is to get the proxy class of the UI element and modifying the appearance properties. The code below shows how to change the default track tint color on the `UIProgressView`.

{% highlight objective-c %}
[[UIProgressView appearance] setTrackTintColor:[UIColor greenColor]];
{% endhighlight %}

The UI Controls' properties can be customized based on where it is contained. `UIAppearance` provides the method `appearanceWhenContainedIn:` that enables to get the proxy of the Control when it is present in the specified container. So to change the `UIButton`'s tint color for a state when inside a `UIToolbar` would be as follows:

{% highlight objective-c %}
[[UIButton appearanceWhenContainedIn:[UIToolbar class], nil] setTintColor:[UIColor redColor] 
																 forState:state];
{% endhighlight %}

### Custom Views
`UIAppearance` can also be added to a custom UI element that is not part of `UIKit`. A particular property of the View can be made customizable. This is done by tagging it with `UI_APPEARANCE_SELECTOR`. The only caveat for using `UI_APPEARANCE_SELECTOR` is that `BOOL` properties don't work well with `UI_APPEARANCE_SELECTOR` so they need to be converted to NSNumbers. In a recent project, this is how I setup my custom UIView:

{% highlight objective-c %}
#import <UIKit/UIKit.h>

@interface HTACircularProgressView : UIView

/**
 *  The color of the progress indicator circle.
 */
@property (strong) UIColor *progressTintColor       UI_APPEARANCE_SELECTOR;

@property (assign) CGFloat progressStrokeWidthRatio UI_APPEARANCE_SELECTOR;
{% endhighlight %}

To customize this UI Controls `progressTintColor`, the `UIAppearance` proxy for `HTACircularProgressView` can be obtained and modified similar to `UIProgressView` mentioned above.

{% highlight objective-c %}
HTACircularProgressView *apperanceProxy = [HTACircularProgressView appearance];
[apperanceProxy setProgressTintColor:[UIColor greenColor]];
{% endhighlight %}

### Default Styles
Setting up the default styles for Custom Views could be a challenge. One option is to setup the customizations for the View in the app's `application:didFinishLaunchingWithOptions:` method. However, this breaks the MVC pattern. One of the other options is to keep the setup for default value is to use the `NSObjects`'s `initialize` method. 

This is the discussion on the method from the [Apple Doc](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/Foundation/Classes/NSObject_Class/Reference/Reference.html#//apple_ref/occ/clm/NSObject/initialize):
> ####initialize
> Initializes the class before it receives its first message.

> `+ (void)initialize`
> ####Discussion
> The runtime sends initialize to each class in a program just before the class, or any class that inherits from it, is sent its first message from within the program. The runtime sends the initialize message to classes in a thread-safe manner. Superclasses receive this message before their subclasses.

The implementation for the custom class will be as follows:

{% highlight objective-c %}
+ (void)initialize {
	if (self == [HTACircularProgressView class]) {
		HTACircularProgressView *proxy = [HTACircularProgressView appearance];	
		[proxy setProgressTintColor:[UIColor greenColor]];
	}
}
{% endhighlight %}

So now all the presentation code is available in one place (`UIView`) instead of being strewn around in multiple locations. For more in depth information of how `UIAppearance` works, please check this [post](http://petersteinberger.com/blog/2013/uiappearance-for-custom-views/) by Peter Steinberger.
