<!-- .slide: data-state="section-break" id="compute-plane" data-menu-title="Compute plane HA" data-timing="10" -->
# HA in OpenStack's compute plane


<!-- .slide: data-state="blank-slide" class="full-screen" id="user-story" data-menu-title="User story" data-timing="40" -->
<a href="http://specs.openstack.org/openstack/openstack-user-stories/user-stories/proposed/ha_vm.html">
    <img alt="User story" data-src="images/user-story.png"/>
</a>


<!-- .slide: data-state="normal" id="pacemaker-remote" data-menu-title="pacemaker_remote" class="architecture" data-timing="40" -->
## `pacemaker_remote` to the rescue!

<div class="architecture">
    <img alt="Architecture with pacemaker_remote"
         class="architecture"
         data-src="images/standard-architecture.svg" />

    <img alt="Architecture with pacemaker_remote arrows"
         class="architecture fragment"
         data-src="images/standard-architecture-remote-arrows.svg" />
</div>

Note:
Scalability issue solved by `pacemaker_remote`

*   New(-ish) Pacemaker feature
*   Allows core cluster nodes to control "remote"
    nodes via a `pacemaker_remote` proxy service (daemon)
*   Can scale to very large numbers


<!-- .slide: data-state="normal" id="failure-modes" class="architecture" data-menu-title="Failure modes" data-timing="80" -->
## Handling different failure modes

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
    <span class="fragment" data-fragment-index="2">
        <img class="fragment fade-out fence"
             data-fragment-index="3"
             alt="fencing dead compute node"
             data-src="images/cross.svg" />
        <img class="fragment fade-out migration"
             data-fragment-index="3"
             alt="resurrecting dead VMs elsewhere"
             data-src="images/migration-arrow.svg" />
    </span>
    <span class="fragment" data-fragment-index="3">
        <img class="fragment fade-out kernel bang"
             data-fragment-index="4"
             alt="kernel / OS crash or hang"
             data-src="images/explosion.svg" />
    </span>
    <span class="fragment" data-fragment-index="4">
        <img class="fragment fade-out fence"
             data-fragment-index="5"
             alt="fencing dead compute node"
             data-src="images/cross.svg" />
        <img class="fragment fade-out migration"
             data-fragment-index="5"
             alt="resurrecting dead VMs elsewhere"
             data-src="images/migration-arrow.svg" />
    </span>
    <span class="fragment" data-fragment-index="5">
        <img class="fragment fade-out libvirt bang"
             data-fragment-index="6"
             alt="libvirt crash or hang"
             data-src="images/explosion.svg" />
    </span>
    <span class="fragment" data-fragment-index="6">
        <img class="fragment fade-out nova-compute bang"
             data-fragment-index="7"
             alt="nova-compute crash or hang"
             data-src="images/explosion.svg" />
    </span>
    <span class="fragment" data-fragment-index="7">
        <img class="fragment fade-out nova-api bang"
             data-fragment-index="8"
             alt="nova-api crash or hang"
             data-src="images/explosion.svg" />
    </span>
    <span class="fragment" data-fragment-index="8">
        <img class="fragment fade-out recovery bang"
             data-fragment-index="9"
             alt="recovery controller crash or hang"
             data-src="images/explosion.svg" />
    </span>
    <span class="fragment" data-fragment-index="9">
        <img class="fragment fade-out VM bang"
             data-fragment-index="10"
             alt="VM crash or hang"
             data-src="images/explosion.svg" />
    </span>
        <img class="fragment workload bang"
             data-fragment-index="10"
             alt="workload crash or hang"
             data-src="images/explosion.svg" />
</div>

Note:

*   If we have a compute node failure, after fencing the node,
    we need to resurrect the VMs in a way which OpenStack is aware of.
*   Luckily `nova` provides an API for doing this, which is called
    `nova evacuate`.  So we just call that API and `nova` takes care
    of the rest.
*   Without shared storage, simply rebuilds from scratch
*   Needs to handle failure or (temporary) freeze of:
    *   Hardware (including various NICs)
    *   Kernel
    *   Hypervisor services (e.g. `libvirt`)
    *   OpenStack control plane services
        *   including resurrection workflow
    *   VM
    *   Workload inside VM (ideally)
*   We assume that Pacemaker is reliable, otherwise we're sunk!


<!-- .slide: data-state="normal" id="ocf-architecture" data-menu-title="OCF RAs" class="architecture" data-timing="150" -->
## `NovaCompute` / `NovaEvacuate` OCF agents

<div class="architecture">
    <img alt="OCF RA architecture"
         class="OCF-RA architecture"
         data-src="images/OCF-RA-architecture.svg" />
</div>

Note:
*   Custom OCF Resource Agents (RAs)
    *   Pacemaker plugins to manage resources
*   Custom fencing agent (`fence_compute`) flags host for recovery
*   `NovaEvacuate` RA polls for flags, and initiates recovery
    *   Will keep retrying if recovery not possible
*   `NovaCompute` RA starts / stops `nova-compute`
    *   Start waits for recovery to complete


<!-- .slide: data-state="normal" id="ocf-pros-cons" data-menu-title="OCF RA pros and cons" data-timing="40" -->
## `NovaCompute` / `NovaEvacuate` OCF agents

### Pros

*   Has been production-ready for a "long" time
*   Commercial support available
*   OCF RAs [upstream in `openstack-resource-agents` repo](https://github.com/openstack/openstack-resource-agents/tree/master/ocf) (sort of)

### Cons

*   Known limitations (not bugs):
    *   Only handles failure of compute node, not of VMs, or `nova-compute`
    *   Some corner cases still problematic, e.g. if `nova` fails during recovery


<!-- .slide: data-state="section-break" id="other-design-goals" data-timing="10" -->
# What else is missing?


<!-- .slide: data-state="normal" id="operability" data-menu-title="Operability" data-timing="40" -->
## Design goal: Operability

<figure>
    <img alt="API" data-src="images/API.jpg" style="width: 100%" />
</figure>

Note:

Cloud operators need an API to access details of ongoing and
historical alerts and corresponding actions.

This could be incorporated into Horizon.


<!-- .slide: data-state="normal" id="configurability" data-menu-title="Configurability" data-timing="40" -->
## Design goal: Configurability

Different cloud operators will want to support different SLAs
with different workflows:

*   <!-- .element: class="fragment" -->
    Choose which VMs to auto-resurrect
*   <!-- .element: class="fragment" -->
    Reserve some hypervisor hosts for recovery
*   <!-- .element: class="fragment" -->
    Retry thresholds on recovery of processes and VM instances
*   <!-- .element: class="fragment" -->
    Configurable workflows
    *   <!-- .element: class="fragment" -->
        notify cloud operator
    *   <!-- .element: class="fragment" -->
        notify external monitoring system
        ([Vitrage](https://docs.openstack.org/vitrage/latest/),
        Nagios, Sensu,&nbsp;…)
    *   <!-- .element: class="fragment" -->
        fence
    *   <!-- .element: class="fragment" -->
        request external service to perform some recovery
        (resurrect VMs)

Note: There is no one-size-fits-all solution to compute HA.


<!-- .slide: data-state="normal" id="context-aware" data-menu-title="Context-aware recovery" data-timing="120" -->
## Design goal: Intelligent, Context-aware Recovery

<br />

In some failure scenarios, VMs are still perfectly healthy
but unmanageable.

Should they be automatically killed?  Depends on
the workload.
<!-- .element: class="fragment" -->

<br />

<table class="waffle fragment" cellspacing="0" cellpadding="0">
  <thead>
    <tr>
      <th colspan="4" class="failure" />
      <th colspan="2"> Corrective action for</th>
    </tr>
    <tr>
      <th>Admin network</th>
      <th>Neutron network</th>
      <th>Storage network</th>
      <th>`nova-compute` / `libvirtd`</th>
      <th class="cattle action">cattle</th>
      <th class="pet action">pets</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td class="no">Down</td>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="cattle action" rowspan="4">Fence &rarr; resurrect</td>
      <td class="pet action">Notify operator</td>
    </tr>
    <tr>
      <td class="yes">Up</td>
      <td class="no">Down</td>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="pet action">Migrate</td>
    </tr>
    <tr>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="no">Down</td>
      <td class="yes">Up</td>
      <td class="pet action">Fence, resurrect</td>
    </tr>
    <tr>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="yes">Up</td>
      <td class="no">Down</td>
      <td class="pet action">Restart &rarr; notify operator</td>
    </tr>
  </tbody>
</table>


<!-- .slide: data-state="normal" id="context-aware-2" data-menu-title="   (continued)" data-timing="120" -->
## Design goal: Intelligent, Context-aware Recovery (2)

*   <!-- .element: class="fragment" -->
    Fault deduplication (via [Vitrage](https://docs.openstack.org/vitrage/latest/))
*   <!-- .element: class="fragment" -->
    Set host to maintenance mode until recovery is complete
*   <!-- .element: class="fragment" -->
    Respect ephemeral storage boundaries where applicable


<!-- .slide: data-state="normal" id="performance" data-menu-title="Performance" data-timing="70" -->
## Design goal: Performance

*   <!-- .element: class="fragment" -->
    Quick response to failures
    *   <!-- .element: class="fragment" -->
        Push not pull — don't poll
    *   <!-- .element: class="fragment" -->
        Notifications on message bus
*   <!-- .element: class="fragment" -->
    Fault prioritization / pre-emption
    *   <!-- .element: class="fragment" -->
        e.g. host faults pre-empt instance faults
    *   <!-- .element: class="fragment" -->
        Pre-emption is visible to operators


<!-- .slide: data-state="normal" id="upgradability" data-timing="50" -->
# Upgradability

<figure>
    <img alt="Upgrade failed dialog box"
         data-src="images/upgrade-are-failed-2.gif"
         style="width: 90%" />
</figure>

Note: We need easy migration from existing compute HA deployments, so
don't make life hard for (existing customers of) SUSE, NTT, Red Hat,
or anyone else using upstream solution


<!-- .slide: data-state="section-break" id="masakari" data-menu-title="Masakari" data-timing="10" -->
# Masakari to the rescue!


<!-- .slide: data-state="normal" id="masakari-architecture" class="architecture" data-timing="320" -->
## Masakari architecture

<div class="architecture">
    <img alt="Standard architecture with pacemaker_remote"
         class="architecture fragment fade-out" data-fragment-index="1"
         data-src="images/standard-architecture.svg" />

    <span class="fragment" data-fragment-index="1">
        <img alt="masakari architecture"
             class="masakari architecture fragment fade-out" data-fragment-index="2"
             data-src="images/masakari-architecture2.svg" />
    </span>

    <span class="fragment" data-fragment-index="2">
        <img alt="masakari process failure"
             class="masakari architecture fragment fade-out" data-fragment-index="3"
             data-src="images/masakari-processdown.svg" />
    </span>

    <span class="fragment" data-fragment-index="3">
        <img alt="masakari vm failure"
             class="masakari architecture fragment fade-out" data-fragment-index="4"
             data-src="images/masakari-vmdown.svg" />
    </span>

    <span class="fragment" data-fragment-index="4">
        <img alt="masakari host failure"
             class="masakari architecture"
             data-src="images/masakari-hostdown.svg" />
    </span>
</div>

Note:

*   Similar architectural concept, different code
    *   Recovery handled by separate controller service
    *   Persists state to database
*   Monitors for [3 types of failure](https://github.com/ntt-sic/masakari/blob/master/docs/evacuation_patterns.md):
    *   compute node down
    *   `nova-compute` service down
    *   VM down (detected via `libvirt`)


<!-- .slide: data-state="normal" id="existing-architecture" data-menu-title="OCF architecture" data-timing="70" -->

<div class="new-architecture">
    <img alt="Standard architecture with pacemaker_remote"
         class="architecture"
         data-src="images/OCF-architecture.svg" />
</div>


<!-- .slide: data-state="normal" id="unified-architecture" data-menu-title="Driver-based alerting RA" data-timing="140" -->

<div class="new-architecture">
    <img alt="Driver-based nova-host-alerter RA"
         class="architecture"
         data-src="images/unified-architecture.svg" />
</div>

Note:

- Send host failure notifications from multi-evacuate RA to masakari via python-masakari
    - Eliminates need for masakari-hostmonitor
    - No risk of notification getting lost by Pacemaker (stays in attrd until dispatched)
    - No risk of notification getting lost by Masakari (persistent in DB until handled)
- Replace masakari-processmonitor with standard Pacemaker process monitoring


<!-- .slide: data-state="normal" id="unified-architecture-mistral" data-menu-title="Mistral architecture" data-timing="30" -->

<div class="new-architecture">
    <img alt="Original mistral architecture"
         class="architecture"
         data-src="images/unified-architecture+mistral.svg" />
</div>


<!-- .slide: data-state="normal" id="grand-unified-architecture" data-menu-title="Grand unified architecture" data-timing="70" -->

<div class="new-architecture">
    <img alt="Grand unified architecture"
         class="architecture"
         data-src="images/grand-unified-architecture.svg" />
</div>


<!-- .slide: data-state="normal" id="modular" data-timing="60" -->
# Modular architecture

*   <!-- .element: class="fragment" -->
    Key areas identified in previous Design Summits:
    *   <!-- .element: class="fragment" -->
        Host monitoring / recovery
    *   <!-- .element: class="fragment" -->
        VM monitoring / recovery
    *   <!-- .element: class="fragment" -->
        Process monitoring / recovery
*   <!-- .element: class="fragment" -->
    Agreed to split into independent components
*   <!-- .element: class="fragment" -->
    https://etherpad.openstack.org/p/newton-instance-ha


<!-- .slide: data-state="normal" id="specs" data-timing="50" -->
# Specs

[`openstack-resource-agents-specs` repository](https://github.com/openstack/openstack-resource-agents-specs/tree/master/specs/newton/approved)

*   <!-- .element: class="fragment" -->
    Host monitoring
*   <!-- .element: class="fragment" -->
    Host recovery
*   <!-- .element: class="fragment" -->
    VM monitoring
*   <!-- .element: class="fragment" -->
    VM recovery
*   <!-- .element: class="fragment" -->
    `libvirtd` OCF RA
    *   to take action if migration-threshold reached
*   <!-- .element: class="fragment" -->
    `NovaCompute` OCF RA
    *   ditto
