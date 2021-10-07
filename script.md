Slides

0 - 

Hey everyone, my name is Hassan Khan Shaikley and I am here to talk to you about building a drum machine with Nerves.

1 - ME

So, about me:
I've been married for almost 5 years to the most incredible woman I have ever met.
I love to garden, that means hydroponically and in the dirt.
I love to eat and try different foods.
I love to tinker; that involves making all sorts of things.
I have 3 cats and a dog. I take my small chihuahua, Milo, just about everywhere with me.
I'm a Software Engineer at Community where I work with really talented folks on an amazingly well designed system. We're a fairly young company and we're always hiring, many people from Community are here too. Feel free to reach out to me if you're interested in working with us!

2 - 

So let's get started.

A drum machine is a programmable device that is able to imitate the sound of a drum kit.
With a drum machine, you aren't limited to preset patterns, you can create your own!
There are different UIs such as this one:
The tr808 shown here; 
The row represents the beat. Select a sound like kickdrum, snare, or what have you, and press the button corresponding to the beat. The drum machine will loop and if the button is toggled it will play the sound.
Once you have finished setting up one percussion instrument you can move on to the next.


3 - 
The hardware I chose to use to build the drum machine is
a raspberry pi 3b.
The official rpi 7 inch touchscreen.
A rechargable audio jack speaker.
And A 5.25v/3amp power supply.
Altogether this was about $135.

Note that raspberry PI 4 support isn't fully there due to it requiring a new driver. But support should be coming soon.
	[TODO: Confirm reasoning]

4 - 
This is what it looks like alltogether.
Assembly of the touchscreen is very straightforward. So far so good.

5 -  SOFTWARE
Nerves ships with aplay which allows you to play audio files. In our case we'll be playing wav files.

Scenic is a library that allows you to write cross platform UI code. This means you can run scenic applications on different devices. I can run the UI on my macbook and on my raspberry pi 3b with the touchscreen.

Scenic's only dependencies are Erlang, OTP & OpenGl

It has three layers. The Scene layer (which is what we'll be working with). 

Then there's the Viewport layer which is a bridge between the scene layer and drivers.

Then there's the driver layer which is hardware specific.

MEATWARE

As far as meatware goes, all you need is your lonesome.

GETTING STARTED
It's only a few short steps to get started. There's a scenic new library that will do the heavy lifting and output a simple starter Nerves project for the raspberry pi 3, or whatever target you tell it to. This is documented very well in the Scenic docs.

INITIALIZING THE PROJECT

Once you have the scenic new library installed you can do this.

Create a scenic nerves project. 
Get the dependencies for the given target.
Then, as long as you have an SD card plugged in you can burn the firmware into the SD card!
--

And if you punch in `iex -S mix` on your dev machine you'll see this. Which is also what you'll see on the RPI.

This was a lot of words but all we've done so far is run a generator and we have our hello world embedded scenic project.

SPOILER ALERT

This is what the finished project looks like. The red things I call the step indicators. The highlighted one is the currently active one. The BPM is the beats per minute, basically how quickly the drums are being played.

SOUND OUTPUT TO JACK

The command at the top is essentially what you want to hit the command line with to set the target for the audio output.

I've listed the various options, but I'm using a headphone jack speaker so I'm using analog.

CHANGING VOLUME

To change the volume you want to use this command and substitue percent with the percentage you want.

USING STATIC ASSETS

To use static assets, namely audio files, we place them in `priv/static`. We can access files in priv static with the following command.

We can pass these file locations as arguments to aplay to play our audio files.

PLAYING AN AUDIO FILE

The default Nerves config for the rpi3 ships with aplay, a utility for playing audio files. Just run this command and you’ve got your audio file playing.

SCENIC UI EXAMPLE

In Scenic the UI is represented as data, we call that data the graph.

In this example I'll show you how to build the volume down button.

-

The font is set there at the top. I use my current favorite font, namely Roboto mono.

The text we want to show is a minus sign. So that is why it's the first parameter to button there. button is a scenic primitive, kinda the way a button tag is sort of an html button.

The id is what we use when we want to either handle an event , send it a message, or ipdate this particular things properties like its' style or text.

The theme describes the style.

t describess the translation also know as the location. the x and y coordinates of this element.

SCENIC UI EXAMPLE

Onto the  UI Bits.
Let's explore making this volume up button.

It needs to increase the volume when it is clicked, and it needs to look like that.

--

It is generally recommended to create the starting graph at compile time. This can be done using a module variable.

In the `init` callback you can push the graph state you want to see at intialization like this.

SCENIC UI EXAMPLE

And then finally, in the parent component, we can add that volume down button like so.

EVENTS & COMMUNICATING BETWEEN COMPONENTS

A component is a specialized scene, and a scene is a specialized genserver, so everything you can do with GenServers you can do with components.

Since it is a singleton component we can name them __MODULE__.

For those unaware we can use the name to send messages to this process. Afterall each scene is a specialized genserver.

EVENTS && COMMUNICATING BETWEEN COMPONENTS

From another scene we can send it events with the GenServer cast or call commands. Since the touch event is initially handled in the root scene this `filter_event` captures events in the root scene.

You can see how we're matching on the `click` event and then the id of the component which is `volume_down`.

I want you to notice how we're saving the state here, the root scene is a genserver so I am putting the application state in here. You can use alternatives if you want such as ets, but using the root process is fast and simple, and if any components need any information we can send them the information they need using the GenServer callbacks like we're doing here.

EVENTS && COMMUNICATING BETWEEN COMPONENTS

We can grab that cast event in the last slide with a handle_cast and then update the text that shows the current volume.

To update the UI with the new text we need to modify the graph and push the new graph in the returned tuple.


BREAD & BUTTER

Now onto the more important bit. Setting up the toggle buttons that will actually trigger drum sounds.
Because the UI is represented as code I used reduce to create the data that will go into building our buttons.

This code produces list of tuples with the x and y position, what I called the translation earlier, and a tuple with the x, y which is the 
row and column of the button. that last tuple becomes a part of the id for that specific button.

--

So, Scenic is on the verge of having toggle button support with version 1.11. But until it is out we need to fake it.

We can fake it by using two buttons, one on top of the other, and hiding one of them. This is more expensive than it ought to be because you have twice as many button scenes. 

Don't worry too much about the implementation of the push button function. Just know that it is fed a graph and the data from earlier.

This creates one button directly ontop of another, the pressed button is a different color, and it initializes the down button as invisible.

--

Now we're ready to handle touch events. When a button is touched the unique identifier, which is the row, col and either a :down or :up atom called direction will hit this filter_event function.

We then store that the button is down or up in the  state.

--

And on to the more interesting bit! Just like a game we have a loop, and it is called 500ms after the genserver is  started to give the raspberry PI a little bit of time to think.

Notice how the iteration is in the state. The iteration is a number between 0 and 7 corresponding to the 8 rows.

The BPM is the beats per minute. For those unfamiliar with music that is the speed of the song. The higher the BPM the faster the music.

--

And to me, this is the funnest part.

Here we play the sounds for the current iteration.

We inform any UI components that need to know that a loop just occured.

We store the next iteration in the state.

We call the loop again in however many milliseconds based off of the bpm. We want to do that at the start because what if we spend 450ms executing this code. If the send_after was at the end we would have called it 450ms late @ 450 + 500 ms. 

Note the BPM is changable like the volume which is why it isn't a static value.

OPTIMIZE CPU USAGE

This is another fun part. I checked all of these optimizations myself but I ran them on my macbook. It would make sense to eventually run them on the rpi3 because the results may differ.

Here are some common operations we can optimize.

Like the slide says, you want to use atom id’s because comparison is much faster, it occurs in constant time.

If we want to put some strings together and we can avoid interpolating then that is good, there are some use cases where you can avoid interpolation and use matching instead. If you can, do that as it is even faster.

Also === is marginally faster than ==. This is negligible but I find it mildly interesting. I'm not 100% sure why and as tempted as I am to speculate as to why, I will refrain.

OPTIMIZE CPU USAGE

I tried using ETS but I preferred using the scenes, which again are genservers, for storing state. ETS is one way you can store root state but I didn’t like the overhead that it introduced because I was looking for the fastest solution.

The RPI3B has 4 cores, use them whenever you can to trade storage for CPU.

OPTIMIZE CPU USAGE

I had this idea, what if you could cache pure functions without any external libraries and without memoization.

And I hatched this little trick. Bryan Joseph helped me turn it into reality, so thanks Bryan!

This allows you to cache pure functions at compile time. So as long as the range of inputs is limited you can pay the price through the build size.

Because Matching is extremely fast, it will be faster than doing any computations. You’ll also save some memory doing this.

In this example it was 2x faster with no memory usage.

If this looks unfamiliar to you it basically creates the volume_string function 100 times

--

FWIW this is a fairly contrived example but I hope you guys get the point! That code above generates that code in the middle, which is faster than that code at the bottom.

OPTIMIZE CPU USAGE


For the uninitated nifs are code written in another language that is then executed from the elixir application. The overhead is generally high but sometimes they can result in performance benefits. They're also more fragile, all of OTPs guard rails and guarantees are out the door when you use nifs.

So remember what Uncle Ben said, with Great Power Comes Great Responsibility.


OPTIMIZE POWER COMNSUMPTION

When I first tried running the drum machine on a raspberry pi I noticed a lightnign volt on the top right and it would sometimes brown out. This was due to a lack of power.

To reduce power consumption we can disable what we don't need!

There's an excellent library for this called `power_control`.

Something worth mentioning is the concept of a CPU governor. I'm sure most of you have seen a power save mode on your machines. Under the hood this is controlled by the governor which can adjust the CPU speed to either save power, be performant, and everything in between. The default for Nerves is power save.

If you decide you want to make some kind of instrument, or even a game, and that you need more CPU usage this is an important thing to keep in mind!

DON'T OPTIMIZE MEMORY

Don't optimize memory unless you have to. We have plenty and unless you absolutely need to, don't optimize memory. Pay for less CPU cycles with more memory wherever you can.

POTENTIAL IMPROVEMENTS

Here are some potential improvements in my opinion.

Robotic arms would be awesome.

Accurate timing is really important. Mat Trudel explores that in his talk Mix New Beats. He uses a library called Schedex to time the sounds. If I was building the perfect drum machine I would explore using that.

fluidsynth allows you to play midi files, it isn't bundled with nerves by default but you can make a custom nerves build that includes it. I experimented with this a little bit but didn't have the time to fully do it. I see it as a huge improvement to wav files because midi is more flexible, has a smaller footprint, and allows you to play different intruments with ease.

QUESTIONS

Thank you for coming to my talk! I hope you enjoyed yourself.

If you have any questions now is the time to ask!

__ NOTE __

Talk about using reduce to create several of things



# Maybe put this in the spot for thigns to do differently

Another alternative would be to use midi & soundfonts instead of wav files. But I had to pick my battles when prepping for this talk!


I was messing around with using midi files directly but I didn’t have enough time and ran into some snags