.. usage:

Quickstart
==========

Imports
-------

The public API is exported by `pyroute2/__init__.py`. There
are two main reasons for such approach.

First, it is done so to provide a stable API, that will not
be affected by changes in the package layout. There can be
significant layout changes between versions, but if a
symbol is re-exported via `pyroute2/__init__.py`, it will be
available with the same import signature.

All other objects are also available for import, but they
can change the signature in the next versions.

Another function of `pyroute2/__init__.py` is to provide
deferred imports. Being imported from the root of the
package, classes will be really imported only with the first
constructor call. This make possible to change the base
of pyroute2 classes on the fly. E.g., this way the eventlet
environment support is done.

E.g.::

    # Case #1
    #
    # Import a pyroute2 class directly. In the next versions
    # the import signature can be changed, e.g., NetNS from
    # pyroute2.netns.nslink it can be moved somewhere else.
    #
    from pyroute2.netns.nslink import NetNS  # <- real import
    ns = NetNS('test')

    # Case #2
    #
    # Import the same class from root module. This signature
    # will stay the same, any layout change is reflected in
    # the root module.
    #
    from pyroute2 import NetNS
    ns = NetNS('test')  # <- real import will be done here


The proxy class, used in the second case, supports correct
`isinstance()` and `issubclass()` semantics, and in both
cases the code will work in the same way.

Runtime
-------

In the runtime pyroute2 socket objects behave as normal
sockets. One can use them in the poll/select, one can
call `recv()` and `sendmsg()`::

    from pyroute2 import IPRoute

    # create RTNL socket
    ipr = IPRoute()

    # subscribe to broadcast messages
    ipr.bind()

    # wait for data (do not parse it)
    data = ipr.recv(65535)

    # parse received data
    messages = ipr.marshal.parse(data)

    # shortcut: recv() + parse()
    #
    # (under the hood is much more, but for
    # simplicity it's enough to say so)
    #
    messages = ipr.get()


But pyroute2 objects have a lot of methods, written to
handle specific tasks::

    from pyroute2 import IPRoute
    from pyroute2 import IW

    # RTNL interface
    ipr = IPRoute()

    # WIFI interface
    iw = IW()

    # get devices list
    ipr.get_links()

    # scan WIFI networks on wlo1
    iw.scan(ipr.link_lookup(ifname='wlo1'))


More info on specific modules is written in the next
chapters.

Resource release
----------------

Do not forget to release resources and close sockets. Also
keep in mind, that the real fd will be closed only when the
Python GC will collect closed objects.

Special cases
=============

eventlet
--------

The eventlet environment conflicts in some way with socket
objects, and pyroute2 provides a workaround for that::

    # import symbols
    #
    import eventlet
    from pyroute2 import IPRoute
    from pyroute2.config.eventlet import eventlet_config

    # setup the environment
    eventlet.monkey_patch()
    eventlet_config()

    # run the code
    ipr = IPRoute()
    ipr.get_routes()
    ...

The `eventlet_config()` call changes the base class of the
`IPRoute`, but with the deferred import the code meets the
PEP8 requirements.
