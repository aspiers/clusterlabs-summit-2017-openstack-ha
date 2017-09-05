<!-- .slide: data-state="section-break" id="terminology" data-timing="15" -->
# Brief interlude: `nova evacuate`

Note:
This is a good time to introduce `nova evacuate`.


<!-- .slide: data-state="normal" id="evacuate-architecture" data-menu-title="nova's recovery API" class="architecture" data-timing="30" -->
## `nova`'s recovery API

<div class="architecture">
    <img alt="Architecture with pacemaker_remote"
         class="architecture"
         data-src="images/standard-architecture.svg" />
    <span class="fragment" data-fragment-index="1">
        <img class="fragment fade-out compute-node bang"
             data-fragment-index="2"
             alt="compute node explosion!"
             data-src="images/explosion.svg" />
    </span>
    <img class="fragment fence"
         data-fragment-index="2"
         alt="fencing dead compute node"
         data-src="images/cross.svg" />
    <img alt="Architecture use of evacuate API"
         data-fragment-index="3"
         class="evacuate-api-arrow fragment"
         data-src="images/standard-architecture-evacuate-API-arrow.svg" />
    <img alt="Architecture use of evacuate API"
         data-fragment-index="3"
         class="evacuate-api-arrow fragment"
         data-src="images/standard-architecture-evacuate-API-arrow.svg" />
    <img class="fragment migration"
         data-fragment-index="4"
         alt="resurrecting dead VMs elsewhere"
         data-src="images/migration-arrow.svg" />
</div>

Note:
*   If we have a compute node failure, after fencing the node,
    we need to resurrect the VMs in a way which OpenStack is aware of.
*   Luckily `nova` provides an API for doing this, which is called
    `nova evacuate`.  So we just call that API and `nova` takes care
    of the rest.
*   Without shared storage, simply rebuilds from scratch


<!-- .slide: data-state="normal" id="nova-evacuate" --
## `nova evacuate`

*   API provided by `nova` for initiating recovery of VM
*   http://docs.openstack.org/admin-guide/cli_nova_evacuate.html

```sh
# nova help evacuate
usage: nova evacuate [--password <password>] [--on-shared-storage]
                     <server> [<host>]

Evacuate server from failed host.
```

*   Used by most HA solutions
*   Without shared storage, simply rebuilds from scratch
-->
