# Buildpack to run Redis Server

This project is primarily a buildpack to run a Redis Server; but also includes an internal asset-buildpack for compiling Redis source into executables.

Currently the buildpack includes redis 2.6.16 and 2.8.6.

## Usage

Create a new project with a `redis-version` file containing either `2.6.16` or `2.8.6`:

```
mkdir -p my-redis-service
cd my-redis-service
echo "2.8.6" > redis-version
```

Now deploy to your favourite buildpack-supporting PaaS.

For Cloud Foundry:

```
cf push my-redis-service -b https://github.com/drnic/redis-service-buildpack.git
```

For Heroku:

```
heroku create my-redis-service -b https://github.com/drnic/redis-service-buildpack.git
```

## Development

There are two stages of development:

* Adding & compiling new versions of Redis
* Developing the buildpack (`bin/compile`, `bin/detect`, `bin/release`)

For each new version of Redis supported, the `config/slugs` file is updated with the accessible URL of the pre-compiled version of redis.

### Adding & compiling new versions of Redis

Extend the `bin/fetch_assets` script to fetch the new redis src asset. Then run it to fetch it (and all other redis versions):

```
./bin/fetch_assets
```

Now modify `redis-compiler/bin/compile` for the new version Redis (in future this will be beautifully automated) in the blobs folder.

To create the pre-compiled slugs:

```
gem install anvil-cli
anvil build . -b redis-compiler -p
```

The last line is the published slug URL.

Now add this into `config/slugs` so buildpack users will have access to the pre-compiled version of Redis.

### Developing the buildpack

You can also use `anvil` to do some basic development/test of the buildpack itself (the files `bin/compile`, `bin/detect`, `bin/release`).

```
anvil build testapp -b . -p
```

The last line will be the slug URL. If you download and unpack it, you will see something like:

```
$ tree
.
├── Procfile
├── bin
│   ├── configure-and-run
│   ├── fetch_blobs
│   ├── redis-benchmark
│   ├── redis-check-aof
│   ├── redis-check-dump
│   ├── redis-cli
│   └── redis-server
├── etc
│   └── redis.conf
├── redis-compiler
│   └── bin
│       ├── compile
│       └── detect
└── redis-version
```

The `redis-version` file comes from the built-in `testapp` used above. Everything else is included from the buildpack. `bin/redis-server` contains the compiled Redis server.

### Run locally

If you're not in Ubuntu 10.04, the use Vagrant:

```
vagrant box add lucid64 http://files.vagrantup.com/lucid64.box
vagrant init lucid64
vagrant up
vagrant ssh
```

Inside vagrant:

```
cd /vagrant
PORT=5555 ./bin/configure-and-run ./bin/redis-server etc/runtime.conf
```

## Credits

The buildpack portion was based on 
Based on https://github.com/dpiddy/heroku-buildpack-redis
