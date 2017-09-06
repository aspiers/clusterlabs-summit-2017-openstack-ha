<!-- .slide: data-state="section-break" id="general-question-section" data-menu-title="General questions" data-timing="5" -->
# General question time


<!-- .slide: data-state="normal" id="general-questions" data-menu-title="Question menu" data-timing="40" -->
## General Questions

*   <!-- .element: class="fragment" -->
    Can we make Pacemaker less scary?
*   <!-- .element: class="fragment" -->
    How can we get better testing?
    *   https://review.openstack.org/#/c/443504/


<!-- .slide: data-state="normal" id="debugging" data-menu-title="Debugging" data-timing="40" -->
## Debugging / trouble-shooting

*   <!-- .element: class="fragment" -->
    Logs are often incredibly hard to understand
*   <!-- .element: class="fragment" -->
    Less cryptic log messages would help a lot
*   <!-- .element: class="fragment" -->
    Currently all log messages are targeted at developers
    *  but those should be the `DEBUG`-level ones
*   <!-- .element: class="fragment" -->
    messages logged at higher levels than `DEBUG` should be targetted at users
    <br/> i.e. don't assume in-depth knowledge of Pacemaker internals!
    *   https://bugs.clusterlabs.org/show_bug.cgi?id=5294


<!-- .slide: data-state="normal" id="docs" data-menu-title="Documentation" data-timing="40" -->
## Better documentation needed

Already some great steps in the right direction, e.g.

*   http://crmsh.github.io/history-guide/
*   https://hackweek.suse.com/12/projects/852

but often documentation / man pages out of date or incomplete.

*   <!-- .element: class="fragment" -->
    challenge: find documentation which explains `crm_simulate -SL`
    *   https://bugs.clusterlabs.org/show_bug.cgi?id=5294
*   <!-- .element: class="fragment" -->
    a document explaining how the various timeout/migration parameters affect pengine is sorely needed
    *   something like http://ourobengr.com/ha/#worstcase but not just for STONITH
