
SOUND OUTPUT TO JACK

The command at the top is essentially what you want to hit the command line with to set the target for the audio output. 

1 Means analog or headphone jack. There are some other options like automatic, HDMI and none.

You may not need to run this but including it because it's important to sound work.


CHANGING VOLUME

To change the volume you want to use this command and substitue percent with the percentage you want.





```elixir
 def child_spec({args, opts}) do
   # name allows us to communicate via the name
   start_opts = [__MODULE__, args, Keyword.put_new(opts, :name, __MODULE__)]
   %{
     id: make_ref(),
     # important bit 👇
     start:
       {Scenic.Scene, :start_link, start_opts},
     type: :worker,
     restart: :permanent,
     shutdown: 500
   }
 end
```
---

## Events & Communicating between components 2/4

```elixir
def filter_event({:click, element_id}, _context, state) do

end
```
---

## Events & Communicating between components 3/4

In the root component where we want to send messages

```elixir
  alias RpiDrumMachineNerves.Components.VolumeControls
  ...
  def filter_event({:click, :volume_down}, _context, state) do
    new_volume = decrease_volume(state.volume)
    GenServer.cast(VolumeControls, {:update_volume, new_volume})
    new_state = Map.put(state, :volume, new_volume)
    {:noreply, new_state}
  end
```
---

## Events & Communicating between components 4/4

In the component we want to receive messages from

```elixir
def handle_cast({:update_volume, new_volume}, state) do
  vol = Integer.to_string(new_volume)
  graph = Graph.modify(state.graph, :volume_label, &text(&1, vol))
  {:noreply, state, push: graph}
end
```


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







fluidsynth allows you to play midi files, it isn't bundled with nerves by default but you can make a custom nerves build that includes it. I experimented with this a little bit but didn't have the time to fully do it. I see it as a huge improvement to wav files because midi is more flexible, has a smaller footprint, and allows you to play different intruments with ease.






PLAYING AN AUDIO FILE

The default Nerves config for the rpi3 ships with aplay, a utility for playing audio files. Just run this command and you’ve got your audio file playing.


USING STATIC ASSETS

To use static assets, namely audio files, we place them in `priv/static`. We can access files in priv static with the following command.

We can pass these file locations as arguments to aplay to play our audio files.




## Playing an audio file

```bash
aplay -q path/to/audio/file.wav
```

☝️ terminal command, 👇 Elixir equivalent

```elixir
System.cmd("aplay", ["-q", path_to_audio_file])
```


---

## Using static assets

put wav files in `priv/static` 
accessible at
```elixir
priv_dir = :code.priv_dir(:rpi_drum_machine_nerves)
Path.join(priv_dir, "static")`
```

---




Many would say Elixir isn't well suited for applications that have strong timing requirements. In a sense they're right. Process.send_after is  not good if you care about the time when it is executed. Your ear might detect a delay when the BPM gets high and good music gets really bad really fast if the timing is off



ACTUALLY MAKING SOUNDS

There's an amazing tool called fluidsynth that allows you to play midi files. Midi has a small footprint, is great quality, is interoperable and allows for many insturments not just drums.


But midi was great. Frank hunthless has an elixir adapter for fluidsynth called midi synth.

It isn't bundled with nerves so I created a custom nerves system that included it. This part was really fun.