<!-- .slide: data-state="section-break" id="compute-plane-question-section" data-menu-title="Compute plane questions" data-timing="5" -->
# Compute plane questions


<!-- .slide: data-state="normal" id="compute-questions" data-menu-title="Question menu" data-timing="40" -->
## How to handle more flexible recovery of compute nodes

*   need to handle failures on different networks
*   need more options than just "restart"
*   options for Pacemaker
    *   fence
    *   notify cloud operator
    *   notify external monitoring system
        *   vitrage
        *   Nagios / Sensu?
    *   request external service to perform some recovery
        *   masakari
        *   mistral
        *   configurable policy for
            *   which VMs to recover
            *   where to recover to


<!-- .slide: data-state="normal" id="clone-failures" data-menu-title="Clone failures" data-timing="40" -->
## How to handle repeated failure of clones

* `start-failure-is-fatal=false` and `migration-threshold=3`
