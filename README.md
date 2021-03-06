#FlightAnimator

[![Cocoapods Compatible](https://img.shields.io/badge/pod-v0.5.0-blue.svg)]()
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)]()
[![Platform](https://img.shields.io/badge/platform-ios-lightgrey.svg)]()
[![License](https://img.shields.io/badge/license-MIT-343434.svg)](/LICENSE.md)

![alt tag](/Documentation/FlightBanner.jpg?raw=true)

##Introduction

FlightAnimator is a natural animation engine built on top of CoreAnimation. Implemented with a blocks based approach it is very easy to create, configure, cache, and reuse animations dynamically based on the current state. 

FlightAnimator uses CAKeyframeAnimation(s) and CoreAnimationGroup(s) under the hood. You can apply animations on a view directly, or cache animations to define states to apply at a later time. The animations are technically a custom CAAnimationGroup, once applied to the layer, will dynamically synchronize the remaining progress based on the current presentationLayer's values.

Before beginning the tutorial feel free to clone the repository, and checkout the demo app included with the project. In the project you can set different curves for bounds, position, alpha, and transform the experiment by adjusting the easing to see the resulting effects.

<br>


##Features

* [Support for 43+ parametric curves](/Documentation/parametric_easings.md)
* Custom springs and decay animations 
* Support for custom springs, and decay
* Blocks based animation builder
* Muti-Curve group synchronisation
* Progress based animation sequencing
* Support for triggering cached animations
* Easing curve synchronization


##Installation

* [Installation Documentation](/Documentation/installation.md)
* [Release Notes](/Documentation/release_notes.md)

##Basic Use 

There are two ways you can use this framework, you can perform an animation on a specific view, or register an animation on a view to perform later. 

When creating or registering an animation, the frame work uses a blocks based syntax to build the animation. You can apply a value, a timing, and set the primary flag, which will be discussed at a later point in the documentation.

###Simple Animation

To perform a simple animation  call the `animate(:)` method on the view you want to animate. Let's look at a simple example below.

```
view.animate { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
      animator.position(newPositon).duration(0.5).easing(.EaseInSine)
}
```
The closure returns an instance of an FAAnimationMaker, which can be used to build a complex animation to perform, one property at a time. You can apply different durations, and easing curves for each individual property in the animation. And that's it, the animation kicks itself off, applies the final animation to the layer, and sets all the final layers values on the model layer.

In the case you have defined a custom NSManaged animatable property, i.e progress to draw a circle. You can use the `value(value:forKeyPath:)` method on the animator to animate that property.

```
view.animate { (animator) in
      animator.value(value, forKeyPath : "progress").duration(0.5).easing(.EaseOutCubic)
}
```

##Sequence

Chaining animations together in flight animator is very easy, and allows you to trigger another animation based on the time progress, or the value progress of an animation.

You can nest a trigger on a parent animation at a specified progress, and trigger which will trigger accordingly, and can be applied to the view being animated, or any other view define.

Let's look at how we can nest some animations using time and value based progress triggers.

####Time Progress Trigger

A time based trigger will apply the next animation based on the the progressed time of the overall parent animation. Below is an examples that will trigger the second animation at the halfway point in time of the parent animation by calling `triggerAtTimeProgress(...)`

```
view.animate { (animator) in
	animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
    animator.position(newPositon).duration(0.5).easing(.EaseOutCubic)
    
    animator.triggerAtTimeProgress(atProgress: 0.5, onView: self.secondaryView, animator: { (animator) in
         animator.bounds(newSecondaryBounds).duration(0.5).easing(.EaseOutCubic)
         animator.position(newSecondaryCenter).duration(0.5).easing(.EaseOutCubic)
    })
}
```

####Value Progress Trigger

A value based progress trigger will apply the next animation based on the the value progress of the overall parent animation. Below is an examples that will trigger the second animation at the halfway point of the value progress on the parent animation by calling `animator.triggerAtValueProgress(...)`

```
view.animate { (animator) in
	animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
    animator.position(newPositon).duration(0.5).easing(.EaseOutCubic)
    
    animator.triggerAtValueProgress(atProgress: 0.5, onView: self.secondaryView, animator: { (animator) in
         animator.bounds(newSecondaryBounds).duration(0.5).easing(.EaseOutCubic)
         animator.position(newSecondaryCenter).duration(0.5).easing(.EaseOutCubic)
    })
}
```
##Cache and Reuse Animations

You can define animation states up fron using keys, and triggers then at any other time in your application flow. When the animation is applied, if the view is in mid flight, it will synchronize itself accordingly, and animate to it's final destination. To register an animation, you can call a glabally defined method, and just as you did earlier define the property animations within the maker block.

####Register Animation

The following example shows how to register, and cache it for a key on a specified view view. This animation is only cached, and is not performed until it is manually triggered at a later point.

```
struct AnimationKeys {
	static let CenterStateFrameAnimation  = "CenterStateFrameAnimation"
}
...

registerAnimation(onView : view, forKey : AnimationKeys.CenterStateFrameAnimation) { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
      animator.position(newPositon).duration(0.5).easing(.EaseOutCubic)
})
```

####Trigger Keyed Animation


To trigger the animation all you have to do is call the following 

```
view.applyAnimation(forKey: AnimationKeys.CenterStateFrameAnimation)
```

In the case there is a need to apply the final values without actually animating the view, you can override the default animated flag to false, and it will apply all the final values to the model layer of the associated view.

```
view.applyAnimation(forKey: AnimationKeys.CenterStateFrameAnimation, animated : false)
```


##Advanced Use

Due to the dynamic nature of the framework, it won't always perform the way that you expect at first, and may take a few tweaks to get it just right. FlightAnimator has a few settings that allow for customization your animation duration.

The options you have are the following:

* Designating timing priority during synchronization for the overall animation
* Designating a primary driver on individual property animations within a group

####Timing Priority

First a little background, the framework basically does some magic so synchronize the time by prioritizing the maximun time remaining based on progress if redirected in mid flight.


Lets look at the following example of setting the timingPriority on a group animation to .MaxTime, which is the default value, and start with a behavior you are familiar with from FlightAnimator.

```
func animateView(toFrame : CGRect) {
	
	let newBounds = CGRectMake(0,0, toFrame.width, toFrame.height)
	let newPosition = CGPointMake(toFrame.midX, toFrame.midY)
	
	view.animate(.MaxTime) { (animator) in
      	animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
      	animator.position(newPositon).duration(0.5).easing(.EaseInSine)
	}
}
```
Just like the demo app, This method gets called by different buttons, and takes on the frame value of button that triggered the method. Let's the animation has been triggered, and is in mid flight. While in mid flight another button is tapped, a new animation is applied, and ehe position changes, but the bounds stay the same. 

Internally the framework will figure out the current progress in reference to the last animation, and will select the max duration value from the array of surations on the grouped property animations. 

Lets assume the bounds don't change, thus animation's duration is assumed to be 0.0 after synchronization. The new animation will synchronize to the duration of the position animation based on progress, and automatically becomes the max duration based on the **.MaxTime** timing priority.

The more animations that you append, the more likely you will need to adjust how the timing is applied. For this purpose there are 4 timing priorities to choose from:

* .MaxTime 
* .MinTime
* .Median
* .Average

Now this leads into the next topic, and that is the primary flag.

####Primary Flag

As in the example prior, there is a mention that animations can get quite complex, and the more property animtions we append, the more likely the animation will have a hick-up in the timing, especially when synchronizing 4+ animations with different curves and durations.

For this purpose, we can set the pripary flag on the property animations, and designate them as primary duration drivers. By default, if no property animation is set to primary, during synchronization, FlightAnimator will use the timing priority setting to find the corresponding value from all the animations after progress synchonization.

If we need only some specific property animations to define the progress accordingly, and become the primary drivers, you can set the primary flag to true, which will exclude any other animation which is not marked as primary from consideration.

Let's look at an example below of a simple view that is being animated from it's current position to a new frame using bounds and position.

```
view.animate(.MaxTime) { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic).primary(true)
      animator.position(newPositon).duration(0.5).easing(.EaseInSine).primary(true)
      animator.alpha(0.0).duration(0.5).easing(.EaseOutCubic)
      animator.transform(newTransform).duration(0.5).easing(.EaseInSine)
}
```

Simple as that, now when we redirect the animation in mid flight, only the bounds and position animations will be considered as part of the timing synchronization.


##Reference 
[Supported Parametric Curves](/Documentation/parametric_easings.md)

[CALayer's Supported Animatable Property](/Documentation/supported_animatable_properties.md)


## License

FlightAnimator is released under the MIT license. See [License](/LICENSE.md) for details.