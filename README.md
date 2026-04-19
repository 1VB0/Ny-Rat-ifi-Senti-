// ─────────────────────────────────────────────────────────────────────────────
// NY retest - Rat-ifi Zone Senti  ·  v6  ·  by 1vb0
// + 5m Range Mean Line + 5m Median Line + Mean Rejection Signal
// ─────────────────────────────────────────────────────────────────────────────
//@version=6
strategy("NY retest - Rat-ifi Zone Senti by 1vb0",
     overlay         = true,
     max_lines_count  = 500,
     max_labels_count = 200,
     max_boxes_count  = 200)

// ─────────────────────────────────────────────────────────────────────────────
// INPUTS
// ─────────────────────────────────────────────────────────────────────────────
rsi_length   = input.int   (14,   "RSI Length",                  minval = 2)
rsi_src      = input.source(close,"RSI Source")
div_lookback = input.int   (5,    "Divergence Lookback (bars)",  minval = 2, maxval = 20)
require_div  = input.bool  (true, "Require RSI Divergence",
     tooltip = "When ON the retest-breakout signal also requires a recent RSI divergence in the same direction. When OFF only the price-action retest sequence is needed.")
div_memory   = input.int   (20,   "Divergence Memory (bars)",    minval = 1, maxval = 50)
retest_tol   = input.float (1.0,  "Retest Tolerance (ticks)",    minval = 0., maxval = 10., step = 0.5)
show_labels  = input.bool  (true, "Show Long/Short Labels")
show_ranges  = input.bool  (true, "Show Range Lines")
show_state_dots = input.bool(false, "Show State Indicator Dots",
     tooltip = "Debug only — shows state ①/② dots on every bar. Keep OFF for clean chart.")

// ── 5m Mean ──────────────────────────────────────────────────────────────────
grp_mean = "── 5m Range Mean & Median ──────────────────"
show_mean       = input.bool  (true,  "Show 5m Mean Line  (midpoint H+L/2)", group = grp_mean)
col_mean        = input.color (color.new(#FFD700, 0), "Mean Line Colour",    group = grp_mean,
     tooltip = "Gold — sits exactly halfway between hi_5m and lo_5m.")
show_median     = input.bool  (true,  "Show 5m Median Line (true middle price)", group = grp_mean)
col_median      = input.color (color.new(#FF69B4, 0), "Median Line Colour",  group = grp_mean,
     tooltip = "Hot-pink. The median is the centre value when every hlc3 from 9:30 to 9:35 is sorted — differs from the mean when price spikes skew the range.")
show_mean_rej   = input.bool  (true,  "Show Mean Rejection Signal",          group = grp_mean)
mean_rej_tol    = input.float (2.0,   "Rejection Tolerance (ticks)",         group = grp_mean,
     minval = 0.5, maxval = 20., step = 0.5,
     tooltip = "How many ticks above/below the mean counts as 'touching' it for a rejection. Widen for volatile instruments like crude oil.")
mean_rej_bars   = input.int   (3,     "Rejection Confirm Bars",              group = grp_mean,
     minval = 1, maxval = 10,
     tooltip = "How many consecutive closes in the OPPOSITE direction after touching the mean are needed before the rejection label fires. 1 = next bar close, 3 = default.")

// ── SL / Entry ────────────────────────────────────────────────────────────────
grp_sl = "── SL / Entry Display ──────────────────────"
show_sl_line  = input.bool (true,  "Show SL & Entry Lines (extends right)", group = grp_sl)
show_sl_label = input.bool (true,  "Show SL & Entry Price Axis Tags",       group = grp_sl)
sl_col        = input.color(color.new(color.red,  0), "Stop Loss Colour",   group = grp_sl)
en_col        = input.color(color.new(color.lime, 0), "Entry Colour",       group = grp_sl)

// ── 5m Zone Box ───────────────────────────────────────────────────────────────
grp_zone     = "═══ 5m Zone Highlight ═══"
show_5m_zone = input.bool (true,  "Show 5m Zone Highlight",             group = grp_zone)
col_5m_bull  = input.color(color.new(color.green, 75), "Bull (above hi_5m)", group = grp_zone)
col_5m_bear  = input.color(color.new(color.red,   75), "Bear (below lo_5m)", group = grp_zone)
col_5m_neut  = input.color(color.new(color.gray,  88), "Neutral (inside)",   group = grp_zone)

// ─────────────────────────────────────────────────────────────────────────────
// TIME HELPERS
// ─────────────────────────────────────────────────────────────────────────────
f_snapshot(string sess) =>
    in_now  = not na(time("1", sess))
    in_prev = not na(time("1", sess)[1])
    in_now and not in_prev

snap_930  = f_snapshot("0930-0931:23456")
snap_935  = f_snapshot("0935-0936:23456")
snap_940  = f_snapshot("0940-0941:23456")
snap_945  = f_snapshot("0945-0946:23456")
snap_1000 = f_snapshot("1000-1001:23456")
snap_1030 = f_snapshot("1030-1031:23456")
snap_1330 = f_snapshot("1330-1331:23456")
snap_1800 = f_snapshot("1800-1801:23456")

in_ny = not na(time("1", "0930-2000:23456"))

// ─────────────────────────────────────────────────────────────────────────────
// ROLLING HIGH / LOW
// ─────────────────────────────────────────────────────────────────────────────
var float roll_hi = na
var float roll_lo = na

if snap_930
    roll_hi := high
    roll_lo := low
else if in_ny
    roll_hi := math.max(roll_hi, high)
    roll_lo := math.min(roll_lo, low)

var float hi_5m  = na
var float lo_5m  = na
var float hi_10m = na
var float lo_10m = na
var float hi_15m = na
var float lo_15m = na
var float hi_30m = na
var float lo_30m = na
var float hi_60m = na
var float lo_60m = na
var float hi_4h  = na
var float lo_4h  = na
var float hi_8h  = na
var float lo_8h  = na

if snap_935
    hi_5m  := roll_hi
    lo_5m  := roll_lo
if snap_940
    hi_10m := roll_hi
    lo_10m := roll_lo
if snap_945
    hi_15m := roll_hi
    lo_15m := roll_lo
if snap_1000
    hi_30m := roll_hi
    lo_30m := roll_lo
if snap_1030
    hi_60m := roll_hi
    lo_60m := roll_lo
if snap_1330
    hi_4h  := roll_hi
    lo_4h  := roll_lo
if snap_1800
    hi_8h  := roll_hi
    lo_8h  := roll_lo

// ─────────────────────────────────────────────────────────────────────────────
// 5m MEAN
//  (hi_5m + lo_5m) / 2  —  the simple midpoint of the locked range boundaries.
//  Locked at snap_935. Resets at snap_930 each day.
// ─────────────────────────────────────────────────────────────────────────────
var float mean_5m = na

if snap_935 and not na(hi_5m) and not na(lo_5m)
    mean_5m := (hi_5m + lo_5m) / 2.0

if snap_930
    mean_5m := na

// ─────────────────────────────────────────────────────────────────────────────
// 5m MEDIAN
//  The true statistical median of every bar's hlc3 from 9:30 up to (not
//  including) 9:35.  Collected bar-by-bar, sorted at lock, middle value taken.
//
//  WHY IT DIFFERS FROM THE MEAN:
//    If one wick spikes far above/below, the mean shifts toward that spike
//    but the median stays at the true centre of the distribution.
//    The median is therefore a more robust "fair value" reference for the range.
//
//  IMPLEMENTATION:
//    • median_buf   — collects hlc3 values from every bar inside the window
//    • At snap_935  — array is sorted; middle index is picked
//    • At snap_930  — array is cleared for the next day
// ─────────────────────────────────────────────────────────────────────────────
var float          median_5m  = na
var array<float>   median_buf = array.new_float()

// Collect every hlc3 during the 9:30-9:35 window (before the lock fires)
if in_ny and not na(time("1", "0930-0935:23456"))
    array.push(median_buf, hlc3)

// Lock: sort and pick the centre value
if snap_935
    if array.size(median_buf) > 0
        array.sort(median_buf, order.ascending)
        int sz  = array.size(median_buf)
        int mid = int(sz / 2)
        // For even-length arrays take the average of the two centre values
        if sz % 2 == 0 and sz >= 2
            median_5m := (array.get(median_buf, mid - 1) + array.get(median_buf, mid)) / 2.0
        else
            median_5m := array.get(median_buf, mid)

// Reset on new day
if snap_930
    median_5m := na
    array.clear(median_buf)

// ─────────────────────────────────────────────────────────────────────────────
// COLORS
// ─────────────────────────────────────────────────────────────────────────────
c_5m  = color.new(color.red,    0)
c_10m = color.new(color.orange, 0)
c_15m = color.new(color.yellow, 0)
c_30m = color.new(color.green,  0)
c_60m = color.new(color.blue,   0)
c_4h  = color.new(#ca97ee,      0)
c_8h  = color.new(#8B00FF,      0)

// ─────────────────────────────────────────────────────────────────────────────
// PLOT RANGE LINES
// ─────────────────────────────────────────────────────────────────────────────
plot(show_ranges ? hi_5m  : na, "5m High",  color = c_5m,  linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? lo_5m  : na, "5m Low",   color = c_5m,  linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? hi_10m : na, "10m High", color = c_10m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? lo_10m : na, "10m Low",  color = c_10m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? hi_15m : na, "15m High", color = c_15m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? lo_15m : na, "15m Low",  color = c_15m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? hi_30m : na, "30m High", color = c_30m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? lo_30m : na, "30m Low",  color = c_30m, linewidth = 1, style = plot.style_stepline)
plot(show_ranges ? hi_60m : na, "60m High", color = c_60m, linewidth = 2, style = plot.style_stepline)
plot(show_ranges ? lo_60m : na, "60m Low",  color = c_60m, linewidth = 2, style = plot.style_stepline)
plot(show_ranges ? hi_4h  : na, "4H High",  color = c_4h,  linewidth = 2, style = plot.style_stepline)
plot(show_ranges ? lo_4h  : na, "4H Low",   color = c_4h,  linewidth = 2, style = plot.style_stepline)
plot(show_ranges ? hi_8h  : na, "8H High",  color = c_8h,  linewidth = 2, style = plot.style_stepline)
plot(show_ranges ? lo_8h  : na, "8H Low",   color = c_8h,  linewidth = 2, style = plot.style_stepline)

// ── 5m Mean line ─────────────────────────────────────────────────────────────
//  Dashed gold line exactly between hi_5m and lo_5m.
//  Thinner than the H/L lines so it reads as a midpoint reference, not a level.
plot(show_mean and not na(mean_5m) ? mean_5m : na,
     "5m Mean",
     color     = col_mean,
     linewidth = 1,
     style     = plot.style_stepline)

// ── 5m Median line ────────────────────────────────────────────────────────────
//  Dotted hot-pink line — the true statistical centre of all 9:30-9:35 prints.
plot(show_median and not na(median_5m) ? median_5m : na,
     "5m Median",
     color     = col_median,
     linewidth = 1,
     style     = plot.style_stepline)

bgcolor(snap_930 ? color.new(color.white, 80) : na, title = "NY Open Flash")

// ─────────────────────────────────────────────────────────────────────────────
// RANGE LABELS
// ─────────────────────────────────────────────────────────────────────────────
if show_ranges
    if snap_935
        label.new(x = time, y = hi_5m,  text = "5m H",  xloc = xloc.bar_time, style = label.style_label_left, color = c_5m,    textcolor = color.white, size = size.tiny)
        label.new(x = time, y = lo_5m,  text = "5m L",  xloc = xloc.bar_time, style = label.style_label_left, color = c_5m,    textcolor = color.white, size = size.tiny)
        if show_mean and not na(mean_5m)
            label.new(x = time, y = mean_5m, text = "5m M", xloc = xloc.bar_time, style = label.style_label_left, color = col_mean, textcolor = color.black, size = size.tiny)
        if show_median and not na(median_5m)
            label.new(x = time, y = median_5m, text = "5m Mdn", xloc = xloc.bar_time, style = label.style_label_left, color = col_median, textcolor = color.black, size = size.tiny)
    if snap_940
        label.new(x = time, y = hi_10m, text = "10m H", xloc = xloc.bar_time, style = label.style_label_left, color = c_10m, textcolor = color.black, size = size.tiny)
        label.new(x = time, y = lo_10m, text = "10m L", xloc = xloc.bar_time, style = label.style_label_left, color = c_10m, textcolor = color.black, size = size.tiny)
    if snap_945
        label.new(x = time, y = hi_15m, text = "15m H", xloc = xloc.bar_time, style = label.style_label_left, color = c_15m, textcolor = color.black, size = size.tiny)
        label.new(x = time, y = lo_15m, text = "15m L", xloc = xloc.bar_time, style = label.style_label_left, color = c_15m, textcolor = color.black, size = size.tiny)
    if snap_1000
        label.new(x = time, y = hi_30m, text = "30m H", xloc = xloc.bar_time, style = label.style_label_left, color = c_30m, textcolor = color.black, size = size.tiny)
        label.new(x = time, y = lo_30m, text = "30m L", xloc = xloc.bar_time, style = label.style_label_left, color = c_30m, textcolor = color.black, size = size.tiny)
    if snap_1030
        label.new(x = time, y = hi_60m, text = "60m H", xloc = xloc.bar_time, style = label.style_label_left, color = c_60m, textcolor = color.white, size = size.tiny)
        label.new(x = time, y = lo_60m, text = "60m L", xloc = xloc.bar_time, style = label.style_label_left, color = c_60m, textcolor = color.white, size = size.tiny)
    if snap_1330
        label.new(x = time, y = hi_4h,  text = "4H H",  xloc = xloc.bar_time, style = label.style_label_left, color = c_4h,  textcolor = color.white, size = size.tiny)
        label.new(x = time, y = lo_4h,  text = "4H L",  xloc = xloc.bar_time, style = label.style_label_left, color = c_4h,  textcolor = color.white, size = size.tiny)
    if snap_1800
        label.new(x = time, y = hi_8h,  text = "8H H",  xloc = xloc.bar_time, style = label.style_label_left, color = c_8h,  textcolor = color.white, size = size.tiny)
        label.new(x = time, y = lo_8h,  text = "8H L",  xloc = xloc.bar_time, style = label.style_label_left, color = c_8h,  textcolor = color.white, size = size.tiny)

// ─────────────────────────────────────────────────────────────────────────────
// 5M ZONE BOX
// ─────────────────────────────────────────────────────────────────────────────
var box zone_box_5m = na

if snap_930
    if not na(zone_box_5m)
        box.delete(zone_box_5m)
        zone_box_5m := na

if snap_935 and show_5m_zone and not na(hi_5m) and not na(lo_5m)
    zone_box_5m := box.new(
         left         = bar_index,
         top          = hi_5m,
         right        = bar_index + 1,
         bottom       = lo_5m,
         border_color = color.new(color.gray, 80),
         bgcolor      = col_5m_neut,
         border_width = 1,
         xloc         = xloc.bar_index)

if show_5m_zone and not na(zone_box_5m) and in_ny and not na(hi_5m) and not na(lo_5m)
    box.set_right(zone_box_5m, bar_index + 1)
    if close > hi_5m
        box.set_bgcolor(zone_box_5m, col_5m_bull)
    else if close < lo_5m
        box.set_bgcolor(zone_box_5m, col_5m_bear)
    else
        box.set_bgcolor(zone_box_5m, col_5m_neut)

// ─────────────────────────────────────────────────────────────────────────────
// RSI DIVERGENCE ENGINE
// ─────────────────────────────────────────────────────────────────────────────
rsi_val = ta.rsi(rsi_src, rsi_length)

pivot_lo_price = ta.pivotlow (low,     div_lookback, div_lookback)
pivot_hi_price = ta.pivothigh(high,    div_lookback, div_lookback)
pivot_lo_rsi   = ta.pivotlow (rsi_val, div_lookback, div_lookback)
pivot_hi_rsi   = ta.pivothigh(rsi_val, div_lookback, div_lookback)

var float prev_lo_price = na
var float prev_lo_rsi   = na
var float last_lo_price = na
var float last_lo_rsi   = na
var float prev_hi_price = na
var float prev_hi_rsi   = na
var float last_hi_price = na
var float last_hi_rsi   = na

if not na(pivot_lo_price)
    prev_lo_price := last_lo_price
    prev_lo_rsi   := last_lo_rsi
    last_lo_price := pivot_lo_price
    last_lo_rsi   := pivot_lo_rsi

if not na(pivot_hi_price)
    prev_hi_price := last_hi_price
    prev_hi_rsi   := last_hi_rsi
    last_hi_price := pivot_hi_price
    last_hi_rsi   := pivot_hi_rsi

bull_div_now = not na(prev_lo_price) and not na(last_lo_price) and
               last_lo_price < prev_lo_price and last_lo_rsi > prev_lo_rsi

bear_div_now = not na(prev_hi_price) and not na(last_hi_price) and
               last_hi_price > prev_hi_price and last_hi_rsi < prev_hi_rsi

var int bull_div_age = 999
var int bear_div_age = 999

if bull_div_now
    bull_div_age := 0
else
    bull_div_age += 1

if bear_div_now
    bear_div_age := 0
else
    bear_div_age += 1

recent_bull_div = bull_div_age <= div_memory
recent_bear_div = bear_div_age <= div_memory

// ─────────────────────────────────────────────────────────────────────────────
// RETEST-BREAKOUT STATE MACHINE
// ─────────────────────────────────────────────────────────────────────────────
var int bull_state = 0
var int bear_state = 0

if snap_930
    bull_state := 0
    bear_state := 0

tol_v = retest_tol * syminfo.mintick

retest_long_raw  = bull_state == 2 and not na(hi_5m) and close > hi_5m and in_ny
retest_short_raw = bear_state == 2 and not na(lo_5m) and close < lo_5m and in_ny

long_signal  = retest_long_raw  and (not require_div or recent_bull_div)
short_signal = retest_short_raw and (not require_div or recent_bear_div)

if not na(hi_5m) and not na(lo_5m) and in_ny
    if bull_state == 0
        if close > hi_5m
            bull_state := 1
    else if bull_state == 1
        if close < lo_5m
            bull_state := 0
        else if low <= hi_5m + tol_v
            bull_state := 2
    else if bull_state == 2
        if close > hi_5m
            bull_state := 1
        else if close < lo_5m
            bull_state := 0

    if bear_state == 0
        if close < lo_5m
            bear_state := 1
    else if bear_state == 1
        if close > hi_5m
            bear_state := 0
        else if high >= lo_5m - tol_v
            bear_state := 2
    else if bear_state == 2
        if close < lo_5m
            bear_state := 1
        else if close > hi_5m
            bear_state := 0

// ─────────────────────────────────────────────────────────────────────────────
// 5m MEAN REJECTION ENGINE
// ─────────────────────────────────────────────────────────────────────────────
//
//  A rejection fires when ALL of the following are true on bar N:
//
//  BULLISH REJECTION FROM MEAN  (price drops to mean then bounces UP)
//    ① Price is INSIDE the 5m range  (between lo_5m and hi_5m)
//    ② At least one of the last N bars (where N = mean_rej_bars) had its
//       LOW touch or pierce the mean  (within mean_rej_tol ticks)
//    ③ The current close is ABOVE the mean  (confirming the bounce)
//    ④ The current close > open  (green bar, confirming upward push)
//
//  BEARISH REJECTION FROM MEAN  (price rises to mean then gets rejected DOWN)
//    ① Price is INSIDE the 5m range
//    ② At least one of the last N bars had its HIGH touch or pierce the mean
//    ③ The current close is BELOW the mean
//    ④ The current close < open  (red bar, confirming downward push)
//
//  Cooldown: once a mean rejection fires, a 5-bar cooldown prevents
//  back-to-back labels at the same level.
//
//  Why this approach:
//    Using a "touch window" (checking back mean_rej_bars bars) rather than
//    requiring the touch and confirmation on the SAME bar prevents the signal
//    from firing prematurely on the touch bar itself.  The confirmation bar
//    (current close direction) is what fires the label.
// ─────────────────────────────────────────────────────────────────────────────

mean_tol_v = mean_rej_tol * syminfo.mintick

// Is price inside the 5m range right now?
inside_range = not na(mean_5m) and not na(hi_5m) and not na(lo_5m) and
               close <= hi_5m and close >= lo_5m and in_ny

// Did any of the last mean_rej_bars bars touch the mean from below (low touched mean)?
mean_touched_low  = false
mean_touched_high = false

if not na(mean_5m)
    for i = 1 to mean_rej_bars
        if low[i]  <= mean_5m + mean_tol_v and low[i]  >= mean_5m - mean_tol_v
            mean_touched_low  := true
        if high[i] >= mean_5m - mean_tol_v and high[i] <= mean_5m + mean_tol_v
            mean_touched_high := true

// Bullish rejection: wick touched mean from above → price now closes above mean
mean_bull_rej_raw = inside_range and mean_touched_low  and close > mean_5m and close > open

// Bearish rejection: wick touched mean from below → price now closes below mean
mean_bear_rej_raw = inside_range and mean_touched_high and close < mean_5m and close < open

// Cooldown counter — prevents signal spam at the same level
var int mean_rej_cooldown = 0

if mean_rej_cooldown > 0
    mean_rej_cooldown -= 1

mean_bull_rej = show_mean_rej and mean_bull_rej_raw and mean_rej_cooldown == 0
mean_bear_rej = show_mean_rej and mean_bear_rej_raw and mean_rej_cooldown == 0

if mean_bull_rej or mean_bear_rej
    mean_rej_cooldown := 5

// ─────────────────────────────────────────────────────────────────────────────
// SIGNAL SHAPES  —  main breakout
// ─────────────────────────────────────────────────────────────────────────────
plotshape(show_labels and long_signal,
     title     = "LONG",
     style     = shape.labelup,
     location  = location.belowbar,
     color     = color.new(color.lime, 5),
     textcolor = color.black,
     text      = "LONG",
     size      = size.normal)

plotshape(show_labels and short_signal,
     title     = "SHORT",
     style     = shape.labeldown,
     location  = location.abovebar,
     color     = color.new(color.red, 5),
     textcolor = color.white,
     text      = "SHORT",
     size      = size.normal)

// ─────────────────────────────────────────────────────────────────────────────
// MEAN REJECTION LABELS
//  Smaller than the main LONG/SHORT labels so they read as a secondary signal.
//  Gold border to match the mean line colour.
//  Text: "M↑" for bullish rejection off mean, "M↓" for bearish.
// ─────────────────────────────────────────────────────────────────────────────
plotshape(mean_bull_rej,
     title     = "Mean Bounce ▲",
     style     = shape.labelup,
     location  = location.belowbar,
     color     = color.new(col_mean, 10),
     textcolor = color.black,
     text      = "M↑",
     size      = size.small)

plotshape(mean_bear_rej,
     title     = "Mean Rejection ▼",
     style     = shape.labeldown,
     location  = location.abovebar,
     color     = color.new(col_mean, 10),
     textcolor = color.black,
     text      = "M↓",
     size      = size.small)

// ─────────────────────────────────────────────────────────────────────────────
// STATE DOTS  (debug only, OFF by default)
// ─────────────────────────────────────────────────────────────────────────────
plotshape(show_state_dots and bull_state == 1 and not long_signal,
     title = "Bull Setup Active", style = shape.circle,
     location = location.belowbar, color = color.new(color.lime, 50), size = size.tiny)

plotshape(show_state_dots and bull_state == 2,
     title = "Bull Retest Ready", style = shape.triangleup,
     location = location.belowbar, color = color.new(color.lime, 20), size = size.tiny)

plotshape(show_state_dots and bear_state == 1 and not short_signal,
     title = "Bear Setup Active", style = shape.circle,
     location = location.abovebar, color = color.new(color.red, 50), size = size.tiny)

plotshape(show_state_dots and bear_state == 2,
     title = "Bear Retest Ready", style = shape.triangledown,
     location = location.abovebar, color = color.new(color.red, 20), size = size.tiny)

// ─────────────────────────────────────────────────────────────────────────────
// SL / ENTRY LINES + PRICE AXIS LABELS
// ─────────────────────────────────────────────────────────────────────────────
var line  sl_line      = na
var line  entry_line   = na
var label sl_axis_lbl  = na
var label en_axis_lbl  = na
var float active_sl    = na
var float active_entry = na
var string active_dir  = ""

if long_signal
    if not na(sl_line)
        line.delete(sl_line)
    if not na(entry_line)
        line.delete(entry_line)
    active_sl    := low
    active_entry := close
    active_dir   := "long"
    if show_sl_line
        sl_line    := line.new(bar_index, active_sl,    bar_index + 1, active_sl,
             color = sl_col, width = 1, style = line.style_dashed, extend = extend.right)
        entry_line := line.new(bar_index, active_entry, bar_index + 1, active_entry,
             color = en_col, width = 1, style = line.style_dashed, extend = extend.right)

if short_signal
    if not na(sl_line)
        line.delete(sl_line)
    if not na(entry_line)
        line.delete(entry_line)
    active_sl    := high
    active_entry := close
    active_dir   := "short"
    if show_sl_line
        sl_line    := line.new(bar_index, active_sl,    bar_index + 1, active_sl,
             color = sl_col, width = 1, style = line.style_dashed, extend = extend.right)
        entry_line := line.new(bar_index, active_entry, bar_index + 1, active_entry,
             color = en_col, width = 1, style = line.style_dashed, extend = extend.right)

if barstate.islast and show_sl_label and not na(active_sl)
    if not na(sl_axis_lbl)
        label.delete(sl_axis_lbl)
    if not na(en_axis_lbl)
        label.delete(en_axis_lbl)
    dir_icon = active_dir == "long" ? "▲" : "▼"
    sl_axis_lbl := label.new(
         x = bar_index + 3, y = active_sl,
         text  = "◀ SL   " + str.tostring(active_sl,    format.mintick),
         xloc  = xloc.bar_index, style = label.style_label_left,
         color = sl_col, textcolor = color.white, size = size.small)
    en_axis_lbl := label.new(
         x = bar_index + 3, y = active_entry,
         text  = "◀ ENTRY  " + dir_icon + "  " + str.tostring(active_entry, format.mintick),
         xloc  = xloc.bar_index, style = label.style_label_left,
         color = en_col, textcolor = color.black, size = size.small)

// ─────────────────────────────────────────────────────────────────────────────
// STRATEGY ENTRIES
// ─────────────────────────────────────────────────────────────────────────────
if long_signal
    strategy.entry("Long",  strategy.long,  stop = active_sl)

if short_signal
    strategy.entry("Short", strategy.short, stop = active_sl)

// ─────────────────────────────────────────────────────────────────────────────
// ALERTS
// ─────────────────────────────────────────────────────────────────────────────
alertcondition(long_signal,     "LONG Signal",             "NY 5m Retest Breakout — LONG  |  Entry: close  |  SL: candle low")
alertcondition(short_signal,    "SHORT Signal",            "NY 5m Retest Breakdown — SHORT  |  Entry: close  |  SL: candle high")
alertcondition(bull_state == 2, "Bull Retest Zone Active", "5m hi_5m retested — watching for long re-break")
alertcondition(bear_state == 2, "Bear Retest Zone Active", "5m lo_5m retested — watching for short re-break")
alertcondition(mean_bull_rej,   "Mean Bounce ▲",           "Price bounced UP off 5m mean — bullish rejection of mean")
alertcondition(mean_bear_rej,   "Mean Rejection ▼",        "Price rejected DOWN off 5m mean — bearish rejection of mean")
alertcondition(not na(median_5m) and math.abs(close - median_5m) <= mean_rej_tol * syminfo.mintick and in_ny, "Price at 5m Median", "Price is touching the 5m median level")

// ─────────────────────────────────────────────────────────────────────────────
// LEGEND TABLE
// ─────────────────────────────────────────────────────────────────────────────
var table leg = table.new(
     position.top_right, 3, 15,
     bgcolor      = color.new(color.black, 70),
     border_color = color.new(color.white, 55),
     border_width = 1,
     frame_color  = color.new(color.white, 35),
     frame_width  = 1)

if barstate.islast
    hb  = color.new(color.gray, 45)
    sbg = color.new(color.black, 55)

    table.cell(leg, 0, 0, "Range",  text_color = color.white, text_size = size.small, bgcolor = hb)
    table.cell(leg, 1, 0, "Lock",   text_color = color.white, text_size = size.small, bgcolor = hb)
    table.cell(leg, 2, 0, "Color",  text_color = color.white, text_size = size.small, bgcolor = hb)

    table.cell(leg, 0, 1, "5m",     text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 1, "9:35",   text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 1, "",       bgcolor = color.red,       text_size = size.tiny)

    table.cell(leg, 0, 2, "10m",    text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 2, "9:40",   text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 2, "",       bgcolor = color.orange,    text_size = size.tiny)

    table.cell(leg, 0, 3, "15m",    text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 3, "9:45",   text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 3, "",       bgcolor = color.yellow,    text_size = size.tiny)

    table.cell(leg, 0, 4, "30m",    text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 4, "10:00",  text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 4, "",       bgcolor = color.green,     text_size = size.tiny)

    table.cell(leg, 0, 5, "60m",    text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 5, "10:30",  text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 5, "",       bgcolor = color.blue,      text_size = size.tiny)

    table.cell(leg, 0, 6, "4H",     text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 6, "1:30p",  text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 6, "",       bgcolor = #ca97ee,         text_size = size.tiny)

    table.cell(leg, 0, 7, "8H",     text_color = color.white, text_size = size.tiny)
    table.cell(leg, 1, 7, "6:00p",  text_color = color.white, text_size = size.tiny)
    table.cell(leg, 2, 7, "",       bgcolor = #8B00FF,         text_size = size.tiny)

    table.cell(leg, 0, 8, "DIV",    text_color = color.yellow, text_size = size.tiny)
    table.cell(leg, 1, 8, "RSI",    text_color = color.yellow, text_size = size.tiny)
    table.cell(leg, 2, 8, "",       bgcolor = color.new(color.purple, 30), text_size = size.tiny)

    // ── 5m Mean row ─────────────────────────────────────────────────
    mean_txt = not na(mean_5m) ? str.tostring(mean_5m, format.mintick) : "—"
    table.cell(leg, 0, 9, "5m Mean", text_color = col_mean,    text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 9, mean_txt,  text_color = col_mean,    text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 9, "",        bgcolor = color.new(col_mean, 40), text_size = size.tiny)

    // ── 5m Median row ───────────────────────────────────────────────
    med_txt = not na(median_5m) ? str.tostring(median_5m, format.mintick) : "—"
    table.cell(leg, 0, 10, "5m Median", text_color = col_median,   text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 10, med_txt,     text_color = col_median,   text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 10, "",          bgcolor = color.new(col_median, 40), text_size = size.tiny)

    // ── Mean rejection last signal ───────────────────────────────────
    rej_txt = mean_bull_rej ? "M↑ Bounce" : mean_bear_rej ? "M↓ Reject" : "—"
    rej_col = mean_bull_rej ? color.lime   : mean_bear_rej ? color.red   : color.gray
    table.cell(leg, 0, 11, "Mean Sig", text_color = color.silver, text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 11, rej_txt,    text_color = rej_col,      text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 11, "",         bgcolor = color.new(rej_col, 55), text_size = size.tiny)

    // ── Retest state readout ────────────────────────────────────────
    bull_st_txt = bull_state == 0 ? "idle" : bull_state == 1 ? "① broke ▲" : "② retest ▲"
    bear_st_txt = bear_state == 0 ? "idle" : bear_state == 1 ? "① broke ▼" : "② retest ▼"
    bull_st_col = bull_state == 0 ? color.gray : bull_state == 1 ? color.new(color.lime, 40) : color.lime
    bear_st_col = bear_state == 0 ? color.gray : bear_state == 1 ? color.new(color.red,  40) : color.red

    table.cell(leg, 0, 12, "Bull",      text_color = color.silver,  text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 12, bull_st_txt, text_color = bull_st_col,   text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 12, "",          bgcolor = bull_st_col,       text_size = size.tiny)

    table.cell(leg, 0, 13, "Bear",      text_color = color.silver,  text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 13, bear_st_txt, text_color = bear_st_col,   text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 13, "",          bgcolor = bear_st_col,       text_size = size.tiny)

    sl_txt = not na(active_sl)    ? str.tostring(active_sl,    format.mintick) : "—"
    en_txt = not na(active_entry) ? str.tostring(active_entry, format.mintick) : "—"
    table.cell(leg, 0, 14, "SL / Entry", text_color = color.silver, text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 1, 14, sl_txt,       text_color = sl_col,        text_size = size.tiny, bgcolor = sbg)
    table.cell(leg, 2, 14, en_txt,       text_color = en_col,        text_size = size.tiny, bgcolor = sbg)
