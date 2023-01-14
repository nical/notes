
Paper by Diego Nehab: [[Nehab - Stroke to fill.pdf]] ([link](https://w3.impa.br/~diego/publications/Neh20.pdf)).

Exact stroke to fill is hard because of many ways the offset curves can cause self-overlaps. One way is to first offset the path with self-overlaps and remove the overlaps using a sweep-line algorithm that resolves the non-zero fill rule, or render the self-overlapping shape directly as a filled path with the non-zero fill rule.


## Skia's stroker

Source code: https://github.com/google/skia/blob/main/src/core/SkStroke.cpp
