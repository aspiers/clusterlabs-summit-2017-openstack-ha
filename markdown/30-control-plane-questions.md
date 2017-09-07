<!-- .slide: data-state="section-break" id="control-plane-question-section" data-menu-title="Control plane questions" data-timing="5" -->
# Question time


<!-- .slide: data-state="normal" id="control-plane-poll" data-timing="40" -->
# Quick poll

### How's control plane HA going for you so far?
<!-- .element: style="font-size: 1.4em" -->

<div class="face fragment">&#128516;</div>
<div class="face fragment">&#128530;</div>
<div class="face fragment">&#128561;</div>

Note:
- emoticon is "face screaming in fear"


<!-- .slide: data-state="normal" id="control-plane-questions" data-menu-title="Question menu" data-timing="40" -->
# Questions relating to control plane

*   <!-- .element: class="fragment" -->
    Shouldn't we use an active-active HAProxy architecture?
*   <!-- .element: class="fragment" -->
    How should we handle stateless active-active API services?
*   <!-- .element: class="fragment" -->
    Should we tackle Pacemaker phobia?  If so, how?
*   <!-- .element: class="fragment" -->
    Should we start moving resources out of core cluster onto remotes?
*   <!-- .element: class="fragment" -->
    Does Pacemaker need better maintenance capabilities?
*   <!-- .element: class="fragment" -->
    How do we handle multi-site clouds?


<!-- .slide: data-state="normal" id="active-active-haproxy" data-menu-title="A/A HAProxy" data-timing="40" -->
## Fixing the HAProxy bottleneck

Standard active-passive HAProxy architecture:

*   One VIP
*   One HAProxy always takes all the load

### Instead, why not have:
<!-- .element: class="fragment" data-fragment-index="1" -->

*   <!-- .element: class="fragment" data-fragment-index="1" -->
    One VIP per HAProxy node
*   <!-- .element: class="fragment" data-fragment-index="2" -->
    VIPs float as before
*   <!-- .element: class="fragment" data-fragment-index="3" -->
    DNS round-robin between them

Any downsides?
<!-- .element: class="fragment" data-fragment-index="4" -->


<!-- .slide: data-state="normal" id="control-plane-api-1" data-menu-title="OCF RAs" data-timing="40" -->
## How should we handle API services?

### Option 1: Use OCF RAs

https://launchpad.net/openstack-resource-agents

*   <!-- .element: class="fragment" data-fragment-index="1" -->
    Pros
    *   Fairly robust functional monitoring (in theory)
*   <!-- .element: class="fragment" data-fragment-index="2" -->
    Cons
    *   <!-- .element: class="fragment" data-fragment-index="2" -->
        Duplicates a lot of logic&nbsp;/ data already packaged by vendor
        *   location of binary&nbsp;/ pid&nbsp;/ config file,
            start parameters, uid&nbsp;/ guid,&nbsp;…
    *   <!-- .element: class="fragment" data-fragment-index="3" -->
        Layer boundary violation — requires knowledge of packaging
    *   <!-- .element: class="fragment" data-fragment-index="4" -->
        Monitoring logic probably duplicates probes from "real"
        monitoring systems (Nagios, Sensu,&nbsp;…)
    *   <!-- .element: class="fragment" data-fragment-index="5" -->
        Outdated assumptions (e.g. permanent DB connection)
    *   <!-- .element: class="fragment" data-fragment-index="6" -->
        Maintained by me

Note:

- Health check can actually query the API and check that it's responding,
  rather than just checking whether the pid exists


<!-- .slide: data-state="normal" id="control-plane-api-2" data-menu-title="Pacemaker + systemd" data-timing="40" -->
## How should we handle API services?

### Option 2: Pacemaker `systemd:` resources

Pacemaker auto-restarts service on crash

*   <!-- .element: class="fragment" data-fragment-index="1" -->
    Pros
    *   Fairly easy to set up cluster
    *   <!-- .element: class="fragment" data-fragment-index="2" -->
        Poor man's failure notifications for free via Pacemaker UIs
*   <!-- .element: class="fragment" data-fragment-index="3" -->
    Cons
    *   <!-- .element: class="fragment" data-fragment-index="4" -->
        **Does not handle malfunctioning services, only crashing
        ones**
        <!-- .element: class="fg-bright-red" -->


<!-- .slide: data-state="normal" id="control-plane-api-3" data-menu-title="systemd only" data-timing="40" -->
## How should we handle API services?

### Option 3: `systemd` only, no Pacemaker, as per Red Hat

*   http://blog.clusterlabs.org/blog/2016/next-openstack-ha-arch
*   `systemd` auto-restarts service on crash

*   <!-- .element: class="fragment" data-fragment-index="1" -->
    Pros
    *   Simplifies cluster
    *   Prepares for containerized future
*   <!-- .element: class="fragment" data-fragment-index="2" -->
    Cons
    *   <!-- .element: class="fragment" data-fragment-index="2" -->
        Assumes all services can robustly tolerate dependencies going down
        <!-- .element: class="fg-medium-dark-neutral" -->
    *   <!-- .element: class="fragment" data-fragment-index="3" -->
        Failures not surfaced via UI &rarr;
        requires separate monitoring&nbsp;/ alerting component
        <!-- .element: class="fg-medium-dark-neutral" -->
    *   <!-- .element: class="fragment" data-fragment-index="4" -->
        **Does not handle malfunctioning services, only crashing
        ones**
        <!-- .element: class="fg-bright-red" -->


<!-- .slide: data-state="normal" id="control-plane-api-4" data-menu-title="systemd health checks" data-timing="40" -->
## How should we handle API services?

### Option 4: Enhance `systemd` to allow health checks

*   <!-- .element: class="fragment" -->
    Pros
    *   Eliminates all duplication
    *   Cluster configuration is trivial
*   <!-- .element: class="fragment" -->
    Cons
    *   Requires `systemd` development


<!-- .slide: data-state="normal" id="control-plane-api-5" data-menu-title="OCF wrapping systemd" data-timing="40" -->
## How should we handle API services?

### Option 5: OCF RAs wrapping `systemctl` / `service(8)`

*   `start` / `stop` / `status` delegated to `systemd`
*   `monitor` implemented in OCF RA
    *   (or better, delegate to script reusable by other monitoring
        frameworks)

*   <!-- .element: class="fragment" -->
    Pros
    *   Eliminates most duplication between RA and package
    *   Clean separation of concerns (monitoring vs. control)
*   <!-- .element: class="fragment" -->
    Cons
    *   Cluster configuration remains relatively complex
    *   Have to poll; can't interact with `systemd` via `dbus`
        signalling
    *   Would have to duplicate Pacemaker's logic for
        dealing with `systemd` quirks


<!-- .slide: data-state="normal" id="control-plane-api-6" data-menu-title="OCF / systemd hybrids" data-timing="40" -->
## How should we handle API services?

### Option 6: enhance Pacemaker to support `systemd:`&nbsp;/ `ocf:` hybrids

*   `start` / `stop` / `status` via `systemd`
*   `monitor` via OCF RA
    *   (or better, delegate to script reusable by other monitoring
        frameworks)

*   <!-- .element: class="fragment" -->
    Pros
    *   Eliminates most duplication between RA and package
    *   Clean separation of concerns (monitoring vs. control)
*   <!-- .element: class="fragment" -->
    Cons
    *   Cluster configuration remains relatively complex
    *   <!-- .element: class="fragment" -->
        More work for Ken&nbsp;…
        <span class="fragment">actually, not really!</span>


<!-- .slide: data-state="normal" id="control-plane-api-7" data-menu-title="systemd health checks" data-timing="40" -->
## How should we handle API services?

### Option 7: use undocumented `container` meta-attribute

Pair `systemd` resource with monitor-only OCF RA

*   <!-- .element: class="fragment" -->
    Pros
    *   Eliminates all duplication
    *   No modification to Pacemaker or `systemd` required
*   <!-- .element: class="fragment" -->
    Cons
    *   Cluster configuration remains relatively complex


<!-- .slide: data-state="normal" id="composable-roles" data-menu-title="Composable roles" data-timing="40" -->
## Should we start moving resources out of core cluster onto remotes?

http://blog.clusterlabs.org/blog/2016/composable-openstack-ha

*   <!-- .element: class="fragment" -->
    Pros:
    *   much more flexible architecture
    *   avoids need for multiple clusters
    *   avoids problems with cross-cluster ordering
*   <!-- .element: class="fragment" -->
    Cons
    *   liveness check reduced to depending on single TCP connection?

Can we auto-promote remotes to core to maintain quorum?
<!-- .element: class="fragment" -->


<!-- .slide: data-state="normal" id="maintenance" data-timing="40" -->
## Resource maintenance

*   Sometimes need to restart services (e.g. reload config)
*   <!-- .element: class="fragment" -->
    Can't put a single resource on a single node in maintenance mode
*   <!-- .element: class="fragment" -->
    Can only put a whole node or a whole service in maintenance mode

Can we do better?  Do we need to?
<!-- .element: class="fragment" -->


<!-- .slide: data-state="normal" id="multi-site" data-menu-title="Multi-site" data-timing="40" -->
## How do we handle multi-site clouds?

*   e.g. multiple regions, shared Keystone
*   <!-- .element: class="fragment" -->
    Keystone reads can be distributed across sites
*   <!-- .element: class="fragment" -->
    Keystone writes are more difficult
    *   Need to ensure they get directed to one site
    *   Can do this with HAProxy
*   <!-- .element: class="fragment" -->
    `booth` is obvious candidate for deciding master site
