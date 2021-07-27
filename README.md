# First Sphere Analyzer

First Sphere Analyzer takes an input .xyz trajectory file and attempts to determine the radial distribution function (RDF) and subsequent coordination number. Using the coordination number and average ligand-ion distance (maximum of the RDF), the root-mean-square deviation (as implemented [here](https://github.com/charnley/rmsd)) is computed between the first coordination sphere atoms in each frame and the idealized geometries as specified by the coordination number and average ligand-ion distance.

## Usage
There are three files that come with the First Sphere Analyzer 
program:
1. ***run.py***
2. ***rmsd.py***
3. ***geometries.py***

The first, ***run.py***, is the one that requires user interaction. The 
following describes the usage of ***run.py*** line-by-line:
- line 3: the variable `outputFile` stores a string that is used 
                in the name of files that will be created by the program.
                There are four .csv files created by the program: 
                (1) radial distribution function (RDF), 
                (2) integral of radial distribution function (RDF-integral),
                (3) angle distribution function (ADF),
                (4) root mean square distances (RMSDs)

- line 5:   `Trajectory` type object is created and can be initialized 
                with certain properties used in the program. Some properties 
                include:
  - **ionID** (*required*):
    string of the ion name to be used as the central 
    atom in the RDF and RMSD calculations

    ex. `'La'` or `'Cr'`
  - **elements** (*required*):
    array of strings of elements that are included in 
    the RDF and RMSD calculations.
    
    ex. `['O']` or `['O', 'N']`
    
    For example, if the `ionID` is specified as `'La'` and 
    the elements are specified as `['O', 'N']`, then the program 
    will compute the RDF using all La-O and La-N distances in 
    the trajectory
   - **boxSize** (*optional*, default=10):
                        float of the size of the box of the trajectory. It is 
                        currently not being used
   - **framesForRMSD** (*optional*, default=100):
                        integer that specifies the number of frames between each 
                        redetermination of the optimal ideal coordinate permutation*
   - **startFrame** (*optional*, default=first frame):
                        integer that specifies on which frame to begin analysis
   - **endFrame** (*optional*, default=last frame):
                        integer that specifies the last frame to analyze. If no 
                        endFrame is specified, the program will analyze to the last
                        frame of the trajectory by default

      > \* This program calculates RMSDs using the Kabsch algorithm
        Two sets of coordinates, a real set and an ideal set, are 
        utilized in the Kabsch algorithm which is dependent on the 
        arrangement of the individual coordinates in the two sets.
        Thus, all permutations of coordinates in one of the sets 
        are calculated to find which permutation of that set produces 
        the minimum RMSD value. This is a brute force approach, and 
        takes a long time. Rather than do this step every single 
        frame of the trajectory, the program uses the optimal permutation 
        of the ideal set for as many frames as the user specifies. If 
        the user specifies 1, than the optimal permutation will be 
        calculated each frame, and there will be no error in RMSD values.
        However, if the user specifies 1000, then the program will 
        find and use the optimal permutation every 1000 frames. This 
        has an effect on the accuracy of RMSD values, but usually not 
        large
                
- line 6:    `getAtoms()` function takes a filename string as an input 
            and stores all atoms within a trajectory as an array

- line 7:    `getIonNum()` function finds the number of atoms within 
            the trajectory that are the name specified by the `ionID` variable
            
  ex. if there are two La atoms per frame of a trajectory, then 
  the program will prompt the user in the following lines to 
  select which ion they wish to use as the central atom in 
  following calculations

- lines 8-9: if there are more than one of the specified ion type (specified 
            by ionId), then the program will ask the user which 
            one to use by calling the getWhichIon function

- line 10:   `getRDF()` function calculates the RDF using the `ionID` and elements
            variables

- line 11:   `getDist()` function attempts to find the maximum of the RDF, this 
            value is used in the creation of the ideal coordination geometries 
            as the average value between central and coordinating atoms

- line 12:   `getMaxR()` function attempts to find the maximum radius that constitutes 
            the first coordination sphere. This is important because the program 
            uses determines if atoms lie in the first coordination sphere based 
            on this value

- line 13:   `getCN()` function attempts to find the coordination number using 
            the integral of the RDF

- line 14:   `printRDF()` function outputs the RDF and integral of RDF as .csv files
            the name for the files is determined by the user specified `outputFile`
            as found in line 3.
            
  ex. if the user specifies `outputFile` to be 'La-AIMD', then the files 
  will be named: 'La-AIMD-rdf.csv' and 'La-AIMD-rdf-integral.csv'

- line 15:   `checkWithUser()` function asks the user whether the max of the RDF, 
            threshold of first coordination sphere, and coordination number as 
            determined by the program's algorithm are correct or not. The user 
            can open the -rdf.csv and -rdf-integral.csv files in a graphing software 
            to visually check the program's results. If the values look correct, 
            the user can enter 'Y' tell the program to continue using its determined 
            values. If the values look incorrect, the user can enter 'N', at which 
            point the program will ask for the correct value.

- line 16:   `getThresholdAtoms()` function attempts to collect all atoms in the first 
            coordination sphere per frame in the trajectory. This step uses the 
            threshold distance (variable named `maxR`) to determine which atoms 
            lie in the first coordination sphere

- line 17:   `getADF()` function calculates the angle distribution function (ADF) for 
            the first coordination sphere of atoms in each frame. The ADF is the 
            set of all angles between unique ligand-ion-ligand pairs

- line 18:   `printADF()` function outputs the ADF just as the printRDF() function does.
            The name of the file is dependent on the `outputFile` variable.
            
  ex. if the user specifies `outputFile` to be 'La-AIMD', then the file 
  will be named: 'La-AIMD-adf.csv'

- line 19:   `getIdealGeos()` function utilizes the geometries.py file to generate 
            the ideal geometries for a given average ligand-ion distance and coordination
            number.
            
  ex. using the coordination number of 8 and an average ditance of 2.3 (specified
  either by the user or found by the program), ideal geometries such as square 
  antiprism, cubic, bicapped trigonal prismatic, etc. will be generated such that 
  the average distance between the central ion and all coordinating atoms will be 2.3

- line 20:   `getRMSDs()` function calculates the RMSD between the real (frame) and ideal sets 
            of coordinates. This function utilizes the Kabsch algorithm. The algorithm checks all permutations of 
            coordinates to find the minimum RMSD per frame. This can be slow, so the user
            can adjust the framesForRMSD in line 5 to speed up calculation time at the cost 
            of accuracy

- line 21:   `printRMSDs()` function prints the RMSD value per frame and per ideal geometry 
            in the same fashion as the `printRDF()` and `printADF()` functions.
            
  ex. if the user specifies `outputFile` to be 'La-AIMD', then the file 
  will be named: 'La-AIMD-RMSDs.csv'

- line 22:   `outputIdealGeometries()` function outputs the idealized geometries as .xyz files.
            The program will output one of each type of geometry depending on the coordination 
            number and average distance used.
            
  ex. if the program used a coordination number of 8 and an average distance of 2.3, 
  the outputted files will be square-antiprism.xyz, cubic.xyz, bicapped-trigonal-
  antiprism.xyz, etc. all with average distance 2.3
