---
title: "TL;DR Core Animation - removedOnCompletion"
layout: post
---
*TL;DR While using any of the `CAAnimation`'s subclasses in Core Animation, do not use the `removedOnCompletion` property unless you know what you are doing. A better way to perform the animation is to set the `model` layer to the final state before the animation starts there by not needing to use the `removedOnCompletion` flag.*

##Implicit Animation##
Implicit Animations are the most trivial forms of animation in Core Animations. Implicit Animation are performed by changing the property of the `CALayer` to the final state. This change in property triggers an animation from the initial state to the final state. This is the most straight forward way of performing animation on the properties of a `CALayer`. The following snippet shows how to animate the opacity change of a `CALayer`.

{% highlight objective-c %}
#import "ViewController.h"

@interface ViewController ()

@property (nonatomic, strong) CALayer *centerLayer;

@end

@implementation ViewController

- (void)viewDidLoad {
    [super viewDidLoad];
    [self createSubLayer];
}

- (void)createSubLayer {
    
    CGRect layerRect = CGRectMake(100.0f, 100.0f, 100.0f, 100.0f);
    
    self.centerLayer = [CALayer layer];
    self.centerLayer.frame = layerRect;
    self.centerLayer.backgroundColor = [[UIColor redColor] CGColor];
    
    [self.view.layer addSublayer:self.centerLayer];
}

- (IBAction)animateButton:(id)sender {
    [CATransaction begin];
    self.centerLayer.opacity = 0.0f;
    [CATransaction commit];
}

@end
{% endhighlight %}

##Model Layer and Presentation Layer##
Core Animation implements an small scale MVC pattern to represent the various components in an animation. The `model layer ` is the equivalent of the *Model* in MVC. The `presentation layer` is the *View* and the Core Animation libraries act as the *Controller* in performing the animation.

Let us consider the example of Implicit Animation mentioned above. The animating `centerLayer` acts as the Model. So once the value of the `opacity ` is modified, it instantly changes to the new value of `0.0f`. This change in property triggers an animation which by default takes 0.25 seconds. Querying the `opacity ` of the CALayer during this duration will return the new value `0.0f`.

Core Animation maintains a copy of the layer that is actually displayed on the screen during the animation. Usually this is updated for every run loop. This is called the `presentation layer `. This can be obtained by calling the `-presentationLayer` method on a `CALayer`. So in the above example, querying the `opacity ` on the `presentation layer ` would provide for a range of values from 1.0f to 0.0f (Fully Visible to Completely hidden). Once the animation is finished, it is removed from the layer and the layer reverts back to the `model layer `.

The `presentation layer ` is really useful in performing Hit Test to see if there is a tap on the layer during animation.

##Explicit Animation##
Explicit Animation are the class of animation offered in Core Animation that enable us to perform complex animations. This also enables us to customize various aspects of the animation such as custom properties, non-linear animations or even key frames to animate between.

The basic premise of the Explicit Animation is that you specify a `From State` and a `To State`. The states are usually specified in one of the subclasses of `CAAnimation`. Core Animation then animates between these two states.

###Incorrect Usage###
One of the popular incorrect way of performing this type of animation is shown below.

{% highlight objective-c %}
- (IBAction)animateButton:(id)sender {
	CABasicAnimation *strokeEndAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
    strokeEndAnimation.toValue = @(0.5f);
    strokeEndAnimation.removedOnCompletion = NO;
    strokeEndAnimation.fillMode = kCAFillModeForwards;
    [self.progressCircleLayer addAnimation:self.strokeEndAnimation forKey:@"strokeEndAnimation"];
}
{% endhighlight %}

Let me explain what the code is doing first and then explain why this is incorrect. In this case the animation is changing the `strokeEnd` property of a `CALayer`. 

* A `CABasicAnimation` is created and the Key Path of interest is set to `strokeEnd`. 
* In the case of this animation, the initial state is what is present in the `model layer `. 
* The final state of the animation is specified by the property `toValue`. Here the final value is set to be 0.5. 
* Assuming the next two lines does not exist, the last line adds this animation to the layer of interest. 

Now if we try to run this code, we see that once the animation completes, the layer reverts back to its original state. The property `removedOnCompletion` controls if the animation should be removed after the animation completes. Setting this to `NO` does not remove the animation. Hence we see that the layer does not revert to the original state.

##Why is this Incorrect?##
The principle of Model and Presentation layers still exists for Explicit Animation. So what is happening above is that the `CABasicAnimation` animates the property from the current value to the value specified in the `toValue`. The issues is that the `model`'s `strokeEnd` property is unchanged. So once the animation is over, the value reverts to the model's value. So overcome this issue, `removedOnCompletion` is set to `NO`.

This is completely incorrect. From the discussion above, we know that the model and the presentation layers need to by in sync. One other issue here is that the animation is not removed there by taking up memory.

##Correct Usage##
The correct way to perform this animation would be to setup the `model` correctly before starting the animation. `CABasicAnimation` has two properties of importance, `fromValue` and `toValue`. So configuring the `model layer ` and these two properties would enable the animation to complete and not cause the layer to revert to the old state.

Let us take a look at the corrected version of the code above.
{% highlight objective-c %}
- (IBAction)animateButton:(id)sender {
	CABasicAnimation *strokeEndAnimation = [CABasicAnimation animationWithKeyPath:@"strokeEnd"];
	strokeEndAnimation.fromValue = [self.progressCircleLayer.presentationLayer ? :self.progressCircleLayer valueForKeyPath:self.strokeEndAnimation.keyPath];
    strokeEndAnimation.toValue = @(0.5f);
    [CATransaction begin];
	[CATransaction setDisableActions:YES];
	self.progressCircleLayer.strokeEnd = 0.5f;
	[CATransaction commit];
    [self.progressCircleLayer addAnimation:self.strokeEndAnimation forKey:@"strokeEndAnimation"];
}
{% endhighlight %}

* The `CABasicAnimation` is created for the property of interest `strokeEnd`.
* Next we setup the initial state of the layer in `fromValue`. We check to see if a presentation layer is available, if it does we obtain the `strokeEnd` from this presentation layer.
* Then we setup the `toValue` to the final state.
* The next four lines of code basically changes the `strokeEnd` property of the layer, however we prevent it from animating this change by using the `+setDisableActions` on the `CATransaction`.

Now when we run the code above, we see that the layer does not revert back to its initial state after the animation.