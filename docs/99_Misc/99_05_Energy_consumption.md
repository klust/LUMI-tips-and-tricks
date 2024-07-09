# Energy consumption on LUMI

-    System counters are in `/sys/cray/pm_counters` and more specifically,
     `/sys/cray/pm_counters/energy`.

-   [CUG 2018 paper on power management](https://cug.org/proceedings/cug2018_proceedings/includes/files/pap174s2-file1.pdf)

-   From a message of Orian in the chat:

    It is possible to compute it using the energy field using the sacct command with, for example, the following output format: `-oJobID,Elapsed,TRESUsageInTot%100`.

    I assume the values are in joule. The user can then get the carbon footprint by converting the joule in kWh (1 joule is 2.7778E-7 kWh) then eqCO2 (24 gCO2eq/kWh for hydroelectricity)

    But before answering to the user it would be good to confirm that the energy reported by Slurm is correct.

    **Answer from Emmanuel**: According to Fredrik, it seems correct for single-node computations but rather weird for multi-node ones.

    Measurements at the job step level should be used though, as measurements for the `batch` step are only for the first node.

    Divide by $3.6\cdot 10^6$ to get energy consumption in $\mathrm{kWh}$ as 
    $1 \mathrm{kWh} = 1000 \mathrm{J}/\mathrm{s} \times 3600 \mathrm{s} = 3600000 \mathrm{J} = 3.6\cdot 10^6 \mathrm{J}$.
