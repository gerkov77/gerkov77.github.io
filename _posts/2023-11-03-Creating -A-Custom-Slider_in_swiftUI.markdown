---
layout: posts
title:  "Creating A Custom Slider in SwiftUI"
categories: coding
date: "2023-11-07"
author_profile: true
read_time: true
comments: true
share: true
---

In my current work as an iOS developer I have found it kind of hard to customize  Sliders im SwiftUI in a convenient way.
So as it is relatively easy to implement a new control in SwiftUI, I have decided to give it a shot.  ( You can download the source code for the project [Here](https://gist.github.com/gerkov77/30adc7333177c13629f0a749363a34de) )
My process for this project was the following:   

1. Creating the basic slider skeleton - a bar with a draggable handle
2. Add drag gesture - update the handle position by dragging
3. Add a binding value for the slider -  passing an initial value, position the handle accordingly, and update the value by dragging the handle.
4. Add on tint color and off tint color
5. Add initializers for cusomizing the appearance
6. Add a Range initializer - for setting min and max values, and make the slider to update the values whithin that range instead of just 0.0 - to 1.0

## 1. Creating The Basic Slider Skeleton

I have started with the basic shapes.  
So lets's Create an Xcode project, choose SwiftUI, and create a new file - choose SwiftUI View from the templates, or just create a sift file.
I knew I want to use a capsule for the slider, so the edges are rounded, otherwise I needed a straight line for the scale.  
So here is my struct.

{% highlight swift %}
struct CustomSlider: View {
    var body: some View {
        Capsule()
    }
}

{% endhighlight %}

That of course looks pretty weird, because it needs some frame.
Embed the view in a GeometryReader and set the frame as the following:  

{% highlight swift %}
 GeometryReader { reader in
            Capsule()
                .frame(width: reader.size.width, height: 2)
        }
{% endhighlight %}

Next up, I want to add the handle. I will use a Circle() shape for this.  
Let's embed our view in a ZStack, and add the Circle as the following:  

{% highlight swift %}
  GeometryReader { reader in
                ZStack {
                    Capsule()
                        .frame(width: reader.size.width, height: 2)
                    Circle()
                        .frame(width: 10)
                        .foregroundStyle(.white)
                }
            }

{% endhighlight %}

Note that the handle's foreground color is white, and it is not quite visible on the white background. When you are looking at it on previews. So in PreViews (at the end of your file) embed the whole thing in a ZStack and add Color.yellow above it, or a color of your choice. If you do not have previews because started with a pure Swift file, add it now.  
In Xcode 15 and above it looks like this:

{% highlight swift %}
#Preview {
    ZStack {
        Color.yellow
        CustomSlider()
    }
}

{% endhighlight %}

Now, I want to get that slider to the middle, so lets wrap it into a Geometry reader as well passing the closure parameter as r.  
Now the only problem is that our handle is not at the right place.  

 ![slider-screenshot-01]({{baseurl}}/assets/img/slider-screenshot-01.png){: style="border-radius: 5px"}

 Let's fix this by adding a position modifier to the circle:

 {% highlight swift %}

 .position(CGPoint(x: r.size.width/2 , y: r.size.height/2 ))
{% endhighlight %}  

## 2. Add drag gesture

Now, that our handle is in the right position - at least on the slider -, it is time to implement the drag gesture.  
The simplest method to add a drag gesture is to add a   {% highlight swift %} @Gesturestate {% endhighlight %} variable and updating it from the .updating(_:) function. However this will not do in our case, as we need to handle the phase when we released the handle, to keep our handle on the new position.  
Thus, we need to implement a DragGesture with both the {% highlight swift %} .onChanged() {% endhighlight %} and {% highlight swift %} .onEnded() {% endhighlight %} functions.  
Let's do that now!  
So, how we want to position our handle? 
We need to get the value's translation from the drag gesture, and we also need to know how that relates to the slider's legth.  
But first of all we do not want to hard code the position of the handle of course.  
So let's move that CGPoint value we have in our position modifier to a @State variable in our struct and set it's x and y values to zero.
Then substitute this variable in the position modifier.  
Next we want tu update the position of the handle, when it appears. We can do this by adding an onAppear() function after the ZStack, and update the x and y values from the reader size values.  
Next we need to add code to our onChanged function for respond to drag gestures.
Let's try something naive:  

  {% highlight swift %} 
struct CustomSlider: View {

    @State private var position = CGPoint(x: 0.0, y: 0.0)
    var body: some View {

        GeometryReader { reader in
            ZStack {
                Capsule()
                    .frame(width: reader.size.width, height: 2)

                GeometryReader { r in
                    Circle()
                        .frame(width: 10)
                        .foregroundStyle(.white)
                        .position(position)
                        .gesture(
                            DragGesture()
                                .onChanged({ value in
                                    self.position.x = self.position.x + value.translation.width
                                })
                                .onEnded({ value in

                                })
                        )
                }
            }
            .onAppear {
                position = CGPoint(x: reader.size.width/2, y: reader.size.height/2)
            }
        }
    }
}
{% endhighlight %}

The problem is that when we drag the handle it starts moving, but then leaves the screen. Dissappears to the infinite space.  
Of course, we need to implement .onEnded()!
Let's do that!
But how to use these values? Let's try something stupid!  
Let's add this code:   
{% highlight swift %} 

 .onEnded({ value in
      self.position.x = value.translation.width
    })

{% endhighlight %}  

What happens now, is that every time I drag the handle, the drag distance with practically bacomes the position x value - the wider the drag gesture is, the further the handle gets from the leading edge. We should somehow store the handle's position to solve this issue. We want to calculate the position relative to the last position of the handle. Lets add a new state variable, called lastPosition, and set it to an optional CGPoint. Then work woth this variable to calculate the handle's new position:  

{% highlight swift %} 

struct CustomSlider: View {

    @State private var position = CGPoint(x: 0.0, y: 0.0)
    // Add new variable: lastPosition
    @State private var lastPosition: CGPoint?
    var body: some View {

        GeometryReader { reader in
            ZStack {
                Capsule()
                    .frame(width: reader.size.width, height: 2)

                GeometryReader { r in
                    Circle()
                        .frame(width: 10)
                        .foregroundStyle(.white)
                        .position(position)
                        .gesture(
                            DragGesture()
                                .onChanged({ value in
                                    // set the position of the handle relative to it's last position, based on
                                    // the change of drag value on the x axis
                                    position.x = lastPosition!.x + value.translation.width

                                })
                                .onEnded({ value in
                                    // set the lastPosition variable, to the current position to keep track of it
                                    lastPosition?.x = position.x
                                })
                        )
                }
            }
            .onAppear {
                // 1. putting the handle position in the middle at start
                position = CGPoint(x: reader.size.width/2, y: reader.size.height/2)
                // 2. seting the last position same as the initial position
                lastPosition = position
            }
        }
    }
}  
{% endhighlight %}    
Now we have a slider with a draggable handle.

## 3. Add slider binding value


Our slider is not doing too much at the moment, so let's give it a value. A slider always should have an initial value and a range. But for now let's just build in the simplest steps.  
Add a @State variable called sliderValue, And set it to a Double value of 0.0.
For now we're gonna calculate a value between 0.0 and 1.0, so we add the following code after calculating the position.x in .onChanged():  

{% highlight swift %} 
   sliderValue = position.x / r.size.width
   print("slider Value: \(sliderValue)")
{% endhighlight %}

Add the slider to your ContentView and run the app. Drag the slider, and observe the values printed on the console.
We calculated the slider value by dividing the position by the slider widht. It will be 1.0 when the slider is fully on, and 0.0, when set entirely to the left side.  
Now that you are updating the slider value, change your @State variable to @Binding and remove the initial value. Note, that you'll have to update the Slider in xou preview too, by passing .constant(0.5) as for thenew sliderValue parameter. You won't be able to use the slider in the preview anymore, since the binding you passed is a constant, but that is how bindings in previews work.  
Youll have to add that parameter in the ContentView where you are calling the Slider. Create a @State variable called sliderValue in ContentView, and pass it to the slider as $sliderValue. The dollar sign means, that this is a binding value - this will be the initial value of the slider, and also this will be the variable the slider is going to write to.
Now that we have working slider, add some improvements before we start cusomizing it.  

## 4. Add on tint color and off tint color

We would like to see better, how much the slider is on or off, by adding an on tint color. Let's add the following modifier to the slider' first Capsule:  

{% highlight swift %} 
.overlay { // overlay will match the frame heigth and origin of the parent
            Capsule()
                .frame(width: position.x )
                .frame(maxWidth: reader.size.width, alignment: .leading)
                .foregroundStyle(Color.blue) // on tint color

                    }

 {% endhighlight %} 

The off tint color is basically the original color of the slider for now. But this is what wer're about to change in the next chapter.

## 5. Add initializers

We want to add initializers for all the parameters we wnt to make cusomizable. We want to add onTintColor, offTintColor, handleSize, handleColor parameters for greater flexibility. As this is already four parameters, let's wrap it into a configuration object.
{% highlight swift %} 

extension CustomSlider {
    struct Configuration {
        var onTintColor: Color = .blue
        var offTintColor: Color = .black
        var handleSize: CGFloat = 10
        var handleColor: Color = .white
    }
}
 {% endhighlight %} 

As all parameters are initialized to a default value, we can initialize this config struct without passing any parameters.  
And we're going to do the same with our Slider for now. Add this initializer:  
  
{% highlight swift %} 
struct CustomSlider: View {


    @Binding var sliderValue: Double
    let configuration: Configuration

    @State private var position = CGPoint(x: 0.0, y: 0.0)
    @State private var lastPosition: CGPoint?

    init(sliderValue: Binding<Double>, configuration: Configuration = Configuration()) {
        _sliderValue = sliderValue
        self.configuration = configuration
    }
    // rest of code omitted
}
 {% endhighlight %} 

...then we just use the variables on the view:  

{% highlight swift %}
 var body: some View {

        GeometryReader { reader in
            ZStack {
                Capsule()
                    .frame(width: reader.size.width, height: 2)
                    .foregroundStyle(configuration.offTintColor) // off tint color
                    .overlay {
                        Capsule()
                            .frame(width: position.x )
                            .frame(maxWidth: reader.size.width, alignment: .leading)
                            .foregroundStyle(configuration.onTintColor) // on tint color

                    }

                GeometryReader { r in
                    Circle()
                        .frame(width: configuration.handleSize) // handle size
                        .foregroundStyle(configuration.handleColor) // handle color
                        .position(position)
                        // rest of code omitted...
{% endhighlight %}

And now the trickiest part - add a range parameter, and use its lowerBound and upperBound parameters and all the values in between for the slider value.

## 6. Add Range parameter

1. We add a Range<Double> constant parameter for our struct with the let keyword
2. We're going to have to calculate the initial handle position based on the slider value and the range
3. We have to calculate the new slider value based on the new handle position

Add and initialize the range variable in init, and perform the changes needed to silence the compiler errors.  
First thing we need is the scale length of the slider with the new range parameter. We calculate this the following way:  

{% highlight swift %} 
extension CustomSlider {
    private var scalelength: CGFloat {
        range.upperBound - range.lowerBound
    }
}
 {% endhighlight %} 

 Then we calculate the initial handle position.  
  If you think of it, the position will be the frame width mulitplied by the ratio of the slider value and the scale length. Well, not exactly! Only if we assume, that the range's lower bound will always be 0. But it can be any positive number lower than the upper bound(!), and it can be a negative number as well. So yes, we have to extract the lower bound  from the slider value!

  Add the following code to onAppear {} 

  {% highlight swift %} 
   position = CGPoint(
                    x: reader.size.width * (sliderValue - range.lowerBound) / scalelength ,
                    y: reader.size.height/2)
                lastPosition = position
 {% endhighlight %} 

 Now we have set the initial handle position correctly!

 Let's update the sliderValue according to the drag. (We are still printing out values roughly between 0.0 and 1.0, no matter what the actual range and position of the slider is)  
 We need to implement that in .onChanged() and .onEnded() of the drag gesture.
 First we need to introduce a new @State variable called lastSliderValue (in analogy with the lastPosition) to keep track of the current values. So let's put that in the struct, and initialize it with Double 0.0. Scrap the code in .onChanded(). Replace it with:  

  {% highlight swift %} 
      .onChanged({ value in
             // 1. we are calculating the new slider value based on the last value and the 
             // ratio of drag distance and scale length
            let calcValue = lastSliderValue + value.translation.width / scalelength
                                    sliderValue = calcValue
                                    // We calculate the new position
                                    self.position.x = reader.size.width * (sliderValue - range.lowerBound)
                                })
{% endhighlight %} 
Also update the last slider value in onEnded:  

 {% highlight swift %} 
        .onEnded({ value in
             lastPosition.x = self.position!.x
             // update last slider value
            lastSliderValue = sliderValue
            })
 {% endhighlight %} 

 We also have to update the lasSliderValue with sliderValue in onAppear, to have correct values!
 Run the app now! As you see it doesn't seem to work just right!  
 The handle runs offscreen and we got a purple warning, saying 'Invalid frame dimension (negative or non-finite)' at the handle's position midifier. 
 So let's set the minimum and maximum positions correctly in onChanged!

{% highlight swift %} 
let minPosition = min(lastPosition.x + value.translation.width, r.size.width)
let maxPosition = max(minPosition, r.frame(in: .local).origin.x)
position?.x = maxPosition
// you get the slider value by rearranging 
// the equation invented in the onappear function
// caclulating the position from slider value
sliderValue = (position!.x * scalelength) / r.size.width + range.lowerBound
 {% endhighlight %}  

 That' pretty much it. The complete source code is available [here!](https://gist.github.com/gerkov77/30adc7333177c13629f0a749363a34de)  

 Happy Coding!











