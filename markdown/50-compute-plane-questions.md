<!-- .slide: data-state="section-break" id="compute-plane-question-section" data-menu-title="Compute plane questions" data-timing="5" -->
# Question time again


<!-- .slide: data-state="normal" id="compute-questions" data-menu-title="Question menu" data-timing="40" -->
## Questions relating to compute plane

*   How do we handle repeated failures of clones?
*   How do we handle failures on different networks?
*   How does the recovery service notify Pacemaker
    when recovery is complete?


<!-- .slide: data-state="normal" id="clone-failures" data-menu-title="Clone failures" data-timing="40" -->
## How to handle repeated failure of clones

e.g. `NovaCompute`

* `start-failure-is-fatal=false`
* `migration-threshold=3`

After third failure, Pacemaker gives up!

We need something more proactive.
