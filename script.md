Slides

0 - 

Hey everyone, my name is Hassan Khan Shaikley and I am here to talk to you about building a drum machine with Nerves.

1 - ME

So, about me:
I've been married for almost 5 years to the most incredible woman I have ever met.
I love to garden hydroponically and in the dirt.
I love to eat and try different foods.
Tea and Milk tea are my life. I drink way too much.
I love to tinker; that involves making all sorts of things.
I have 3 cats and a dog. I take my small chihuahua, Milo, just about everywhere with me.

2 - 

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

Note that raspberry PI 4 support isn't fully there due to it requiring a new driver. 
	[TODO: Confirm reasoning]

4 - 
This is what it looks like alltogether.
Assembly of the touchscreen is very straightforward. So far so good.

5 -  SOFTWARE
Nerves ships with aplay which allows you to play audio files. In our case we'll be playing wav files.

Scenic is a library that allows you to write cross platform UI code. This means you can run scenic applications on different devices. I can run the UI on my macbook and on my raspberry pi 3b with the touchscreen.

Scenic's only dependencies are Erlang, OTP & OpenGl

It has three layers. The Scene layer (which is what we'll be working with). 

Then there's the Viewport layer which is a bridge between the scenes and drivers. It controls the scenes life cycle and sends graphs to the drivers.

Then there's the driver layer which is hardware specific.

MEATWARE
I heard this word for the first time in The Falcon and The Winter Soldier and just loved it. I wanted to incorporate it into my vocabulary & I will jump at any opportunity to use it.


GETTING STARTED
It's only a few short steps to get started. There's a scenic new library that will do the heavy lifting and output a simple starter Nerves project for the raspberry pi 3, or whatever target you tell it to. This is documented very well in the Scenic docs.

INITIALIZING THE PROJECT

Once you have the scenic new library installed you can do this.

Create a scenic nerves project. 
Get the dependencies for the given target.
Then, as long as you have an SD card plugged in you can burn the firmware into the SD card!

And if you punch in `iex -S mix` on your dev machine you'll see this. Which is also what you'll see on the RPI.

This was a lot of words but all we've done so far is run a generator and we have our hello world embedded scenic project.

SOUND OUTPUT TO JACK

The command at the top is essentially what you want to hit the command line with to set the target for the audio output.

I've listed the various options, but I'm using a headphone jack speaker so I stick to analog.

CHANGING VOLUME

To change the volume you want to use this command and substitue percent with the percentage you want.

USING STATIC ASSETS

To use static assets, namely audio files, we place them in `priv/static`. We can access files in priv static with the following command.

We can pass these file locations as arguments to aplay to play our audio files.

PLAYING AN AUDIO FILE

The default Nerves config for the rpi3 ships with aplay, a utility for playing audio files. Just run this command and you’ve got your audio file playing.

SCENIC UI EXAMPLE

In Scenic the UI is represented as data, we call that data the graph.

In this example I'll show you a volume down button.

Here is a simple example.

The font is set there at the top.

The text we want to show is a minus sign.

The id is what is used when we want to either handle an event or send it a message / update this particular graph.

The theme describes the style.

t describess the translation also know as the location. the x and y coordinates of this element.

SCENIC UI EXAMPLE

It is generally recommended to create the starting graph at compile time. This can be done using a module variable.

In the `init` callback you can push the graph state you want to see at intialization like this.

SCENIC UI EXAMPLE

And then finally, in the root component, we can add that volume down button like so.

EVENTS & COMMUNICATING BETWEEN COMPONENTS

A component is a specialized scene, and a scene is a specialized genserver, so everything you can do with GenServers you can do with components.

It is a singleton so we can give it a name, here it is __MODULE__.

And And now we can send messages to the Scene process using the name!

EVENTS && COMMUNICATING BETWEEN COMPONENTS

From another scene we can send it events with the GenServer cast command. Since the touch event is initially handled in the root scene this `filter_event` captures events in the root scene.

You can see how we're matching on the `click` event and then the id of the component which is `volume_down`.

I want you to notice how we're saving the state here, the root scene is a genserver so I am putting the application state in here. You can use alternatives if you want such as ets, but using the root process is fast and simple, and if any components need any information we can send them the information they need using the GenServer callbacks like we're doing here.

EVENTS && COMMUNICATING BETWEEN COMPONENTS

And in the Volume button where we want to receive the cast event.

If we want to update the UI we simple update the state here, the push part is the magic and is what updates the UI with the new graph!

OPTIMIZE CPU USAGE

This is pretty fun. Full disclosure I didn’t run these on the RPI but I very well could have. I took my MacBooks word.

Here are some common operations we can optimize.

Like the slide says, you want to use atom id’s because comparison is much faster, it occurs in constant time.

If we want to put some strings together and we can avoid interpolating then that is good, there are some use cases where you can avoid interpolation and use matching instead. Be creative where you can!

Also === is a tad bit faster. This is negligible but may be mildly interesting to you.

OPTIMIZE CPU USAGE

I tried using ETS but I preferred the scene GenServer for storing state. ETS is one way you can store root state but I didn’t like the overhead that it introduced for this purpose. I did a little bit of experimenting here and I wouldn’t completely avoid using ETS. Try it if you want!

I had a benchmark. Should probably get it.

The PI B has 4 cores, use them!

OPTIMIZE CPU USAGE

I had this idea what if you could cache pure functions without any external libraries and without memoization.

And I hatched this little trick. Bryan Joseph helped me turn it into reality, thanks Bryan!

This allows you to cache pure functions at compile time. So as long as the range of inputs is limited you can pay the price via build size.

Matching is super fast, and will be faster than doing any computations. You’ll also save some memory doing this.

In this example it was 2x faster with no memory usage.

If this looks unfamiliar to you don’t worry too much about that it is almost magical.


OPTIMIZE CPU USAGE

Like Uncle Ben says With Great Power Comes Great Responsibility.

OPTIMIZE POWER COMNSUMPTION

When I first tried running making the drum machine I noticed a lightnign volt on the top right and it would sometimes brown out. This was due to a lack of power.

To reduce power consumption we can disable what we don't need!

There's an excellent library for this called `power_control`.

Something worth mentioning is the concept of a CPU governor. I'm sure most of you have seen a power save mode on your machines. Under the hood this is controlled by the governor which can adjust the CPU speed to either save power, be performant, and everything in between. The default for Nerves is power save.

If you decide you want to make some kind of instrument, or even a game, and that you need more CPU usage this is an important concept to keep in mind!

DON'T OPTIMIZE MEMORY

We have plenty and unless you absolutely need to, don't optimize memory. Pay for less CPU cycles with more memory wherever you can.

POTENTIAL IMPROVEMENTS

Here are some potential improvements in my humble opinion.

Robotic arms would be legit. I know in the past I was really into being a one man band, and have played the melodica and guitar at the same time! Which is just madness. Since we're not quite at mass produced extra arms technology yet this might be a viable alternative.

Accurate timing is really improtant to very serious musicians. I am not that serious. In the industry there's a term called noodle. I like to noodle with my guitar.

QUESTIONS

Thank you for coming to my talk! I hope you enjoyed yourself.

If you have any questions now is the time to ask!

__ NOTE __

Talk about using reduce to create several of things



# Maybe put this in the spot for thigns to do differently

Another alternative would be to use midi & soundfonts instead of wav files. But I had to pick my battles when prepping for this talk!


I was messing around with using midi files directly but I didn’t have enough time and ran into some snags