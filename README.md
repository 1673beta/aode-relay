# AodeRelay
_A simple and efficient activitypub relay_

### Installation
#### Docker
If running docker, you can start the relay with the following command:
```
$ sudo docker run --rm -it \
    -v "./:/mnt/" \
    -e ADDR=0.0.0.0 \
    -e SLED_PATH=/mnt/sled/db-0.34 \
    -p 8080:8080 \
    asonix/relay:0.3.23
```
This will launch the relay with the database stored in "./sled/db-0.34" and listening on port 8080
#### Cargo
With cargo installed, the relay can be installed to your cargo bin directory with the following command
```
$ cargo install ap-relay
```
Then it can be run with this:
```
$ ADDR=0.0.0.0 relay
```
This will launch the relay with the database stored in "./sled/db-0.34" and listening on port 8080
#### Source
The relay can be launched directly from this git repository with the following commands:
```
$ git clone https://git.asonix.dog/asonix/relay
$ ADDR=0.0.0.0 cargo run --release
```

### Usage
To simply run the server, the command is as follows
```bash
$ ./relay
```

To learn about any other tasks, the `--help` flag can be passed
```bash
An activitypub relay

Usage: relay [OPTIONS]

Options:
  -b <BLOCKS>       A list of domains that should be blocked
  -a <ALLOWED>      A list of domains that should be allowed
  -u, --undo        Undo allowing or blocking domains
  -h, --help        Print help information
```

To add domains to the blocklist, use the `-b` flag and pass a list of domains
```bash
$ ./relay -b asonix.dog blimps.xyz
```
To remove domains from the blocklist, simply pass the `-u` flag along with `-b`
```bash
$ ./relay -ub asonix.dog blimps.xyz
```
The same rules apply for allowing domains, although domains are allowed with the `-a` flag
```bash
$ ./relay -a asonix.dog blimps.xyz
$ ./relay -ua asonix.dog blimps.xyz
```

### Configuration
By default, all these values are set to development values. These are read from the environment, or
from the `.env` file in the working directory.
```env
HOSTNAME=localhost:8080
ADDR=127.0.0.1
PORT=8080
DEBUG=true
RESTRICTED_MODE=false
VALIDATE_SIGNATURES=false
HTTPS=false
PRETTY_LOG=true
PUBLISH_BLOCKS=false
SLED_PATH=./sled/db-0.34
```
To run this server in production, you'll likely want to set most of them
```env
HOSTNAME=relay.my.tld
ADDR=0.0.0.0
PORT=8080
DEBUG=false
RESTRICTED_MODE=false
VALIDATE_SIGNATURES=true
HTTPS=true
PRETTY_LOG=false
PUBLISH_BLOCKS=true
SLED_PATH=./sled/db-0.34
RUST_LOG=warn
OPENTELEMETRY_URL=localhost:4317
TELEGRAM_TOKEN=secret
TELEGRAM_ADMIN_HANDLE=your_handle
```

#### Descriptions
##### `HOSTNAME`
The domain or IP address the relay is hosted on. If you launch the relay on `example.com`, that would be your HOSTNAME. The default is `localhost:8080`
##### `ADDR`
The address the server binds to. By default, this is `127.0.0.1`, so for production cases it should be set to `0.0.0.0` or another public address.
##### `PORT`
The port the server binds to, this is `8080` by default but can be changed if needed.
##### `DEBUG`
Whether to print incoming activities to the console when requests hit the /inbox route. This defaults to `true`, but should be set to `false` in production cases. Since every activity sent to the relay is public anyway, this doesn't represent a security risk.
##### `RESTRICTED_MODE`
This setting enables an 'allowlist' setup where only servers that have been explicitly enabled through the `relay -a` command can join the relay. This is `false` by default. If `RESTRICTED_MODE` is not enabled, then manually allowing domains with `relay -a` has no effect.
##### `VALIDATE_SIGNATURES`
This setting enforces checking HTTP signatures on incoming activities. It defaults to `false` but should be set to `true` in production scenarios
##### `HTTPS`
Whether the current server is running on an HTTPS port or not. This is used for generating URLs to the current running relay. By default it is set to `false`, but should be `true` in production scenarios.
##### `PUBLISH_BLOCKS`
Whether or not to publish a list of blocked domains in the `nodeinfo` metadata for the server. It defaults to `false`.
##### `SLED_PATH`
Where to store the on-disk database of connected servers. This defaults to `./sled/db-0.34`.
##### `RUST_LOG`
The log level to print. Available levels are `ERROR`, `WARN`, `INFO`, `DEBUG`, and `TRACE`. You can also specify module paths to enable some logs but not others, such as `RUST_LOG=warn,tracing_actix_web=info,relay=info`
##### `SOURCE_REPO`
The URL to the source code for the relay. This defaults to `https://git.asonix.dog/asonix/relay`, but should be changed if you're running a fork hosted elsewhere.
##### `OPENTELEMETRY_URL`
A URL for exporting opentelemetry spans. This is mostly useful for debugging. There is no default, since most people probably don't run an opentelemetry collector.
##### `TELEGRAM_TOKEN`
A Telegram Bot Token for running the relay administration bot. There is no default.
##### `TELEGRAM_ADMIN_HANDLE`
The handle of the telegram user allowed to administer the relay. There is no default.

### Subscribing
Mastodon admins can subscribe to this relay by adding the `/inbox` route to their relay settings.
For example, if the server is `https://relay.my.tld`, the correct URL would be
`https://relay.my.tld/inbox`.

Pleroma admins can subscribe to this relay by adding the `/actor` route to their relay settings. For
example, if the server is `https://relay.my.tld`, the correct URL would be
`https://relay.my.tld/actor`.

### Supported Activities
- Accept Follow {remote-actor}, this is a no-op
- Reject Follow {remote-actor}, an Undo Follow is sent to {remote-actor}
- Announce {anything}, {anything} is Announced to listening servers
- Create {anything}, {anything} is Announced to listening servers
- Follow {self-actor}, become a listener of the relay, a Follow will be sent back
- Follow Public, become a listener of the relay
- Undo Follow {self-actor}, stop listening on the relay, an Undo Follow will be sent back
- Undo Follow Public, stop listening on the relay
- Delete {anything}, the Delete {anything} is relayed verbatim to listening servers
    Note that this activity will likely be rejected by the listening servers unless it has been
    signed with a JSON-LD signature
- Update {anything}, the Update {anything} is relayed verbatim to listening servers
    Note that this activity will likely be rejected by the listening servers unless it has been
    signed with a JSON-LD signature

### Supported Discovery Protocols
- Webfinger
- NodeInfo

### Contributing
Feel free to open issues for anything you find an issue with. Please note that any contributed code will be licensed under the AGPLv3.

### License
Copyright © 2022 Riley Trautman

AodeRelay is free software: you can redistribute it and/or modify it under the terms of the GNU General Public License as published by the Free Software Foundation, either version 3 of the License, or (at your option) any later version.

AodeRelay is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. This file is part of AodeRelay.

You should have received a copy of the GNU General Public License along with AodeRelay. If not, see [http://www.gnu.org/licenses/](http://www.gnu.org/licenses/).
