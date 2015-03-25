---
title: "TL;DR Core Animation - CACurrentMediaTime"
layout: post
---
*TL;DR When creating animations with delays between them, always set it up with `CACurrentMediaTime()` for the `beingTime` property.*

## Loading Animation
In one of the recent apps I worked on, I had to create an animation when a network request is made. It was setup as three little blocks moving up and down with a delay between each one of them.
![animation](https://raw.githubusercontent.com/malolans/BasicDelayAnimation/master/animation.gif)

As shown in the animation above each block is moving up and down in `0.4 seconds`. However there is a delay between each of the blocks. This delay is supposed to be `0.15 seconds`.

## Animation in Code
Setting up the individual animation was pretty straight forward. A `CABasicAnimation` is setup for `0.4 seconds`. The animations `toValue` is setup to the translated transform of the current position. To return back to the original position, the `autoreverses` is set to `YES`. Lopping the animation is achieved by setting the `repeatCount` to `HUGE_VALF`.

{% highlight objective-c %}
- (CABasicAnimation *)getAnimationForLayer:(CALayer *)layer andViewHeight:(CGFloat)height {
    CGFloat layerHeight = CGRectGetHeight(layer.frame);
    
    CATransform3D transform = CATransform3DIdentity;
    transform = CATransform3DTranslate(transform, 0, height - layerHeight, 0);
    //create a basic animation
    CABasicAnimation *animation = [CABasicAnimation animation];
    animation.keyPath = @"transform";
    animation.duration = 0.4f;
    animation.autoreverses = YES;
    animation.repeatCount = HUGE_VALF;
    animation.toValue = [NSValue valueWithCATransform3D:transform];
    
    return animation;
}
{% endhighlight %}

## Setting up the Delay
To set up the delay, using the `timeOffset` is not the right thing to do. Instead the other `CAMediaTiming` protocol property `beginTime` time should be set. This begin should not be the exact delay. This value should be `CACurrentMediaTime() + actualDelay`.

The following code shows how to setup the delay of `0.15 seconds` between the first and the second animation.
{% highlight objective-c %}
CALayer *layer1 = [self getLayerAtIndex:0];
[view.layer addSublayer:layer1];
CABasicAnimation *animation1 = [self getAnimationForLayer:layer1 andViewHeight:CGRectGetHeight(view.frame)];
animation1.beginTime = CACurrentMediaTime();
[layer1 addAnimation:animation1 forKey:nil];

CALayer *layer2 = [self getLayerAtIndex:1];
[view.layer addSublayer:layer2];
CABasicAnimation *animation2 = [self getAnimationForLayer:layer2 andViewHeight:CGRectGetHeight(view.frame)];
animation2.beginTime = CACurrentMediaTime() + 0.15f;
[layer2 addAnimation:animation2 forKey:nil];
{% endhighlight %}

[This project](https://github.com/malolans/BasicDelayAnimation) has the sample code for the animation.

