# Summary documentation of dbus-cli

This is a reformatted copy of the old wiki page on google code.

## Introduction

dbus-cli (executable name: dbus) is a tool to send messages to existing DBus
services. It is like KDE's dcop (but for DBus) or dbus-send (but much more
use-friendly)


## General options
First, you have the well-know --help (-h). No need to explain that, right ?
By default, dbus-cli work with the session bus. To work with the system bus, use the --system-bus (-y) option.

Since the goal of dbus is to call methods and not to send signals, signals in introspection are just ignored. You can reverse this with --signals (-s)


## Listing services
To get a list of services, just type

    $ dbus

without argument (but you can specify options of course ;))

By default, dbus-cli introspects the services to see if it has a reachable
object containing at least one method (or signal if you used --signals). This
can cause problems (see "known problems" below) so you can turn this off with
--empty (-e).

dbus-cli don't list services starting with a ':': a service starting with ':'
is likely a "client" and not a server and won't responds to introspection
messages. You can turn this off with --unnamed (-u) but use --empty (-e) or
dbus-cli will timeout on all of these services.

## Listing objects

You found the service you'll play with ? Fine, so we have to find usable
objects on this service. Nothing easier, just use. We'll take
org.gnome.feed.Reader (liferea) for example.

    $ dbus org.gnome.feed.Reader

This will list usable objects. As above, dbus-cli will try to see if an object
has at least one method, else it won't show you this object. To turn this off,
use -e

## Listing methods

Well, we have a fun service (org.gnome.feed.Reader), a cool object
(/org/gnome/feed/Reader), let's see what we can do with:

    $ dbus org.gnome.feed.Reader /org/gnome/feed/Reader

This will list all methods you can apply to this object, with the signature.
Now, you just have to call this method.

## Calling methods

For example, if you want to say to Liferea that you will soon be disconnected,
you can use:

    $ dbus org.gnome.feed.Reader /org/gnome/feed/Reader org.gnome.feed.Reader.SetOnline %False

or faster

    $ dbus org.gnome.feed.Reader /org/gnome/feed/Reader SetOnline %False

Note the '%' before the False. That mean that following is not a string but a
python expression that will be evaluated. That's right, dbus-cli don't (yet)
convert arguments according to introspected signature and you have to give the
right type.

## Known problems

 * One bug : when you try to introspect services which aren't services (I mean prograls using DBus as client but not as service), dbus-cli fail after 30 seconds. That mean that on the services list, you have to wait 30*(nb of non-services) seconds ! We need to decrease the timeout and parallel introspection

 * One missing feature : you have to explicitly give the type of each argument. (%False, %0 instead of just False or 0)

 * Bogus services like NetworkManager went crazy when we try to introspect them (just try "dbus -y -e org.freedesktop.NetworkManager /org/freedesktop/NetworkManager/Devices org.freedesktop.DBus.Introspectable.Introspect")

 * Bogus services give us non-existent object paths. For example, HAL:

     $ dbus -ye org.freedesktop.Hal /org/freedesktop/Hal/devices Introspect
     ...
     <node name="DVD_ROM_SBW242B"/>
     ...
     $ dbus -ye org.freedesktop.Hal /org/freedesktop/Hal/devices/DVD_ROM_SBW242B Introspect
     ...
     dbus.DBusException: org.freedesktop.Hal.NoSuchDevice: No device with id /org/freedesktop/Hal/devices/DVD_ROM_SBW242B

## ZSH completion

For those who have a good shell, a ZSH completion script is also included
