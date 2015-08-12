# CF v4

Replaces CF with a Concourse pipeline, relying on [ngrok](https://ngrok.io) for
routing.

## Usage

1. Create a Git repository for the [Pool
   resource](https://github.com/concourse/pool-resource). As a shortcut, you
   can just fork this repo. This repo should be public; no secrets will be
   going into it, and the load balancer needs to be able to fetch it without
   credentials to know which instances to your too.

1. Create an S3 bucket for storing the compiled app bits (droplets).

1. Create an account at [ngrok](https://ngrok.io). For a reserved subdomain
   you'll need to sign up for a paid account. The account type also determines
   how many instances of your app you can run.

1. Set the desired number of concurrent app instances as `max_in_flight` on the
   `run` job.

1. Set the `app-source` and `buildpack` resources to point at your app's source
   code and the appropriate buildpack, respectively. They default to
   [Dora](https://github.com/vito/dora) and the [Ruby
   buildpack](https://github.com/cloudfoundry/ruby-buildpack).

1. Configure the pipeline with the appropriate credentials provided for the
   various parameters. `fly configure` will yell at you until you provide them
   all, so just do that.

1. Trigger the `load-balancer` job to get things started. This will spin up an
   `nginx` server that will balance requests acros your instances.

1. Trigger `stage` to build a droplet. This will run automatically whenever
   your source bits change.

1. Trigger `run` to spin up an instance. It will eventually become routable.


## Features

1. Dynamic routing with zero downtime (though a small delay in taking effect).

1. Crash recovery: `hm9001` will ensure a build is enqueued for the
   `load-balancer` and `run` jobs, ensuring they meet their max-in-flight
   configuration within a few seconds.

1. Logging: just watch each instance of `run`.

1. Debugging: `fly hijack`.

1. True continuous delivery with zero downtime.

1. Versioning of your app. Disable a droplet version if it's going haywire.

1. Rolling (manual) deploys. Once new staged bits are available, spinning up
   the new version is just a matter of killing the old instances one-by-one and
   letting new instances come up. If things aren't going well on the new
   instances, just disable the new droplet and kill them.


## Anti-features

1. No proper HA/app instance balancing. They may end up all running on the same
   worker.

1. Pretty reliant on `ngrok` at the moment. If that goes down you're SOL. Also
   requires a paid account, but it's pretty cheap.
