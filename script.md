Slides

INTRO

Hey everyone, my name is Hassan Shaikley and I am here to talk to you about building a drum machine with Nerves.

ABOUT ME

So, about me:
I love to garden.
I love to eat and try different foods.
I love to tinker.
I have 3 cats and a dog. I take my small chihuahua, Milo, just about everywhere with me.


COMMUNITY

I'm a Software Engineer @ Community.com where I work among some really talented and humble people. Please feel free to ask me anything about  working with us!



DRUM MACHINE

In this talk I'll be going over what a drum machine is, why we might want to make one and how to build one with Nerves & Scenic.

2 - 

So let's get started.

A drum machine is a programmable device that is able to imitate the sound of a drum kit.
With a drum machine, you aren't limited to preset patterns you can create your own!
There are different UIs such as this one:

-

The tr808 shown here, a total gamechanger in the world of music production. This humble little device manufactured by roland corporation between 1980 and 1983 changed the game.


You see those buttons? They represent beats. The machine will infinitely loop across those buttons and if the button is down it'll play the sound.

-
If you recognize any of these songs you've heard music produced on a tr808 drum machine. You've also probably heard a tr808 this week either directly or through a sample. The reach of this device can't be overstated. 

-

This is the drum machine I built that I am going to walk you through.

There are volume buttons, bpm buttons and a whole grid of toggle buttons.

BPM stands for beats per minute which is the tempo. The higher the BPM the higher the tempo of the song.


-

Here it is in action. When users toggle the buttons the colors change.

So this would emit a sound like: 

boots n cats n boot

endlessly on a loop.

Let's get to building it.

-

Because this is nerves conf I am going to talk about the hardware first.

The hardware I chose to use to build the drum machine is
a raspberry pi 3b.
The official rpi 7 inch touchscreen.
A rechargable audio jack speaker.
And A 5.25v 3amp power supply.
Altogether this was about $135.


4 - 
This is what it looks like alltogether.
Assembly of the touchscreen is very straightforward. So far so good.

5 -  SOFTWARE

Scenic is an elixir library that allows you to write cross platform UI code. As long as a device can run erlang and opengl, it can run a scenic UI.

midi synth is a library by one of the Nerves core contributors frank hunleth that wraps around fluidsynth. And fluidsynth is a software synthesizer that sounds really good. It uses Midi which is super lightweight, sounds really good and allows for all kinds of different sounds.

fluidsynth isn't bundled with nerves so I created a custom nerves system that includes it.


MEATWARE

As far as meatware goes, all you need is your lonesome.

GETTING STARTED

After assembling the hardware, scenic has a generator that can create a hello world project for you.

In only a few short well documented steps we've got a starter touchscreen project running on our raspberry pi.

-

FWIW this is what it looks like when I run the starter project on my mac. This will  run on a raspbery pi as well and pick up touch events.

--



PRIMITIVES AND COMPONENTS

Scenic gives you a couple of simple building blocks. Shapes, text and some components like buttons and sliders. 

We can use scenic to build our own components that consist of other components and primitives.

-

Here's is an example usage of the text primitive!

While I'm here allow me to explain the graph.

The graph is basically the data representation of the UI. We can push  graphs onto the UI and scenic will find a way to render them by translating the graph into input to for the devices drivers.

HEADER

This header component is just a rectangle and two text boxes. One that says RPI Drum Machine Nerves and one that says version 1 point 0. No events, no nothing. Dead simple.


SCENIC UI EXAMPLE

Let's talk about a more complicated component. Namely, the volume controls.

We're gonna need one button with the plus sign as text, one button with the minus sign as the text primitive and one text to encapsulate what is on the left.


2

This is how we can create one of the buttons.

The theme describes the style.

And just like in HTML we give the buttons ids, and we can easily determine when they're clicked or touched by their id's.

Something worth mentioning is that we generally want to use atoms as IDs for performance reasons, comparing atoms is faster than strings becuase it's basically comparing references under the hood.

This gives us an idea for how to create the bpm buttons which are more or less the same.

-

It's generally recommended to create the starting graph at compile time for performance purposes. This can be done using a module variable.

In the `init` callback you can push the graph state you want to see at intialization like this.

-

And this is how we handle events for that component from the root component. This will capture touch or click events when the component has the id volume_down.

SCENIC UI EXAMPLE

And then finally, in the parent component, we can add that volume down button like so. Easy peasy.

Really the only components left are the grid of buttons that create the sounds and the red rectangles above them. 


BEAT INDICATOR

This is just a bunch of rectangles with id's so we can dynamically change  their color.

CONTROL  PANEL

This is a group of buttons and their ids are their x and y values in a tuple. So when they're clicked we know the row and the column that is clicked.

Ok, now that the UI bits are out of the way, let's get to the meat of it.

The Loop 1/2

Ultimately the way the drum machine works is like a game. There's a loop that is repeatedly calling itself after a delay.

Every time it loops it immediately invokes process.send_after, updates the red indicators above the buttons to show which beat is being played, and finally it plays any sounds that are currently set for that beat. 

-

Except that I don't use process.send_after because it isn't accurate enough. It allows for 1ms precision but when working with music we want to be as precise as possible.

Thankfully there's a little library called MicroTimer which solved this problem for me.




EVENTS & COMMUNICATING BETWEEN COMPONENTS

In scenic, a Component is a specialized scene and a scene is a specialized genserver. that means that we can send and receive messages between UI components, the way we do with other processes.

-

Since Scenic is pretty liberal with its' defoverridables you can override child_spec to give your components names and send them messages. 

Upon receiving those messages they have an opportunity to trigger any UI changes.

 When looping we call GenServer cast to tell the step indicator to highlight the correct rectangle.

OPTIMIZE CPU USAGE

I wasn't sure if I should include this but I think it's important to at least bring up.

So performance is really important for the drum machine since we want it to feel responsive and don't want any timing issues with the music.

Benchmark all the things and follow  performance best practices.

- 

Matching is really fast in Elixir. 

What if instead of doing computations at run time, we did the computations at compile time, and generated a bunch of function heads that we could match against.

This would mean that we didn't need to run the code at run time because it was already ran at compile time.

I tried it, with Bryan Joseph's help and in this example it was 2x faster with no memory usage!

As long as you know the range of input and you're willing to pay the price of a bigger build size this is a really neat trick for pure functions.

Also if this looks unfamiliar to you it basically creates the volume_string function 100 times

--

I'll give you a moment to grok this.

[give them a moment]

I know this is a fairly contrived example but I hope you guys get the point! I'm really excited about this because I've never seen anyone else do it before me.

OPTIMIZE CPU USAGE

nifs are code written in another language that is then executed from a beam application. The overhead is generally high but sometimes they can result in performance benefits. They're also more fragile, all of OTPs guard rails and guarantees are out the door when you use nifs.

I was able to squeeze a 6% performance boost out of a function by rewriting it in c and calling it from Elixir.


POTENTIAL IMPROVEMENTS

Here are some potential improvements in my opinion.

Robotic arms would be awesome.

SchedEx very well may result in more accurate beats. Mat Trudel explores that in his talk Mix New Beats where He uses Schedex to build a drum machine. If I was building the perfect drum machine I would definitely explore using that!


IN CONCLUSION

Nerves is a great tool for making a drum machine. It's also really fun to work with.

DEMO

One of my cats did something to my raspberry pi so I will have to give you this demo from my macbook. 


QUESTIONS

Thank you for coming to my talk! I hope you enjoyed yourself.