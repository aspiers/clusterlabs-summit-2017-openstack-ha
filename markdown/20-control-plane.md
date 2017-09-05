<!-- .slide: data-state="section-break" id="control-plane" data-menu-title="Control plane HA" data-timing="5" -->
# HA in OpenStack's control plane


<!-- .slide: data-state="normal" id="typical-control-plane" class="diagram-and-list" data-timing="40" -->
## Typical HA control plane

<div class="diagrams">
    <img class="services" data-src="images/services-cluster.svg"
         alt="HA services cluster" />
    <img class="db-mq" data-src="images/DB-MQ-cluster.svg"
         alt="database and message queue cluster" />
</div>

*   1 or more clusters, possibly different sizes
*   <!-- .element: class="fragment" -->
    Stateless active-active API services
*   <!-- .element: class="fragment" -->
    Galera DB: active-passive / single master with replication
*   <!-- .element: class="fragment" -->
    RabbitMQ active-active
*   <!-- .element: class="fragment" -->
    [HAProxy](http://www.haproxy.org/) load-balancer
*   <!-- .element: class="fragment" -->
    [Pacemaker](http://clusterlabs.org/) monitors and controls nodes and services

<div class="solved stamp fragment">
    <p class="solved">SOLVED</p>
    <p class="mostly fragment">(mostly)</p>
</div>

Note:
- Sometimes DRBD used instead of shared storage


<!-- .slide: data-state="normal" id="neutron-L3" data-timing="5" data-menu-title="Neutron L3 HA" -->
# L3 HA in Neutron (networking service)

<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__CVR">
    <img alt="Neutron L3 HA" src="images/HPE_HA_Layer-3HA.png"/>
</a>


<!-- .slide: data-state="normal" id="neutron-L3-analysis" data-menu-title="L3 HA analysis" data-timing="5" -->
# L3 HA in Neutron — analysis

*   [`neutron` HA is tricky](https://youtu.be/vBZgtHgSdOY), but out of the
scope of this talk
*   See  https://ethercalc.openstack.org/Pike-Neutron-L3-HA


<!-- .slide: data-state="normal" id="control-plane-caveats" data-timing="5" -->
# Other caveats

*   <!-- .element: class="fragment" -->
    `cinder-volume` (block storage backend) active-active support still
    under development
*   <!-- .element: class="fragment" -->
    General phobia of Pacemaker in some parts of the OpenStack community
    *   `keepalived` often used instead for VIPs

Note:
- Will speculate later in talk on reasons for Pacemaker phobia


<!-- .slide: data-state="section-break" id="HAProxy-VIPs" data-timing="5" -->
# HAProxy and VIPs


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-1" data-menu-title="HAProxy 1" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 1" src="images/HPE_HA_Flow-1.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-2" data-menu-title="HAProxy 2" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 3" src="images/HPE_HA_Flow-2.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-3" data-menu-title="HAProxy 3" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 4" src="images/HPE_HA_Flow-3.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-4" data-menu-title="HAProxy 4" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 5" src="images/HPE_HA_Flow-4.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-5" data-menu-title="HAProxy 5" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 6" src="images/HPE_HA_Flow-5.png"/>
</a>

<!--
<div class="architecture">
    <span class="fragment" data-fragment-index="1">
        <img alt="masakari architecture"
             class="architecture fragment fade-out" data-fragment-index="2"
             data-src="images/HPE_HA_Flow-2.png" />
    </span>
    <span class="fragment" data-fragment-index="2">
        <img alt=""
             class="architecture fragment fade-out" data-fragment-index="3"
             data-src="images/HPE_HA_Flow-3.png" />
    </span>
    <span class="fragment" data-fragment-index="3">
        <img alt=""
             class="architecture fragment fade-out" data-fragment-index="4"
             data-src="images/HPE_HA_Flow-4.png" />
    </span>
    <span class="fragment" data-fragment-index="4">
        <img alt=""
             class="architecture fragment fade-out" data-fragment-index="5"
             data-src="images/HPE_HA_Flow-5.png" />
    </span>
</div>
-->


<!-- .slide: data-state="normal" id="RH-control-plane" data-menu-title="RH OSP" data-timing="40" -->
## Red Hat OpenStack Platform control plane

Pacemaker manages:

*   Galera and Redis (multi-state)
*   RabbitMQ (cloned)

Active-active <!-- .element: class="fragment" --> API services [now
controlled by `systemd`](http://blog.clusterlabs.org/blog/2016/next-openstack-ha-arch)

*   `systemd` does auto-restart on crash
*   Simplifies cluster
*   Prepares for containerized future

<!-- .element: class="fragment" -->

Note:
- [RH OSP 11 documentation](https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/11/html/understanding_red_hat_openstack_platform_high_availability/)
- VIPs for
    -   public IP
    -   controller (in undercloud dhcp range)
    -   APIs
    -   Redis
    -   storage (Glance API, Swift Proxy)
    -   storage management
    -   https://access.redhat.com/documentation/en-us/red_hat_openstack_platform/11/html/understanding_red_hat_openstack_platform_high_availability/pacemaker#pacemaker-virt

