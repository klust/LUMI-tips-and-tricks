# Energy consumption on LUMI

## General information

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


## PMT - Power Measurement Toolkit

From the paper "[Stefano Corda, Bram Veenboer, and Emma Trolley. PMT: Power Measurement Toolkit. 2022IEEE/ACM International Workshop on HPC User Support Tools (HUST)](https://ieeexplore.ieee.org/document/10027520) 
[DOI: 10.1109/HUST56722.2022.00011](https://doi.org/10.1109/HUST56722.2022.00011)"
([Preprint arXiv:2210.03724](https://arxiv.org/abs/2210.03724))
([alternative link to the paper in the IEEE Computer Society Digital Library](https://www.computer.org/csdl/proceedings-article/hust/2022/634900a044/1KnWyMBKdmo)).

BibTeX reference:

```
@INPROCEEDINGS{10027520,
  author={Corda, Stefano and Veenboer, Bram and Tolley, Emma},
  booktitle={2022 IEEE/ACM International Workshop on HPC User Support Tools (HUST)},
  title={PMT: Power Measurement Toolkit},
  year={2022},
  volume={},
  number={},
  pages={44-47},
  doi={10.1109/HUST56722.2022.00011}
}
```

[GitLab repository of the code @ ASTRON](https://git.astron.nl/RD/pmt) with
[release tags](https://git.astron.nl/RD/pmt/-/tags)

Installing (instructions from Orian)

```bash
module load PrgEnv-gnu rocm
git clone --recursive https://git.astron.nl/RD/pmt.git
mkdir pmt/build
cd pmt/build
sed -i 's/Cray::name/name/' ../cray/Cray.h
cmake -DPMT_BUILD_CRAY=ON \
    -DPMT_BUILD_ROCM=ON \
    -DCMAKE_C_COMPILER=gcc \
    -DCMAKE_CXX_COMPILER=g++ \
    -DCMAKE_INSTALL_PREFIX=<changeme> ..
make install
```


