# Buildpack to run Etcd Server

This project is primarily a buildpack to run a Etcd Server; but also includes an internal asset-buildpack for compiling Etcd source into executables.

Currently the buildpack includes etcd 0.3.0.

## Demo Usage

Create a new project with a `etcd-version` file containing either `0.3.0`:

```
mkdir -p /tmp/`whoami`-etcd-service
cd /tmp/`whoami`-etcd-service
echo "0.3.0" > etcd-version
```

Now deploy to your favourite buildpack-supporting PaaS.

For Cloud Foundry:

```
cf push `whoami`-etcd-service -b https://github.com/drnic/etcd-service-buildpack.git
```

And later delete with:

```
cf delete `whoami`-etcd-service
```

For Heroku:

```
git init
git add .
git commit -m "v0.3.0 etcd service"
heroku create `whoami`-etcd-service -b https://github.com/drnic/etcd-service-buildpack.git
git push heroku master
```

And later destroy with:

```
heroku apps:destroy `whoami`-etcd-service --confirm `whoami`-etcd-service
```

## Development

There are two stages of development:

* Adding & compiling new versions of Etcd
* Developing the buildpack (`bin/compile`, `bin/detect`, `bin/release`)

For each new version of Etcd supported, the `config/slugs` file is updated with the accessible URL of the pre-compiled version of etcd.

### Adding & compiling new versions of Etcd

Extend the `bin/fetch_assets` script to fetch the new etcd src asset. Then run it to fetch it (and all other etcd versions):

```
./bin/fetch_assets
```

Now modify `etcd-compiler/bin/compile` for the new version Etcd (in future this will be beautifully automated) in the blobs folder.

To create the pre-compiled slugs:

```
gem install anvil-cli
anvil build . -b etcd-compiler -p
```

The last line is the published slug URL.

Now add this into `config/slugs` so buildpack users will have access to the pre-compiled version of Etcd.

### Developing the buildpack

You can also use `anvil` to do some basic development/test of the buildpack itself (the files `bin/compile`, `bin/detect`, `bin/release`).

```
anvil build testapp -b . -p
...
https://api.anvilworks.org/slugs/93f4b978-6187-4ce4-affa-64ffea110c14.tgz
```

The last line will be the slug URL. If you download and unpack it, you will see something like:

```
$ tree
.
├── Procfile
├── bin
│   ├── configure-and-run
│   ├── fetch_blobs
│   ├── etcd-benchmark
│   ├── etcd-check-aof
│   ├── etcd-check-dump
│   ├── etcd-cli
│   └── etcd-server
├── etc
│   └── etcd.conf
├── etcd-compiler
│   └── bin
│       ├── compile
│       └── detect
└── etcd-version
```

The `etcd-version` file comes from the built-in `testapp` used above. Everything else is included from the buildpack. `bin/etcd-server` contains the compiled Etcd server.

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
mkdir /tmp/app
cd /tmp/app
wget https://api.anvilworks.org/slugs/93f4b978-6187-4ce4-affa-64ffea110c14.tgz
tar xfz 93f4b978-6187-4ce4-affa-64ffea110c14.tgz
cat Procfile
```

Either run the process with foreman/goreman or try manually:

```
export PORT=5656
bin/etcd -name etcd-test -addr 0.0.0.0:$PORT -peer-addr 0.0.0.0:7001 -data-dir data -snapshot true
```

## Credits

The buildpack portion was based on 
Based on https://github.com/dpiddy/heroku-buildpack-etcd
