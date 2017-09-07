<!-- .slide: data-state="normal" id="compute-failure" data-menu-title="Compute failure" data-timing="20" -->
## If only the control plane is HA&nbsp;…

<img class="arch" alt="control/compute architecture" data-src="images/architecture.svg" />
<img class="fragment bang" alt="compute node explosion!" data-src="images/explosion.svg" />

Note:
The control plane on the LHS is HA, but VMs live on the RHS,
so what happens if one of the compute nodes blows up?
