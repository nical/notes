
# Depth buffer

The traditional approach used by 3d graphics engines with forward rendering. It also works very well with 2D content. 

Split rendering into two phases:
- The opaque/order-independent phase, rendered with depth writes and depth test enabled. Within this pass items cab be drawn in any order but it is best for performance if items are drawn front-to-back.
- The apha/ordered phase during which items must be drawn back-to-front with depth test enabled but no depth write.

This aproach works well for content with a lot of opaque parts.  It also makes draw call [[batching]] trivial for items in the opaque phase.

This is what WebRender uses for rendering its layers (pictures).

# Rectangle splitting

Works when the workload contains only axis-aligned rectangles.

Used in WebRender's draw compositor: https://searchfox.org/mozilla-central/rev/251ada467616ddcbe96667c8a3a62a31b3421a21/gfx/wr/webrender/src/rectangle_occlusion.rs