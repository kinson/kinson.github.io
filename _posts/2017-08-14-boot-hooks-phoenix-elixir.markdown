---
layout: post
title:  "Deploying Phoenix Application with Boot Hooks"
date:   2017-08-14 16:42:38 -0500
categories: elixir boot distillery edeliver
---
## How to use distillery to run migrations on update

The goal of this post is two-fold, to write my first blog post that will hopefully help someone who winds up stuck in the same rut that I was in and to fill in some small holes in otherwise very helpful articles. The content and a lot of references in this post come from a [post](http://blog.firstiwaslike.com/elixir-deployments-with-distillery-running-ecto-migrations/) I used to figure this out!

Deploying with Phoenix is not straight forward but [distillery](https://github.com/bitwalker/distillery) and [edeliver](https://github.com/edeliver/edeliver) have made it a lot easier to manage releases and deployments without sacrificing the ability to use the hot upgrades feature. One feature which makes deployments smoother for me is automating database migrations on update - a feature that neither of these tools has out of the box.

*Boot Hooks*, however, do come out of the box and provide a means for doing any preparation or clean up at os level via an executable. Incorporating boot hooks requires changes in three places. Before continuing make sure you have distillery and edeliver installed and the config files for each created.

### Release Configuration Change
The first change happens in `rel/config.exs`.

```elixir
environment :dev do
  set dev_mode: true
  set include_erts: false
  set post_start_hook: "rel/hooks/post_start"
  ...
end


environment :prod do
  set include_erts: true
  set include_src: false
  set post_start_hook: "rel/hooks/post_start"
  ...
end
```

The post start hook is one of the boot hooks specified by distillery. A more complete list can be found here. These refer to executables in the specified directory. For this example we will be using shell scripts to begin the migration process.

### Execute Migration Task From Shell Script

To begin, create a new `rel/hooks` directory and create a file `post_start[.sh]` in it.

```sh
set +e

echo "Preparing to run migrations"

while true; do
  nodetool ping
  EXIT_CODE=$?
  if [ $EXIT_CODE -eq 0 ]; then
    echo "Application is up!"
    break
  fi
done

set -e

echo "Running migrations"
bin/seasaltearrings rpc Elixir.Release.Tasks migrate
echo "Migrations run successfully"
```

This script will execute after the Elixir process has begun so it takes advantage of erlang's [rpc](http://erlang.org/doc/man/rpc.html) module to execute a task on the running process. This leads us to the last step to setting up a boot hook with Distillery and that is creating a task module in Elixir. There is no task module with the phoenix template so you will have to add one and reference in `mix.exs`. I chose to put the tasks file in `release/tasks.ex` and added release to the compile path in `mix.exs`.

```elixir
# Specifies which paths to compile per environment.
 defp elixirc_paths(:test), do: ["lib", "test/support"]
 defp elixirc_paths(_),     do: ["lib", "release"]
 ...
 ```

 The module itself simply ensures the application is running and uses the ecto migrator module to ensure that all migrations in `priv/repo/migrations` have been completed.

 ```elixir
 defmodule Release.Tasks do
  def migrate do
    {:ok, _} = Application.ensure_all_started(:seasaltearrings)

    path = Application.app_dir(:seasaltearrings, "priv/repo/migrations")

    Ecto.Migrator.run(Seasaltearrings.Repo, path, :up, all: true)
  end
end
```

Now running `mix release` will generate a release that will automatically attempt to run migrations with `/bin my_app foreground` locally and on the staging and production host if you are using edeliver.

If you have made it all the way through this post I want to thank you and would welcome any feedback at me@samwhunter.com.
