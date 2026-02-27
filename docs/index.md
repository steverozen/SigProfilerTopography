<h1> SigProfilerTopography </h1>

----------

SigProfilerTopography is a python framework that analyses the distribution of somatic mutations and mutational signatures on the whole genome with respect to cellular processes such as DNA replication and transcription. 

SigProfilerTopography also seamlessly integrates with other SigProfiler tools and leverages them for parts of its computational workflow, including classification of somatic mutations using SigProfilerMatrixGenerator, simulating realistic background mutations with SigProfilerSimulator, and assigning mutational signatures to each somatic mutation using SigProfilerAssignment.

Topography analyses are supported for the following mutations:

- Single Base Substitutions (SBSs)
- Doublet Base Substitutions (DBSs)
- Small Insertions and deletions, indels (IDs)

and carries out the following analyses:

- Nucleosome Occupancy
- Histone Occupancy
- Transcription Factor Occupancy
- Replication Timing
- Replicative Strand Asymmetry
- Transcription Strand Asymmetry
- Genic versus Intergenic Regions
- Strand-coordinated Mutagenesis

## Citation ##
Otlu B, Alexandrov LB: Evaluating topography of mutational signatures with SigProfilerTopography. Genome Biology 2025, https://doi.org/10.1186/s13059-025-03612-8.

Otlu B, Diaz-Gay M, Vermes I, Bergstrom EN, Zhivagui M, Barnes M, Alexandrov LB: Topography of mutational signatures in human cancer. Cell Rep 2023, https://doi.org/10.1016/j.celrep.2023.112930.

## License ## 

This software and its documentation are copyright 2019-2023 as a part of the SigProfiler project. The SigProfilerTopography framework is free software and is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See the GNU General Public License for more details. 

## Contact ##

Please address any queries or bug reports to Burcak Otlu at burcako@metu.edu.tr