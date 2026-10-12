# Methodology Notes: Aviation Spare Parts ROP Project

## Data
- Source: Synthetic Aviation Spare Parts dataset (Zenodo, record 20551017).
- Scope: one stocking location (MRO_BLR), all 60 parts, 72 monthly periods.
- The data is fully synthetic. Results demonstrate the method and say nothing about real spare-parts behaviour.

## Demand pattern classification
- Each part has ADI (average interval between demands) and CV² (squared coefficient of variation of non-zero demand sizes).
- Syntetos, Boylan & Croston (2005) thresholds: ADI 1.32 and CV² 0.49.
  - Smooth: low ADI, low CV²
  - Erratic: low ADI, high CV²
  - Intermittent: high ADI, low CV²
  - Lumpy: high ADI, high CV²
- Result: 37 Intermittent, 20 Smooth, 3 Lumpy, 0 Erratic.
- The dataset's own part_class column was ignored. Mover types were defined from the demand history.

## ABC and XYZ
- ABC is based on total demand quantity, because the dataset has no cost or price field. A value-based ABC is the usual standard and is preferable when cost is available.
- XYZ uses the coefficient of variation of demand. No part fell into Z, so the matrix shows only X and Y.

## Forecasting
- Smooth parts: 6-month moving average. Six months balances responsiveness against noise. It stands in for simple exponential smoothing and is not recursive.
- Intermittent parts: recursive Croston, built month by month with helper columns. It smooths demand size and interval separately, with alpha = 0.2.
- Lumpy (and Erratic) parts: Croston multiplied by (1 − α/2), the SBA correction, which removes Croston's upward bias.
- A static Croston (last size ÷ average interval) was built first and replaced by the recursive version, because the recursive one uses the whole history.

## Safety stock and ROP
- Safety stock = Z × √(LT × σd² + d² × σLT²), combining demand and lead-time variability.
- Z = NORM.S.INV(service level). The slider runs from 90% to 99%, default 95%.
- Lead time is converted from days to months (÷ 30) to match monthly demand.
- ROP = final forecast × lead time + safety stock.
- The same formula is applied to all parts, based on overall demand variability.

## Dashboard notes
- Reorder status compares on-hand stock with ROP. Current stock is a static snapshot, not a live feed.
- All 20 Smooth parts are below ROP and none show "OK". This follows from the simulated stock levels and is not a model error.
- Only 3 Lumpy parts exist, so lumpy results are based on a very small sample and shouldn't be over-read.
- The % of parts below ROP changes with the service-level slider.

## Limitations
- Synthetic data, and simulated inventory was not generated under any ROP policy.
- Quantity-based ABC, flat service level for every part, no Erratic or Z-class parts in the sample.
- SAP MM has no Croston or SBA, so in real use the annual external results would be entered as manual overrides (reorder point, safety stock).

## References
- Syntetos, A.A., Boylan, J.E. & Croston, J.D. (2005). On the categorization of demand patterns. Journal of the Operational Research Society, 56(5), 495–503.
