# Cronex

> [!CAUTION]
> This development is old, and Elixir changed a lot so I decided to teach you how Elixir could do this
> instead of doing that using a dependency. Keep your code with as few dependencies as possible!

## Using Elixir instead of Cronex

Of course, getting a similar cron like it's in Linux isn't as easy doing that by yourself, but
are you really need it? Check this code:

```elixir
defmodule MyApp.MyModule do
  use GenServer

  def start_link([]), do: GenServer.start_link(__MODULE__, [], name: __MODULE__)

  @impl GenServer
  @doc false
  def init([]) do
    Process.send_after(self(), :timeout, 60_000)
    {:ok, []}
  end

  @impl GenServer
  @doc false
  def handle_info(:timeout, state) do
    Process.send_after(self(), :timeout, 60_000)
    # this is running every minute so, you can check the NaiveDateTime.utc_now()
    # and see what you need to run now.
    {:noreply, state}
  end
end
```

So, I recommend you to do it in this way and avoid dependencies for something that's very simple.
And you can get this code as example or build your own based on `:timer.apply_after` for avoiding
the creation of the GenServer, or even using Task inside of a supervisor where it could respawn
a new task and wait for some time previously to start. It's on your own, use what you need and
evaluate different options.

---

A cron like system, built in Elixir, that you can mount in your supervision tree.

Cronex's DSL for adding cron jobs is inspired by [whenever](https://github.com/javan/whenever) Ruby gem.

## Installation

Add `cronex` to your list of dependencies in `mix.exs`:

```elixir
def deps do
  [{:cronex, "~> 0.4.0"}]
end
```

Then run `mix deps.get` to get the package.

## Getting started

Cronex makes it really easy and intuitive to schedule cron like jobs.

You use the `Cronex.Scheduler` module to define a scheduler and add jobs to it.

Cronex will gather jobs from the scheduler you defined and will run them at the expected time.

```elixir
# Somewhere in your application define your scheduler
defmodule MyApp.Scheduler do
  use Cronex.Scheduler

  every :hour do
    IO.puts "Every hour job"
  end

  every :day, at: "10:00" do
    IO.puts "Every day job at 10:00"
  end

  every :monday, at: "10:00" do
    IO.puts "Monday at 10:00"
  end

  every [:friday, :saturday], at: "20:00" do
    IO.puts "Party time! Weekend!"
  end
end

# Start scheduler with start_link
MyApp.Scheduler.start_link

# Or add it to your supervision tree
defmodule MyApp.Supervisor do
  use Supervisor

  # ...

  def init(_opts) do
    children = [
      # ...
      supervisor(MyApp.Scheduler, [])
      # ...
    ]

    supervise(children, ...)
  end

  # ...
end
```

You can define as much schedulers as you want.

## Testing

Cronex comes with `Cronex.Test` module which provides helpers to test your cron jobs.

```elixir
defmodule MyApp.SchedulerTest do
  use ExUnit.Case
  use Cronex.Test

  test "every hour job is defined in MyApp.Scheduler" do
    assert_job_every :hour, in: MyApp.Scheduler 
  end

  test "every day job at 10:00 is defined in MyApp.Scheduler" do
    assert_job_every :day, at: "10:00", in: MyApp.Scheduler 
  end
end
```

## Documentation

The project documentation can be found [here](https://hexdocs.pm/cronex/api-reference.html).

## Contributing

Bug reports and pull requests are welcome on GitHub at https://github.com/jbernardo95/cronex.

## License

Cronex source code is licensed under the [MIT License](LICENSE.md).
