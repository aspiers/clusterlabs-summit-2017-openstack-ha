<!-- .slide: data-state="section-break" id="control-plane" data-timing="5" -->
# HA in OpenStack's control plane


<!-- .slide: data-state="normal" id="typical-control-plane" class="diagram-and-list" data-timing="40" -->
## Typical HA control plane

<div class="diagrams">
    <img class="services" data-src="images/services-cluster.svg"
         alt="HA services cluster" />
    <img class="db-mq" data-src="images/DB-MQ-cluster.svg"
         alt="database and message queue cluster" />
</div>

*   <!-- .element: class="fragment" -->
    1 or more clusters, possibly different sizes
*   <!-- .element: class="fragment" -->
    Stateless active-active API services
*   <!-- .element: class="fragment" -->
    Multi-master DB (Galera)
*   <!-- .element: class="fragment" -->
    RabbitMQ active-active
*   <!-- .element: class="fragment" -->
    [HAProxy](http://www.haproxy.org/) distributes service requests
*   <!-- .element: class="fragment" -->
    [Pacemaker](http://clusterlabs.org/) monitors and controls nodes and services

<div class="solved stamp fragment">
    <p class="solved">SOLVED</p>
    <p class="mostly fragment">(mostly)</p>
</div>

Note:

- [`neutron` HA is tricky](https://youtu.be/vBZgtHgSdOY), but out of the
scope of this talk.
- There's a general phobia of Pacemaker in many parts of the OpenStack community.


<!-- .slide: data-state="section-break" id="HAProxy-VIPs" data-timing="5" -->
# HAProxy and VIPs


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-1" data-menu-title="HAproxy" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 1" src="images/HPE_HA_Flow-1.png"/>
</a>

Note:
- keepalived often used instead of Pacemaker for VIPs


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-2" data-menu-title="HAproxy" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 3" src="images/HPE_HA_Flow-2.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-3" data-menu-title="HAproxy" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 4" src="images/HPE_HA_Flow-3.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-4" data-menu-title="HAproxy" data-timing="40" -->
<a href="https://docs.hpcloud.com/hos-5.x/helion/planning/high_availability.html#HP3.0HA__api_request">
    <img alt="HOS 5.0 API request message flow 5" src="images/HPE_HA_Flow-4.png"/>
</a>


<!-- .slide: data-state="blank-slide" class="full-screen" id="HOS-control-plane-5" data-menu-title="HAproxy" data-timing="40" -->
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


<!-- .slide: data-state="normal" id="RH-control-plane" data-timing="40" -->
## Red Hat OpenStack Platform control plane

*   


<!-- .slide: data-state="normal" id="control-plane-questions" data-timing="40" -->
## Remaining questions

