---
theme: white
separator: <!--s-->
verticalSeparator: <!--v-->
revealOptions:
template: template.html
---

## How to Build a Touch-Screen Drum Machine with Nerves

#### Hassan Shaikley

<small>
Twitter: @hassanshaikley &nbsp;
Github: @hassanshaikley
</small>

---

### Software Engineer @ Community.com

We're hiring!
---

## Me

- Plants
- Food
- Games
- Pets
- Community


---

## Drum Machine ?

- What
- Why 
- How

---

## What is a Drum Machine

Drum is hard, button is easy

---


![](img/drum_machine_old.jpg)

"My heart's beating like an 808" - Britney Spears

---

<!-- --- -->


♫ I wanna dance with somebody ♫ 

♫ When I get that feeling ♫ 

♫ So keep your love lockdown ♫

♫ I love the way you move ♫

♫ Music makes you lose control ♫ 

♫ Beat it ♫



---

![](img/my_drum_machine.png)

---

![](img/boots_cats.gif)

---

## Hardware

<img src="/img/rpi3b.jpeg" style="width: 30%; padding: 10px"/>
<img src="/img/powersupply.png" style="width: 30%; padding: 10px" />
<img src="/img/speaker.jpg" style="width: 30%; padding: 10px" />
<img src="/img/microsdcard.png" style="width: 30%; padding: 10px" />
<img src="/img/7in.jpeg" style="width: 30%; padding: 10px" />

---

![](img/drum_machine.png)

---

## Software

- Scenic
- midi_synth
- nerves_system_rpi3_fluidsynth


---

## Meatware

- Yourself

---

## Getting Started

- Getting Started With Nerves in Scenic docs
- Plug and play

---

![](img/scenic_starter.png)


---


## Primitives & Components

#### Primitives

- Text
- Rectangle 
- Image
- etc...


#### Components

- Button
- Slider
- Drop Down
- etc...

---
## Graph

```elixir
@graph Scenic.Graph.build()
|> text("This is some text", translate: {20, 20})
```


![](img/text.png)

---

## Header

![](img/header.png)

---

## Scenic UI Example 1/5

![](img/vol_controls.png)


---

## Scenic UI Example 2/5

```elixir
    button(graph, "-",
      theme: %{...} # map containing the style
      id: :volume_down, # id, like an html id
      t: {40, 70}, # The position
      height: 70,
      width: 70
    )
```
---

## Scenic UI Example 3/5

```elixir
def init(_, _opts) do
  state = %{
    graph: @graph
  }

  {:ok, state, push: state.graph}
end
```

---

## Scenic UI Example 4/5

```elixir
def filter_event({:click, :volume_down}, _context, state) do
```
---

## Scenic UI Example 5/5

```elixir
root_graph
|> VolumeControls.add_to_graph()
```

---

## Beat Indicator

![](img/stepindicator.png)


---

## Control Panel


![](img/cpane.png)


---

## The Loop 1/2

  - Process.send_after(self(), :loop, next_loop_milliseconds)
  - Play active audio
  - Let other components know a loop occured so they can update their UI
  

---

## The Loop 2/2

<p style="color: red; font-style: italic">Alert!</p>

---
## Events & Communicating between components 1/2

- Component is a specialized Scene
- Scene is a specialized Genserver
---

## Events & Communicating between components 2/2

![](img/boots01.png)

---
## Optimize CPU usage 1/3

- Benchmark
- Follow best practices
---

## Optimize CPU usage 2/3

- Cache pure functions at compile time by generating function heads that return the precalculated value
  - Top 2x faster, 0 mem usage vs 64B
```elixir
  for n <- 0..100 do
      @str "volume is #{n}"
      def volume_string(unquote(n)) do
        @str
      end
  end
```
vs
```elixir
  def volume_string(n) do
    "volume is #{n}"
  end
```
Thank you Bryan Joseph for helping me accomplish this

---

```elixir
    for n <- 0..100 do
        @str "volume is #{n}"
        def volume_string(unquote(n)) do
          @str
        end
    end
```
☝️ Generates👇
```elixir
    def volume_string(0) do
      "volume is 0"
    end
    def volume_string(1) do
      "volume is 1"
    end
    def volume_string(2) do
      "volume is 2"
    end
    ...
    def volume_string(100) do
      "volume is 100"
    end
```
And ☝️ is 2-3x faster than 👇
```elixir
    def volume_string(n), do: "volume is #{n}"
```
and
```elixir
    def volume_string(n), do: "volume is " <> to_string(n)
```
---

## Optimize CPU usage 3/3

- nifs
![](img/great_power.jpeg)

"With great power comes great responsibility" - Uncle Ben, Spider-Man (2002)

---

# Potential improvements

- Robotic arms
- SchedEx
---

# In Conclusion


<img src="img/1404231.png" ></img>


---

# Demo

- It's from my macbook not a raspberry pi sorry

---

## Questions



