# Vissim model inputs

Site: Steeles Ave E / Yonge St / Steeles Ave W
Count: 116429, 2026-06-30, signal PX 131
Period: PM peak hour starting 16:45

## Files

`vehicle_inputs.csv` — peak hour volume for each of the twelve movements, with
the turning fraction within each approach. Enter the approach totals as vehicle
inputs and the fractions as static routing decisions.

`pedestrian_inputs.csv` — pedestrian crossings per hour on each leg.

`controller_A.csv` … `controller_D.csv` — fixed-time signal controller for each
timing alternative. Splits sum to the cycle. Effective green is the split minus
4 s of lost time, which is what the capacity calculation uses; the green
interval is what you enter in Vissim.

## Building it with a script instead

`build_vissim.py` at the repository root builds the whole network through the
Vissim COM API: links, connectors, vehicle inputs, routes, signal controller,
signal heads, evaluation node and simulation settings. Run it on the machine
where Vissim is licensed.

    pip install pywin32
    python build_vissim.py --alt A --visible
    python build_vissim.py --alt A --run --export

Geometry is generated from coordinates, so the intersection is orthogonal rather
than carrying the real skew of Steeles against Yonge. Lane counts, volumes and
timings are the real ones. COM attribute names change between Vissim versions,
so expect one or two errors on the first run; the message names the attribute.

## Building the model by hand

1. Geometry from the DXF in `output/`, imported as a background, or drawn from
   the same lane configuration in `config/`.
2. Vehicle inputs and routing from `vehicle_inputs.csv`.
3. Heavy vehicle share by movement is in the count data; the PM peak is about
   3 percent overall on this corridor.
4. Signal control: fixed time, one controller per alternative.
5. Evaluation: node evaluation with delay and queue length, ten runs with
   different random seeds, 900 s warm up discarded.

## Comparing back

Export the node evaluation as an .att file and run:

    python vissim.py compare vissim/node_results.att \
        output/timing-alternatives.json --alt A

Expect differences. Microsimulation captures blocking, lane changing and
platoon arrivals that the HCM equations do not, so simulated delay usually
exceeds the analytical value where a movement is near capacity.
