# What happened??
The timings may be too tight with the original design, being forced to act in under 20ns. The fix is that instead of using each VGA pixel cycle to write data and preload, use four cycles to write and four to preload. This is good because it has MUCH longer to act, and I can therefore use a more covnentional 128k x 8 SRAM. It can easily act within 55ns, unlike the 12ns one.

