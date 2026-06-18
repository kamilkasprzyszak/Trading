//@version=6

indicator("Historical Quant AVP - S/D Zones V6 MAX \[KDE]", overlay=true, max\_boxes\_count=500, max\_bars\_back=5000, max\_polylines\_count=100)



// --- USTAWIENIA: BAZOWE ---

anchor\_tf = input.string("1D", "Zasięg Profilu (Anchor)", options=\["1H", "4H", "1D", "1W", "1M", "3M", "12M"])

history\_count = input.int(5, "Ilość historycznych okresów", minval=0, maxval=15)

rows = input.int(50, "Rozdzielczość (ilość rzędów)", minval=10, maxval=100)

smooth\_window = input.int(3, "Wygładzanie (Pseudo-KDE)", minval=1, maxval=9, step=2, tooltip="Ustaw 1, aby wyłączyć. Wartości 3, 5, 7 wygładzają szum i redukują fałszywe szczyty (Prominencja).")



// --- USTAWIENIA: METODY ILOŚCIOWE ---

hvn\_method = input.string("Kwantyle (Percentyl)", "Metoda Obliczeń HVN", options=\["Kwantyle (Percentyl)", "Z-Score", "Prominencja (Lokalne Szczyty)"])

quantile\_pct = input.int(85, "Kwantyle: Poziom Percentyla (np. 85 = top 15%)", minval=50, maxval=99)

z\_score\_val = input.float(1.2, "Z-Score: Odchylenie Standardowe", minval=0.5, step=0.1)



// --- USTAWIENIA: WIZUALIZACJA ---

grp\_vis = "Wizualizacja"

poc\_transp = input.int(30, "Przezroczystość POC", minval=0, maxval=100, group=grp\_vis)

min\_hvn\_transp = input.int(85, "Przezroczystość Progu", minval=0, maxval=100, group=grp\_vis)

show\_vol\_data = input.bool(true, "Pokaż Wolumen/Deltę", group=grp\_vis)

hide\_hist\_text = input.bool(true, "Ukryj tekst historii", group=grp\_vis)



// --- USTAWIENIA: PROJEKCJA ---

grp\_ext = "Projekcja Historycznych Stref"

extend\_zones = input.bool(true, "Przeciągaj historyczne strefy", group=grp\_ext)

extend\_transp = input.int(90, "Przezroczystość projekcji", minval=0, maxval=100, group=grp\_ext)



// Ustawienia dPOC

show\_dpoc = input.bool(true, "Pokaż przemieszczanie dPOC", group=grp\_vis)

dpoc\_color = input.color(color.new(color.orange, 0), "Kolor Linii dPOC", group=grp\_vis)



// --- USTAWIENIA: DRUGI POC (MTF) ---

grp\_sec = "Drugi POC (Higher Timeframe)"

show\_sec\_poc = input.bool(false, "Włącz Drugi POC", group=grp\_sec)

sec\_anchor\_tf = input.string("1W", "Zasięg dla 2. POC", options=\["1D", "1W", "1M", "3M", "12M"], group=grp\_sec)

sec\_poc\_color = input.color(#FF00FF, "Kolor 2. POC (Fuksja)", group=grp\_sec)



// --- DETEKCJA KOTWIC ---

get\_anchor() =>

&#x20;   bool is\_new = false

&#x20;   if anchor\_tf == "1H"

&#x20;       is\_new := ta.change(time("60")) != 0

&#x20;   else if anchor\_tf == "4H"

&#x20;       is\_new := ta.change(time("240")) != 0

&#x20;   else if anchor\_tf == "1D"

&#x20;       is\_new := ta.change(time("D")) != 0

&#x20;   else if anchor\_tf == "1W"

&#x20;       is\_new := ta.change(time("W")) != 0

&#x20;   else if anchor\_tf == "1M"

&#x20;       is\_new := ta.change(time("M")) != 0

&#x20;   else if anchor\_tf == "3M"

&#x20;       is\_new := ta.change(time("3M")) != 0

&#x20;   else if anchor\_tf == "12M"

&#x20;       is\_new := ta.change(time("12M")) != 0

&#x20;   is\_new



get\_sec\_anchor() =>

&#x20;   bool is\_new = false

&#x20;   if sec\_anchor\_tf == "1D"

&#x20;       is\_new := ta.change(time("D")) != 0

&#x20;   else if sec\_anchor\_tf == "1W"

&#x20;       is\_new := ta.change(time("W")) != 0

&#x20;   else if sec\_anchor\_tf == "1M"

&#x20;       is\_new := ta.change(time("M")) != 0

&#x20;   else if sec\_anchor\_tf == "3M"

&#x20;       is\_new := ta.change(time("3M")) != 0

&#x20;   else if sec\_anchor\_tf == "12M"

&#x20;       is\_new := ta.change(time("12M")) != 0

&#x20;   is\_new



bool is\_anchor = get\_anchor()

bool is\_sec\_anchor = get\_sec\_anchor()



var int\[] anchor\_indices = array.new\_int()

var int\[] sec\_anchor\_indices = array.new\_int()



if is\_anchor or bar\_index == 0

&#x20;   array.push(anchor\_indices, bar\_index)

&#x20;   if array.size(anchor\_indices) > history\_count + 1

&#x20;       array.shift(anchor\_indices)



if is\_sec\_anchor or bar\_index == 0

&#x20;   array.push(sec\_anchor\_indices, bar\_index)

&#x20;   if array.size(sec\_anchor\_indices) > history\_count + 1

&#x20;       array.shift(sec\_anchor\_indices)



var box\[] zoneBoxes = array.new\_box()

var polyline\[] dpocLines = array.new<polyline>()



// --- FUNKCJE MATEMATYCZNE ---

normalize\_transp(vol, minVol, maxVol, minOut, maxOut) =>

&#x20;   if maxVol == minVol

&#x20;       minOut

&#x20;   else

&#x20;       float mappedValue = maxOut - (( (vol - minVol) / (maxVol - minVol) ) \* (maxOut - minOut))

&#x20;       math.floor(math.max(mappedValue, 0))



format\_volume(v) =>

&#x20;   if math.abs(v) >= 1000000

&#x20;       str.tostring(v / 1000000, "#.##") + "M"

&#x20;   else if math.abs(v) >= 1000

&#x20;       str.tostring(v / 1000, "#.##") + "K"

&#x20;   else

&#x20;       str.tostring(math.round(v))



// Algorytm wygładzania tablicy (Pseudo-KDE)

smooth\_profile(arr, window) =>

&#x20;   int sz = array.size(arr)

&#x20;   float\[] res = array.copy(arr)

&#x20;   if window > 1

&#x20;       int half = window / 2

&#x20;       for i = 0 to sz - 1

&#x20;           float sum = 0.0

&#x20;           int count = 0

&#x20;           for j = math.max(0, i - half) to math.min(sz - 1, i + half)

&#x20;               sum += array.get(arr, j)

&#x20;               count += 1

&#x20;           array.set(res, i, sum / count)

&#x20;   res



// --- SILNIK OBLICZENIOWY ---

if barstate.islast

&#x20;   if array.size(zoneBoxes) > 0

&#x20;       for b in zoneBoxes

&#x20;           box.delete(b)

&#x20;       array.clear(zoneBoxes)

&#x20;       

&#x20;   if array.size(dpocLines) > 0

&#x20;       for pl in dpocLines

&#x20;           polyline.delete(pl)

&#x20;       array.clear(dpocLines)

&#x20;       

&#x20;   int num\_anchors = array.size(anchor\_indices)

&#x20;   int current\_bar\_edge = bar\_index + 15

&#x20;   

&#x20;   // ==========================================

&#x20;   // 1. GŁÓWNY PROFIL

&#x20;   // ==========================================

&#x20;   if num\_anchors > 0

&#x20;       for a = 0 to num\_anchors - 1

&#x20;           int start\_idx = array.get(anchor\_indices, a)

&#x20;           bool is\_current\_period = (a == num\_anchors - 1)

&#x20;           int end\_idx = is\_current\_period ? bar\_index : array.get(anchor\_indices, a + 1) - 1

&#x20;           int right\_edge = is\_current\_period ? current\_bar\_edge : end\_idx

&#x20;           

&#x20;           int bars\_back\_start = bar\_index - start\_idx

&#x20;           int bars\_back\_end = bar\_index - end\_idx

&#x20;           

&#x20;           if bars\_back\_start > 4900

&#x20;               continue

&#x20;               

&#x20;           if bars\_back\_start >= bars\_back\_end

&#x20;               float maxP = nz(high\[bars\_back\_end])

&#x20;               float minP = nz(low\[bars\_back\_end])

&#x20;               for i = bars\_back\_end to bars\_back\_start

&#x20;                   maxP := math.max(maxP, nz(high\[i]))

&#x20;                   minP := math.min(minP, nz(low\[i]))

&#x20;                   

&#x20;               float step = (maxP - minP) / rows

&#x20;               if step == 0

&#x20;                   step := syminfo.mintick

&#x20;                   

&#x20;               float\[] current\_levels = array.new\_float(rows + 1, 0.0)

&#x20;               float\[] raw\_vol = array.new\_float(rows, 0.0)

&#x20;               float\[] current\_delta = array.new\_float(rows, 0.0) 

&#x20;               

&#x20;               for i = 0 to rows

&#x20;                   array.set(current\_levels, i, minP + step \* i)

&#x20;                   

&#x20;               float max\_vol\_so\_far = 0.0

&#x20;               int current\_dpoc\_idx = 0

&#x20;               chart.point\[] dpoc\_pts = array.new<chart.point>()

&#x20;                   

&#x20;               // Agregacja surowych danych

&#x20;               for i = bars\_back\_start to bars\_back\_end by 1

&#x20;                   float v = nz(volume\[i], 0)

&#x20;                   float h = nz(high\[i])

&#x20;                   float l = nz(low\[i])

&#x20;                   float c = nz(close\[i])

&#x20;                   

&#x20;                   if v > 0

&#x20;                       int low\_idx = math.floor((l - minP) / step)

&#x20;                       int high\_idx = math.floor((h - minP) / step)

&#x20;                       

&#x20;                       low\_idx := math.max(0, math.min(low\_idx, rows - 1))

&#x20;                       high\_idx := math.max(0, math.min(high\_idx, rows - 1))

&#x20;                       

&#x20;                       if high\_idx < low\_idx

&#x20;                           high\_idx := low\_idx

&#x20;                           

&#x20;                       int span = high\_idx - low\_idx + 1

&#x20;                       float vol\_per\_bin = v / span

&#x20;                       

&#x20;                       float range\_hl = math.max(h - l, syminfo.mintick)

&#x20;                       float buy\_pct = (c - l) / range\_hl

&#x20;                       float sell\_pct = 1.0 - buy\_pct

&#x20;                       float delta\_value = (v \* buy\_pct) - (v \* sell\_pct)

&#x20;                       float delta\_per\_bin = delta\_value / span

&#x20;                       

&#x20;                       for b\_idx = low\_idx to high\_idx

&#x20;                           float curr\_v = array.get(raw\_vol, b\_idx)

&#x20;                           float updated\_v = curr\_v + vol\_per\_bin

&#x20;                           array.set(raw\_vol, b\_idx, updated\_v)

&#x20;                           

&#x20;                           float curr\_d = array.get(current\_delta, b\_idx)

&#x20;                           array.set(current\_delta, b\_idx, curr\_d + delta\_per\_bin)

&#x20;                           

&#x20;                           // dPOC podąża za surowym obrotem, co jest pożądane

&#x20;                           if updated\_v > max\_vol\_so\_far

&#x20;                               max\_vol\_so\_far := updated\_v

&#x20;                               current\_dpoc\_idx := b\_idx

&#x20;                       

&#x20;                       if show\_dpoc

&#x20;                           float dpoc\_price = minP + step \* current\_dpoc\_idx + (step / 2.0)

&#x20;                           array.push(dpoc\_pts, chart.point.from\_index(bar\_index - i, dpoc\_price))

&#x20;                       

&#x20;               if show\_dpoc and array.size(dpoc\_pts) > 1

&#x20;                   array.push(dpocLines, polyline.new(dpoc\_pts, curved=false, line\_color=dpoc\_color, line\_width=2))

&#x20;               

&#x20;               // --- ZASTOSOWANIE WYGŁADZANIA KDE ---

&#x20;               float\[] smoothed\_vol = smooth\_profile(raw\_vol, smooth\_window)

&#x20;                   

&#x20;               float avgVol = array.sum(smoothed\_vol) / rows

&#x20;               float pocVol = array.max(smoothed\_vol)

&#x20;               int pocIdx = array.indexof(smoothed\_vol, pocVol)

&#x20;               

&#x20;               float hvnThreshold = 0.0

&#x20;               if hvn\_method == "Kwantyle (Percentyl)"

&#x20;                   float\[] sorted\_vol = array.copy(smoothed\_vol)

&#x20;                   array.sort(sorted\_vol)

&#x20;                   int pct\_idx = math.floor((rows - 1) \* (quantile\_pct / 100.0))

&#x20;                   hvnThreshold := array.get(sorted\_vol, pct\_idx)

&#x20;               else if hvn\_method == "Z-Score"

&#x20;                   float sum\_sq = 0.0

&#x20;                   for v in smoothed\_vol

&#x20;                       sum\_sq += math.pow(v - avgVol, 2)

&#x20;                   float stdev = math.sqrt(sum\_sq / rows)

&#x20;                   hvnThreshold := avgVol + (z\_score\_val \* stdev)

&#x20;               else if hvn\_method == "Prominencja (Lokalne Szczyty)"

&#x20;                   hvnThreshold := avgVol

&#x20;                   

&#x20;               for i = 0 to rows - 1

&#x20;                   float cVol\_smooth = array.get(smoothed\_vol, i)

&#x20;                   float cVol\_raw = array.get(raw\_vol, i) // Do wyświetlania

&#x20;                   float cDelta = array.get(current\_delta, i)

&#x20;                   

&#x20;                   bool is\_local\_max = true

&#x20;                   if hvn\_method == "Prominencja (Lokalne Szczyty)"

&#x20;                       float prev\_v = i > 0 ? array.get(smoothed\_vol, i - 1) : 0.0

&#x20;                       float next\_v = i < rows - 1 ? array.get(smoothed\_vol, i + 1) : 0.0

&#x20;                       is\_local\_max := (cVol\_smooth > prev\_v) and (cVol\_smooth >= next\_v)

&#x20;                   

&#x20;                   // Decyzje o HVN zapadają na podstawie Wygładzonej Tablicy

&#x20;                   if cVol\_smooth >= hvnThreshold and cVol\_smooth > 0 and (hvn\_method != "Prominencja (Lokalne Szczyty)" or is\_local\_max)

&#x20;                       float levelLow = array.get(current\_levels, i)

&#x20;                       float levelHigh = array.get(current\_levels, i + 1)

&#x20;                       

&#x20;                       string zoneType = ""

&#x20;                       color baseColor = color.gray

&#x20;                       

&#x20;                       int dynamicTransp = normalize\_transp(cVol\_smooth, hvnThreshold, pocVol, poc\_transp, min\_hvn\_transp)

&#x20;                       

&#x20;                       if i == pocIdx

&#x20;                           zoneType := "POC"

&#x20;                           baseColor := #FFD700

&#x20;                       else if levelHigh > close

&#x20;                           zoneType := "SUPPLY"

&#x20;                           baseColor := color.red

&#x20;                       else

&#x20;                           zoneType := "DEMAND"

&#x20;                           baseColor := color.teal

&#x20;                       

&#x20;                       string zoneText = zoneType

&#x20;                       if show\_vol\_data

&#x20;                           string sign = cDelta > 0 ? "+" : ""

&#x20;                           zoneText += " \[Vol: " + format\_volume(cVol\_raw) + " | Δ: " + sign + format\_volume(cDelta) + "]"

&#x20;                           

&#x20;                       if not is\_current\_period and hide\_hist\_text

&#x20;                           zoneText := ""

&#x20;                           

&#x20;                       color finalColor = color.new(baseColor, dynamicTransp)

&#x20;                       

&#x20;                       box newBox = box.new(left=start\_idx, top=levelHigh, right=right\_edge, bottom=levelLow, border\_color=color.new(finalColor, 100), bgcolor=finalColor, text=zoneText, text\_color=color.white, text\_size=size.tiny, text\_halign=text.align\_right)

&#x20;                       array.push(zoneBoxes, newBox)

&#x20;                       

&#x20;                       if not is\_current\_period and extend\_zones

&#x20;                           color extColor = color.new(baseColor, extend\_transp)

&#x20;                           color extBorder = color.new(baseColor, math.min(extend\_transp + 15, 100))

&#x20;                           box extBox = box.new(left=end\_idx, top=levelHigh, right=current\_bar\_edge, bottom=levelLow, border\_color=extBorder, bgcolor=extColor)

&#x20;                           array.push(zoneBoxes, extBox)



&#x20;   // ==========================================

&#x20;   // 2. OBLICZANIE MTF POC

&#x20;   // ==========================================

&#x20;   int num\_sec\_anchors = array.size(sec\_anchor\_indices)

&#x20;   if show\_sec\_poc and num\_sec\_anchors > 0

&#x20;       for a = 0 to num\_sec\_anchors - 1

&#x20;           int start\_idx = array.get(sec\_anchor\_indices, a)

&#x20;           bool is\_current\_period = (a == num\_sec\_anchors - 1)

&#x20;           int end\_idx = is\_current\_period ? bar\_index : array.get(sec\_anchor\_indices, a + 1) - 1

&#x20;           int right\_edge = is\_current\_period ? current\_bar\_edge : end\_idx

&#x20;           

&#x20;           int bars\_back\_start = bar\_index - start\_idx

&#x20;           int bars\_back\_end = bar\_index - end\_idx

&#x20;           

&#x20;           if bars\_back\_start > 4900

&#x20;               continue

&#x20;               

&#x20;           if bars\_back\_start >= bars\_back\_end

&#x20;               float maxP = nz(high\[bars\_back\_end])

&#x20;               float minP = nz(low\[bars\_back\_end])

&#x20;               for i = bars\_back\_end to bars\_back\_start

&#x20;                   maxP := math.max(maxP, nz(high\[i]))

&#x20;                   minP := math.min(minP, nz(low\[i]))

&#x20;                   

&#x20;               float step = (maxP - minP) / rows

&#x20;               if step == 0

&#x20;                   step := syminfo.mintick

&#x20;                   

&#x20;               float\[] sec\_raw\_vol = array.new\_float(rows, 0.0)

&#x20;               

&#x20;               for i = bars\_back\_start to bars\_back\_end by 1

&#x20;                   float v = nz(volume\[i], 0)

&#x20;                   float h = nz(high\[i])

&#x20;                   float l = nz(low\[i])

&#x20;                   

&#x20;                   if v > 0

&#x20;                       int low\_idx = math.floor((l - minP) / step)

&#x20;                       int high\_idx = math.floor((h - minP) / step)

&#x20;                       

&#x20;                       low\_idx := math.max(0, math.min(low\_idx, rows - 1))

&#x20;                       high\_idx := math.max(0, math.min(high\_idx, rows - 1))

&#x20;                       

&#x20;                       if high\_idx < low\_idx

&#x20;                           high\_idx := low\_idx

&#x20;                           

&#x20;                       int span = high\_idx - low\_idx + 1

&#x20;                       float vol\_per\_bin = v / span

&#x20;                       

&#x20;                       for b\_idx = low\_idx to high\_idx

&#x20;                           array.set(sec\_raw\_vol, b\_idx, array.get(sec\_raw\_vol, b\_idx) + vol\_per\_bin)

&#x20;               

&#x20;               // Wygładzanie również dla MTF POC

&#x20;               float\[] sec\_smoothed\_vol = smooth\_profile(sec\_raw\_vol, smooth\_window)

&#x20;                   

&#x20;               float pocVol = array.max(sec\_smoothed\_vol)

&#x20;               int pocIdx = array.indexof(sec\_smoothed\_vol, pocVol)

&#x20;               

&#x20;               float pocLow = minP + step \* pocIdx

&#x20;               float pocHigh = pocLow + step

&#x20;               

&#x20;               string sec\_text = "POC " + sec\_anchor\_tf

&#x20;               if not is\_current\_period and hide\_hist\_text

&#x20;                   sec\_text := ""

&#x20;               

&#x20;               box secBox = box.new(left=start\_idx, top=pocHigh, right=right\_edge, bottom=pocLow, border\_color=sec\_poc\_color, bgcolor=color.new(sec\_poc\_color, 40), text=sec\_text, text\_color=color.white, text\_size=size.tiny, text\_halign=text.align\_right)

&#x20;               array.push(zoneBoxes, secBox)

&#x20;               

&#x20;               if not is\_current\_period and extend\_zones

&#x20;                   color extSecColor = color.new(sec\_poc\_color, extend\_transp)

&#x20;                   box extSecBox = box.new(left=end\_idx, top=pocHigh, right=current\_bar\_edge, bottom=pocLow, border\_color=color.new(sec\_poc\_color, 100), bgcolor=extSecColor)

&#x20;                   array.push(zoneBoxes, extSecBox)

