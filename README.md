//# Ny-Rat-ifi-Senti-by 1vb0
//@version=6
indicator("NY Rat-ifi Zone Senti by 1vb0", overlay = true, max_lines_count = 500, max_labels_count = 200, max_boxes_count = 200)

// ─────────────────────────────────────────────────────────────────────────────
// INPUTS
// ─────────────────────────────────────────────────────────────────────────────
rsi_length   = input.int(14,   "RSI Length",                 minval = 2)
rsi_src      = input.source(close, "RSI Source")
div_lookback = input.int(5,    "Divergence Lookback (bars)", minval = 2, maxval = 20)
show_labels  = input.bool(true, "Show Long/Short Labels")
show_ranges  = input.bool(true, "Show Range Lines")

grp_zone     = "═══ 5m Zone Highlight ═══"
show_5m_zone = input.bool(true,  "Show 5m Zone Highlight",          group = grp_zone)
col_5m_bull  = input.color(color.new(color.green, 75), "Bull Color (close above hi_5m)", group = grp_zone)
col_5m_bear  = input.color(color.new(color.red,   75), "Bear Color (close below lo_5m)", group = grp_zone)
col_5m_neut  = input.color(color.new(color.gray,  88), "Neutral Color (inside range)",   group = grp_zone)

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
// ROLLING HIGH / LOW — accumulates from 09:30, frozen at each snapshot
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
// COLORS
// ─────────────────────────────────────────────────────────────────────────────
c_5m  = color.new(color.red,    0)
c_10m = color.new(color.orange, 0)
c_15m = color.new(color.yellow, 0)
c_30m = color.new(color.green,  0)
c_60m = color.new(color.blue,   0)
c_4h  = color.new(#ca97ee, 0)
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

bgcolor(snap_930 ? color.new(color.white, 80) : na, title = "NY Open Flash")

// ─────────────────────────────────────────────────────────────────────────────
// RANGE LABELS
// ─────────────────────────────────────────────────────────────────────────────
if show_ranges
    if snap_935
        label.new(x = time, y = hi_5m,  text = "5m H",  xloc = xloc.bar_time, style = label.style_label_left, color = c_5m,  textcolor = color.white, size = size.tiny)
        label.new(x = time, y = lo_5m,  text = "5m L",  xloc = xloc.bar_time, style = label.style_label_left, color = c_5m,  textcolor = color.white, size = size.tiny)

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
// 5M ZONE HIGHLIGHT BOX
// Box sits strictly between lo_5m (bottom) and hi_5m (top).
// Color:
//   green  — most recent close was ABOVE hi_5m
//   red    — most recent close was BELOW lo_5m
//   neutral — close is inside the range
// Box is created at 9:35 and stretches right every bar through the NY session.
// Resets at next day's 9:30.
// ─────────────────────────────────────────────────────────────────────────────
var box  zone_box_5m  = na
var color zone_col_5m = na

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
    // Stretch right every bar
    box.set_right(zone_box_5m, bar_index + 1)

    // Update color based on last close vs the locked 5m levels
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

pivot_lo_price = ta.pivotlow( low,     div_lookback, div_lookback)
pivot_hi_price = ta.pivothigh(high,    div_lookback, div_lookback)
pivot_lo_rsi   = ta.pivotlow( rsi_val, div_lookback, div_lookback)
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

bull_div = not na(prev_lo_price) and not na(last_lo_price) and
           last_lo_price < prev_lo_price and
           last_lo_rsi   > prev_lo_rsi

bear_div = not na(prev_hi_price) and not na(last_hi_price) and
           last_hi_price > prev_hi_price and
           last_hi_rsi   < prev_hi_rsi

// ─────────────────────────────────────────────────────────────────────────────
// BREAKOUT SIGNALS
// ─────────────────────────────────────────────────────────────────────────────
any_range_hi = math.max(
     na(hi_5m)  ? -1e10 : hi_5m,
     na(hi_10m) ? -1e10 : hi_10m,
     na(hi_15m) ? -1e10 : hi_15m,
     na(hi_30m) ? -1e10 : hi_30m,
     na(hi_60m) ? -1e10 : hi_60m,
     na(hi_4h)  ? -1e10 : hi_4h,
     na(hi_8h)  ? -1e10 : hi_8h)

any_range_lo = math.min(
     na(lo_5m)  ?  1e10 : lo_5m,
     na(lo_10m) ?  1e10 : lo_10m,
     na(lo_15m) ?  1e10 : lo_15m,
     na(lo_30m) ?  1e10 : lo_30m,
     na(lo_60m) ?  1e10 : lo_60m,
     na(lo_4h)  ?  1e10 : lo_4h,
     na(lo_8h)  ?  1e10 : lo_8h)

broke_high = close > any_range_hi and any_range_hi != -1e10
broke_low  = close < any_range_lo and any_range_lo !=  1e10

long_signal  = broke_high and bull_div and in_ny
short_signal = broke_low  and bear_div and in_ny

// ─────────────────────────────────────────────────────────────────────────────
// SIGNAL SHAPES
// ─────────────────────────────────────────────────────────────────────────────
plotshape(show_labels and long_signal,
     title = "LONG",  style = shape.labelup,   location = location.belowbar,
     color = color.new(color.lime, 5), textcolor = color.black,
     text = "LONG",   size = size.normal)

plotshape(show_labels and short_signal,
     title = "SHORT", style = shape.labeldown, location = location.abovebar,
     color = color.new(color.red, 5),  textcolor = color.white,
     text = "SHORT",  size = size.normal)

// ─────────────────────────────────────────────────────────────────────────────
// ALERTS
// ─────────────────────────────────────────────────────────────────────────────
alertcondition(long_signal,  "LONG Signal",  "NY Range Breakout — LONG confirmed by bullish RSI divergence")
alertcondition(short_signal, "SHORT Signal", "NY Range Breakout — SHORT confirmed by bearish RSI divergence")

// ─────────────────────────────────────────────────────────────────────────────
// LEGEND TABLE
// ─────────────────────────────────────────────────────────────────────────────
var table leg = table.new(
     position.top_right, 3, 9,
     bgcolor      = color.new(color.black, 70),
     border_color = color.new(color.white, 55),
     border_width = 1,
     frame_color  = color.new(color.white, 35),
     frame_width  = 1)

if barstate.islast
    table.cell(leg, 0, 0, "Range", text_color = color.white,  text_size = size.small, bgcolor = color.new(color.gray, 45))
    table.cell(leg, 1, 0, "Lock",  text_color = color.white,  text_size = size.small, bgcolor = color.new(color.gray, 45))
    table.cell(leg, 2, 0, "Color", text_color = color.white,  text_size = size.small, bgcolor = color.new(color.gray, 45))

    table.cell(leg, 0, 1, "5m",    text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 1, "9:35",  text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 1, "",      bgcolor = color.red,        text_size = size.tiny)

    table.cell(leg, 0, 2, "10m",   text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 2, "9:40",  text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 2, "",      bgcolor = color.orange,     text_size = size.tiny)

    table.cell(leg, 0, 3, "15m",   text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 3, "9:45",  text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 3, "",      bgcolor = color.yellow,     text_size = size.tiny)

    table.cell(leg, 0, 4, "30m",   text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 4, "10:00", text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 4, "",      bgcolor = color.green,      text_size = size.tiny)

    table.cell(leg, 0, 5, "60m",   text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 5, "10:30", text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 5, "",      bgcolor = color.blue,       text_size = size.tiny)

    table.cell(leg, 0, 6, "4H",    text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 6, "1:30p", text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 6, "",      bgcolor = #ca97ee,          text_size = size.tiny)

    table.cell(leg, 0, 7, "8H",    text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 1, 7, "6:00p", text_color = color.white,  text_size = size.tiny)
    table.cell(leg, 2, 7, "",      bgcolor = #8B00FF,          text_size = size.tiny)

    table.cell(leg, 0, 8, "DIV",   text_color = color.yellow, text_size = size.tiny)
    table.cell(leg, 1, 8, "RSI",   text_color = color.yellow, text_size = size.tiny)
    table.cell(leg, 2, 8, "",      bgcolor = color.new(color.purple, 30), text_size = size.tiny) 
