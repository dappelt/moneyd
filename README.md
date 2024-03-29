# Moneyd
> ILP-enable your machine!

- [Description](#description)
- [Usage](#usage)
  - [Remote Deploy](#remote-deploy)
  - [Settlement](#settlement)
- [Try it Out](#try-it-out)

## Description

This repo contains an experimental ILP provider, allowing all applications on
your computer to use funds on the _live_ ILP network.

It works by creating a payment channel to an Interledger connector, and then
running `ilp-plugin-mini-accounts` locally. Any plugin can connect to this
mini-accounts instance by generating a random secret and authenticating via BTP
to `localhost:7768`. By default, only connections from localhost are accepted.

The `ilp-plugin` repo is already designed to do this, so `ilp-curl` and many
other tools will work right out of the box.

Because it's in early stages, don't use it
with a ripple account that has too much money.

## Usage

You'll need:

- A computer (or remote server) with node 8
- The secret for a funded XRP account
- The BTP host of an `ilp-plugin-xrp-asym-server` instance. You can find a suitable
  parent connector on [ConnectorLand](https://connector.land).

The moneyd package exposes a script for managing your moneyd instance. On your
own machine (or on a rented server), install node 8 and run:

```sh
npm install -g moneyd
moneyd start --secret "s..." --parent "example.com"
```

Replace the `s...` and `example.com` with your XRP secret and parent's BTP
host, respectively. Moneyd will then launch in your terminal, and run its server
on `localhost:7768`. _(TODO: Daemonize moneyd)_

Give moneyd a minute or so to do first-time setup. It will create a payment channel
to your parent connector, and then the parent connector will open a payment channel
back to you.

### Remote Deploy

If you did the previous step on your remote server, then you don't need to run any
special software to get moneyd on your local machine. Not only that, but you can
grant access to Interledger to as many machines as you want!

Just forward the moneyd port `7768` to any machine where you want ILP access by
using SSH local port forwarding:

```sh
ssh -N -L 7768:localhost:7768 user@example.com
```

Replace the `user@example.com` with the server on which you're running moneyd.

### Settlement

If you crash or encounter a bug, you might find that your moneyd instance forgot
to send a claim to its parent connector. This results in the parent connector thinking
you owe it money, and refusing to forward any of your packets.

To fix this, just stop moneyd and run:

```
moneyd settle --secret "s..." --parent "example.com" --amount 1000
```

You can adjust the amount if you need to reconcile more. The amount is
represented in XRP drops; 1.000.000 is a single XRP so these amounts are
typically kept quite small.

## Try it out

Now that you have moneyd running, you can test it out by uploading a file to unhash.
Unhash is a simple content-addressed file upload service based on ILP.

You'll use [ILP Curl](https://github.com/interledgerjs/ilp-curl), which will connect
to moneyd and send money to the unhash host.

```sh
npm install -g ilp-curl
echo "This is my example file" > example.txt
ilp-curl -X POST https://alpha.unhash.io/upload --data @example.txt
# --> {"digest":"ff5574cef56e644f3fc4d0311b15a3e95f115080bcc029889f9e32121fd60407"}
curl https://alpha.unhash.io/ff5574cef56e644f3fc4d0311b15a3e95f115080bcc029889f9e32121fd60407
# --> "This is my example file"
```

Now you've successfully sent an ILP payment to pay for a file upload! You can
browse [Interledgerjs on Github](https://github.com/interledgerjs) to find more
use cases.
