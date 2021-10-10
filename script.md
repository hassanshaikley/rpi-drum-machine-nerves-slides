Slides

0 - 

Hey everyone, my name is Hassan Khan Shaikley and I am here to talk to you about building a drum machine with Nerves.

--


1 - ME

So, about me:
I've been married for almost 5 years to the most incredible woman I have ever met.
I love to garden.
I love to eat and try different foods.
I love to tinker.
I have 3 cats and a dog. I take my small chihuahua, Milo, just about everywhere with me.
And I'm a Software Engineer @ Community.com where I work among some really talented and humble people. Please ask me anything about Community if you're interested in working with us!



DRUM MACHINE

In this talk I'll be going over what a drum machine is, why we might want to make one and how we can build one with Nerves & Scenic and if they're a good tool for the job.

2 - 

So let's get started.

A drum machine is a programmable device that is able to imitate the sound of a drum kit.
With a drum machine, you aren't limited to preset patterns you can create your own!
There are different UIs such as this one:

-

The tr808 shown here, a total gamechanger in the world of music production. This humble little device manufactured by roland corporation between 1980 and 1983 changed the game.

The row represents the beat. Select a sound like kickdrum and press the button corresponding to the beat. The drum machine will loop over each of those columns and if the button is toggled it will play the sound.
Once you have finished setting up one percussion instrument you can move on to the next.

-
If you recognize any of these songs you've heard music produced on a tr808 drum machine. I'm lying, you've probably heard a tr808 this week either directly or through a sample. The reach of this device is phenomenal. 

-

This is the drum machine I built that I am going to walk you through.

There are volume buttons, bpm buttons and a whole grid of toggle buttons.


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
And A 5.25v/3amp power supply.
Altogether this was about $135.

Note that raspberry PI 4 support isn't fully there due to it requiring a new driver. But support should be coming soon.


4 - 
This is what it looks like alltogether.
Assembly of the touchscreen is very straightforward. So far so good.

5 -  SOFTWARE

Nerves ships with aplay which is a command line app that allows you to play audio files. In our case we'll be playing wav files.

Scenic is an elixir library that allows you to write cross platform UI code. As long as a device can run erlang and opengl, it can run a scenic UI.


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

We can build out own components that have these components within them.

This is an example usage of the text primitive!

GRAPH

The graph is basically the data representation of the UI. We can push  graphs onto the UI and scenic will find a way to render them by translating the graph into input to for the devices drivers.

HEADER

This header is just a rectangle and two text boxe.


SCENIC UI EXAMPLE

So if we were to create the VolumeControls, we need a text that says vol with the current volume below it, and two buttons.

2

This is how we would create one of the buttons. The other is so similar we'll just go into making one.

Just like in HTML we give the buttons ids, styles also known as the theme, and we can easily when they're clicked or touched by their id's.

Note that we want to use atoms as IDs for performance reasons, comparing atoms is faster than strings becuase it's basically comparing references under the hood.

-

It's generally recommended to create the starting graph at compile time for performance purposes. This can be done using a module variable.

In the `init` callback you can push the graph state you want to see at intialization like this.

SCENIC UI EXAMPLE

And then finally, in the parent component, we can add that volume down button like so. Easy peasy.

The bpm controls are similar enough that we don't need to go over them.

Really the only component left is the grid of buttons that create the sounds. We'll call that the Control Panel component.


BEAT INDICATOR

This is just a bunch of rectangles with id's so we can dynamically change  their color.

CONTROL  PANEL

This is a group of buttons with tuple ids, the x and y values, so the top left is 0,0 and the one immediately below it has an id of 0,1



BREAD & BUTTER 1/2

So ultimately the way that this works is like a game. There's a loop and it's called every n milliseconds where n is determined by the bpm.

Every time it loops it immediately calls itself using process.send_after, updates the UI so that the currently played column gets highlighted, and plays any sounds that are currently set for that beat. 

Many would say Elixir isn't well suited for applications that have strong timing requirements. Process.send_after is  not good if you care about the time when it is executed. Your ear will detect a delay and good music gets really bad really fast if the timing is off.

-

Actually I was running into a lot of issues because of Process.send_after and static files. Using midi and using microtimer alleviated these issues.


I sort of lied earlier when I said I used Process.send_after.


I use MicroTimer which is a small library that has its own implementation of send_after that is far more accurate with its timing. 

EVENTS & COMMUNICATING BETWEEN COMPONENTS

In scenic, a Component is a specialized scene and a scene is a specialized genserver. So the UI is made up of a bunch of processes that can fail and recover like any other process. Components can also send and receive messages, like other processes.

-

Since Scenic is pretty liberal with its' defoverridables you can override child_spec to give your components names and send them messages. 

Upon receiving those messages they have an opportunity to trigger any UI changes.

-

OPTIMIZE CPU USAGE

So performance is really important for the drum machine since we want it to feel responsive and don't want any timing issues with the music.

Benchee is a really neat library for that if you're unaware of it. It allows you to randomize input and compare two function by calling them many many times giving you an idea of the performance comparison between the two functions.

I checked the optimizations myself on my macbook. It would make sense to run them on the rpi3 because the results may differ and unfortunately I never got around to that.

- 

Maybe this goes without saying but follow the performance best practies. 

OPTIMIZE CPU USAGE 3/5


I originally tried using ETS to store the button state at first but the performance was worse than simply storing it in the state of the root scene. 

OPTIMIZE CPU USAGE 4/5

I had this idea, what if you could cache pure functions without any external libraries and without memoization.

And I hatched this little trick. Bryan Joseph helped me turn it into reality, so thanks Bryan!

This allows you to cache pure functions at compile time. So as long as the range of inputs is limited you can pay the price through the build size.

Because Matching is extremely fast, it will be faster than doing any computations. You’ll also save some memory doing this.

In this example it was 2x faster with no memory usage.

If this looks unfamiliar to you it basically creates the volume_string function 100 times

--

FWIW this is a fairly contrived example but I hope you guys get the point! That code above generates that code in the middle, which is faster than that code at the bottom.

OPTIMIZE CPU USAGE

nifs are code written in another language that is then executed from a beam application. The overhead is generally high but sometimes they can result in performance benefits. They're also more fragile, all of OTPs guard rails and guarantees are out the door when you use nifs.

I was able to squeeze a 6% performance boost out of a function by rewriting in c and calling it from Elixir. And every percentage counts.


OPTIMIZE POWER COMNSUMPTION

When I first tried running the drum machine on a raspberry pi I noticed a lightnign volt on the top right and it would sometimes brown out. This was due to a lack of power. 

To reduce power consumption we can disable what we don't need!

There's an excellent library for this called `power_control`.

Something worth mentioning is the concept of a CPU governor. I'm sure most of you have seen a power save mode on your machines. Under the hood this is controlled by the governor which can adjust the CPU speed to either save power, be performant, and everything in between. The default for Nerves is power save.

If you decide you want to make some kind of instrument, game, or some kind of performance intensive application and that you need more CPU usage this is an important thing to keep in mind!

POTENTIAL IMPROVEMENTS

Here are some potential improvements in my opinion.

Robotic arms would be awesome.

Accurate timing is really important. Mat Trudel explores that in his talk Mix New Beats. He uses a library called Schedex to time the sounds. If I was building the perfect drum machine I would explore using that.



---

QUESTIONS

Thank you for coming to my talk! I hope you enjoyed yourself.

If you have any questions now is the time to ask!

__ NOTE __

Talk about using reduce to create several of things



# Maybe put this in the spot for thigns to do differently

Another alternative would be to use midi & soundfonts instead of wav files. But I had to pick my battles when prepping for this talk!


I was messing around with using midi files directly but I didn’t have enough time and ran into some snags