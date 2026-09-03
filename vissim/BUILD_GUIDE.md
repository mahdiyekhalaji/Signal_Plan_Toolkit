# Building the Yonge and Steeles model in Vissim, step by step

Written for someone opening Vissim for the first time. Every number you need is
in this file, so you should not have to look anything up while you work.

Menu names below match Vissim 2023 and 2024. If a menu is missing, check the
version under Help, About, and look for the same words in a nearby menu.

Set aside about three hours for the first alternative. The other three take
twenty minutes each once the network exists.

---

## 0. Before you start

Open Vissim. Go to **Base Data, Network Settings**, tab **Units**, and set units
to **Metric**. Everything in this project is in metres and km/h.

Go to **File, Save As** and save the file now as `steeles_yonge_A.inpx` in your
project folder. Save every fifteen minutes. Vissim does not autosave.

Turn on the toolbar you will use most: **View, Toolbars**, tick **Network
Editor**.

---

## 1. Bring in the drawing as a background

You are going to trace over the CAD drawing rather than guess distances.

1. In the **Network Objects** panel on the left, right-click **Background
   Images**, choose **Add New Background Image**.
2. Select `output/steeles-yonge-plan.dxf`. If Vissim refuses the DXF, open it in
   AutoCAD and export a PNG, then use that.
3. The image lands at an arbitrary size. Right-click it in the list, choose
   **Set Scale**. Click one end of the scale bar in the drawing, then the other,
   and type **50** when it asks for the distance in metres. The drawing's scale
   bar runs 0 to 50 m.
4. Check it worked: the road width on Yonge should measure about 23 m across.

If the image sits on top of everything later, right-click it and send it behind,
or untick its visibility while you draw.

---

## 2. Draw the links

A **link** is a one-way road segment carrying one or more lanes. You need eight:
four approaches feeding the intersection and four departures leaving it.

To draw one: hold the **right mouse button** and drag along the direction of
travel. Release. A dialog opens.

In that dialog set:

- **Name**: `NB approach`, `SB departure`, and so on. Name them properly, you
  will be picking them from lists later.
- **Num lanes**: see the table below.
- **Lane width**: 3.5 m for all.
- **Behavior type**: Urban (motorized).

| Link | Lanes | Length to draw |
|---|---|---|
| NB approach (from the south, heading north) | 4 | about 250 m |
| NB departure (leaving to the north) | 3 | about 150 m |
| SB approach (from the north, heading south) | 4 | 250 m |
| SB departure | 3 | 150 m |
| EB approach (from the west, heading east) | 4 | 250 m |
| EB departure | 3 | 150 m |
| WB approach (from the east, heading west) | 4 | 250 m |
| WB departure | 3 | 150 m |

Four lanes on each approach because there is a left turn lane, two through
lanes and a right turn lane. Three on each departure because the turn lane ends
at the intersection.

Approaches should be long enough to hold the queue. 250 m is generous; if you
see vehicles queueing off the end of a link during the run, lengthen it.

---

## 3. Connect the links with connectors

A **connector** is the path a vehicle takes from one link to another, and it is
how Vissim knows which movements exist. You need twelve, one per movement.

To draw one: hold the **right mouse button**, start on the *lane* of the
approach link, drag to the target *lane* of the departure link, release.

Which lane connects to which:

| Movement | From approach lane | To departure lane |
|---|---|---|
| NBL | lane 1 (leftmost, next to centreline) | leftmost lane of WB departure |
| NBT | lanes 2 and 3 | matching through lanes of NB departure |
| NBR | lane 4 (curb lane) | rightmost lane of EB departure |

Same pattern for the other three approaches. Left turns go to the departure on
your left, right turns to the one on your right.

Lane numbering in Vissim starts at 1 on the **right** by default, so check the
lane numbers in the connector dialog rather than trusting the count direction.

For the through movement, drag from lane 2 to lane 2 and lane 3 to lane 3, or
select both lanes at once in the connector dialog using the From lane and To
lane range fields.

**Checkpoint**: right-click empty space, choose **Show, Connectors**. You should
see twelve paths and no gaps. A missing connector means that movement silently
carries zero traffic later.

---

## 4. Set speeds

**Base Data, Distributions, Desired Speed**. There is a default 50 km/h
distribution. Confirm it exists; the posted speed on both streets here is 50.

Right turns and left turns should slow down. Select each turning connector,
open it, and set **Desired speed decision** is not needed, but do set the
connector's own **Desired speed distribution** if the field is present, or add a
**Reduced Speed Area** on each turn connector: right-click on the connector,
**Add New Reduced Speed Area**, length 20 m, desired speed 25 km/h for lefts and
20 km/h for rights.

Skip this on a first pass if it is slowing you down. It affects delay by a few
seconds, not by a level of service.

---

## 5. Vehicle inputs

This is the demand entering each approach.

**Lists, Private Transport, Vehicle Inputs**, or the toolbar icon. Click the
**+** to add a row.

| Link | Volume (veh/h) | Time interval |
|---|---|---|
| NB approach | 1218 | 0 to 3600 |
| SB approach | 1179 | 0 to 3600 |
| EB approach | 1152 | 0 to 3600 |
| WB approach | 1360 | 0 to 3600 |

These are PM peak hour totals from the City count on 30 June 2026.

**Vehicle composition**: in the same list there is a Vehicle Composition column.
Open **Base Data, Distributions, Vehicle Compositions**, edit the default so it
is roughly 97% Car and 3% HGV. The count shows 4.7% heavy across the day and
about 3% in the PM peak.

---

## 6. Routing

Vehicles need to be told where to go, otherwise they all go straight.

**Lists, Private Transport, Vehicle Routes, Static**, or right-click on an
approach link and choose **Add New Vehicle Routing Decision, Static**.

Place the routing decision on the approach link about **180 m upstream** of the
stop bar, so vehicles have room to get into the correct lane.

Then for each decision, add three routes, one per movement, ending on the
relevant departure link. Set **RelFlow** to these values:

| Approach | Left | Through | Right |
|---|---|---|---|
| NB | 0.1100 | 0.7750 | 0.1149 |
| SB | 0.1281 | 0.8278 | 0.0441 |
| EB | 0.1589 | 0.6693 | 0.1719 |
| WB | 0.1868 | 0.5669 | 0.2463 |

RelFlow values are relative, so they do not need to sum to exactly 1.

**Checkpoint**: run the simulation for a few seconds without signals. Traffic
should flow through and turn in all directions. If everything goes straight, the
routes are not attached to the right link.

---

## 7. Signal controller

**Signal Control, Signal Controllers**, click **+**.

- **Name**: `PX131 Alternative A`
- **Type**: Fixed Time
- **Cycle time**: 130

Click **Edit Signal Control**. In the fixed-time editor, create four **signal
groups**:

| No. | Name |
|---|---|
| 1 | NS left |
| 2 | NS through |
| 3 | EW left |
| 4 | EW through |

Now enter the timing. These are the start and end times within the 130 s cycle,
already worked out from `controller_A.csv`:

| Signal group | Green start | Green end | Amber end |
|---|---|---|---|
| 1 NS left | 0.0 | 13.2 | 16.7 |
| 2 NS through | 18.2 | 60.3 | 63.8 |
| 3 EW left | 65.3 | 85.8 | 89.3 |
| 4 EW through | 90.8 | 125.0 | 128.5 |

Amber is 3.5 s and the all-red that follows is 1.5 s. The gap between one
group's amber end and the next group's green start is that all-red.

Round to whole seconds if the editor insists: 0/13/17, 18/60/64, 65/86/89,
91/125/129.

---

## 8. Signal heads

A signal head is the stop line. Without one, the controller does nothing.

Right-click on a **lane** at the stop bar location, choose **Add New Signal
Head**. In the dialog:

- **Signal controller**: the one you just made
- **Signal group**: which group governs that lane

Place one head per lane:

| Lane | Signal group |
|---|---|
| NB and SB left turn lanes | 1 NS left |
| NB and SB through and right lanes | 2 NS through |
| EB and WB left turn lanes | 3 EW left |
| EB and WB through and right lanes | 4 EW through |

That is 16 heads in total, four per approach.

Position them right at the stop bar, about 2 m upstream of where the crosswalk
starts in the drawing.

**Checkpoint**: run the simulation and watch. Signals should cycle, queues
should form and clear. If a queue never clears, check that lane's signal group.

---

## 9. Node for evaluation

A node tells Vissim which area to measure.

1. Right-click on empty space, **Add New Node**.
2. Draw a polygon around the whole intersection. Include the stop bars on all
   four approaches and a bit beyond. Double-click to close it.
3. In the node dialog, tick **Use for evaluation**.
4. Name it `PX131`.

The polygon must cross every approach link, otherwise movements are missed.

---

## 10. Evaluation settings

**Evaluation, Configuration**.

On the **Result Attributes** tab:
- tick **Node results**
- set **From time**: 900, **To time**: 4500, **Interval**: 3600

The first 900 s is warm-up, discarded, because the network starts empty and the
first few minutes are unrealistically fast.

On the **Direct Output** tab you can leave everything unticked.

---

## 11. Simulation parameters

**Simulation, Parameters**.

- **Period**: 4500 s
- **Random seed**: 42
- **Number of runs**: 10
- **Random seed increment**: 1
- **Simulation resolution**: 10 time steps per second

Ten runs with different seeds, because a single run of a microsimulation is one
sample, not an answer.

---

## 12. Run it

**Simulation, Continuous**, or press F5. Press the fast-forward button or set
simulation speed to maximum, otherwise it runs in real time and takes 75 minutes
per run.

Ten runs should take a few minutes.

---

## 13. Export the results

**Evaluation, Result Lists, Node Results**.

You will see rows per movement with delay and queue columns. Check the
**Movement** column shows things like `NB-Left`, and that all twelve appear.

Right-click the list, **Export to file**, or **File, Export, Node results**.
Save as `node_results_A.att` in the `vissim/` folder.

Then run the comparison:

```
python vissim.py compare vissim/node_results_A.att \
    output/timing-alternatives.json --alt A
```

---

## 14. The other three alternatives

**File, Save As**, `steeles_yonge_B.inpx`. Then open the signal controller and
replace the timings:

**Alternative B**, cycle 100:

| Signal group | Green start | Green end | Amber end |
|---|---|---|---|
| 1 NS left | 0.0 | 9.5 | 13.0 |
| 2 NS through | 14.5 | 45.3 | 48.8 |
| 3 EW left | 50.3 | 65.1 | 68.6 |
| 4 EW through | 70.1 | 95.0 | 98.5 |

**Alternative C**, cycle 130, same timings as A but the left turns run
protected-permitted. To model that, the left turn signal group needs a permitted
period as well: give signal group 1 a second green from 18.2 to 60.3 and add
**Conflict Areas** so left turners yield to opposing through traffic. Select the
area where the NB left connector crosses the SB through connector, right-click,
set the through movement as having priority. Repeat for each opposing pair.

**Alternative D**, cycle 120, protected-permitted:

| Signal group | Green start | Green end | Amber end |
|---|---|---|---|
| 1 NS left | 0.0 | 12.0 | 15.5 |
| 2 NS through | 17.0 | 55.3 | 58.8 |
| 3 EW left | 60.3 | 78.9 | 82.4 |
| 4 EW through | 83.9 | 115.0 | 118.5 |

Alternatives C and D are more work because of the conflict areas. Do A and B
first, get the whole workflow running end to end, then come back.

---

## Things that go wrong

**Vehicles vanish at the intersection.** A connector is missing, or a route ends
on a link that is not connected.

**Everything queues and never clears.** Usually a signal head assigned to the
wrong signal group, or a signal group whose green never comes on. Watch one
cycle in the animation and read the head colours.

**Delay looks impossibly low.** The warm-up was not excluded, or the node
polygon does not cross the approach links.

**Queues run off the end of a link.** Lengthen the approach link. A queue that
hits the end of the network is not being measured properly.

**The .att file has no movement column.** In the Node Results list, right-click
the header, choose the attribute selection, and add Movement, Vehicle delay and
Queue length before exporting.

---

## What to send back

The four `.att` files. The comparison script puts simulated delay and queue next
to the HCM estimate movement by movement, which is the table your write-up
needs.
