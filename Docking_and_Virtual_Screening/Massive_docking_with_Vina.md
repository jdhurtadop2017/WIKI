# Welcome to the Ramirez Lab Wiki - Docking and Virtual Screening

Here we use AutoDock Vina to perform massive docking of one ligand: 100 runs -> top-10 poses per run -> 1000 poses. The idea is to use a single ligand and do multiple dockings on the same receptor to extend the conformational sampling of that ligand. Then you can use a tool to cluster the conformers and know which are the most visited poses during the molecular docking.

You will need: [Vina](http://vina.scripps.edu/download.html), [Vina_split](https://github.com/ramirezlab/WIKI/tree/master/Docking%20and%20Virtual%20Screening/Files) & [Openbabel](http://openbabel.org/wiki/Main_Page) installed in your computer. Also the _input.conf_ file with the configuration, and the script _loop_vina.sh_.


#### Example of _input.conf_
```json
receptor = /path/to/receptor/receptor.pdbqt
ligand = /path/to/ligand/ligand.pdbqt

center_x = -14.33
center_y = -41.71
center_z = -25.95
size_x =  30
size_y =  30
size_z =  30

cpu = 8
num_modes = 10
```



#### loop_vina.sh to create 1000 poses of one ligand and one receptor

```bash
#####################################################################
############# Created by Jessica Martinez - Ramirez Lab #############
#####################################################################

##Carry out the first Docking and convert the outputs from .pdbqt to .pdb
for i in {1..1}
do
	mkdir output
	mkdir pdbqt
	mkdir ./pdbqt/splitpdbqt1/
	mkdir ./pdbqt/merge/
	/path/to/vina --config input.conf --out ./pdbqt/merge/ligand$i.pdbqt > ./output/output$i.log
	/path/to/vina_split --input ./pdbqt/merge/ligand1.pdbqt --ligand ./pdbqt/splitpdbqt1/ligand
	for e in {1..9}
	do
		/path/to/babel -d ./pdbqt/splitpdbqt1/ligand0$e.pdbqt -r ./pdbqt/splitpdbqt1/ligando0$e.pdb
	done
		/path/to/babel -d ./pdbqt/splitpdbqt1/ligand10.pdbqt -r ./pdbqt/splitpdbqt1/ligando10.pdb
done

#Carry out 4 more Docking taking random the 10 first structures and convert all of them from .pdbqt to .pdb

for a in {2..5}
do
	mkdir ./pdbqt/splitpdbqt$a
	/path/to/vina --config input.conf --ligand ./pdbqt/splitpdbqt1/ligand0$((RANDOM % (10 - 1 + 1 ) + 1 )).pdbqt --out ./pdbqt/merge/ligand$a.pdbqt > ./output/output$a.log
	/path/to/vina_split --input ./pdbqt/merge/ligand$a.pdbqt --ligand ./pdbqt/splitpdbqt$a/ligand
       for e in {1..9}
       do      
               /path/to/babel -d ./pdbqt/splitpdbqt$a/ligand0$e.pdbqt -r ./pdbqt/splitpdbqt$a/ligando0$e.pdb
       done
               /path/to/babel -d ./pdbqt/splitpdbqt$a/ligand10.pdbqt -r ./pdbqt/splitpdbqt$a/ligando10.pdb
done

#Carry out 45 more Docking taking random the 50 previous structures and convert all of them from .pdbqt to .pdb

for t in {6..50}
do
    mkdir ./pdbqt/splitpdbqt$t
	/path/to/vina --config input.conf --ligand ./pdbqt/splitpdbqt$(( RANDOM % (5 - 1 + 1 ) + 1 ))/ligand0$(( RANDOM % (9 - 1 + 1 ) + 1 )).pdbqt --out ./pdbqt/merge/ligand$t.pdbqt > ./output/output$t.log
	/path/to/vina_split --input ./pdbqt/merge/ligand$t.pdbqt --ligand ./pdbqt/splitpdbqt$t/ligand
       for y in {1..9}
       do
              /path/to/babel -d ./pdbqt/splitpdbqt$t/ligand0$y.pdbqt -r ./pdbqt/splitpdbqt$t/ligando0$y.pdb
       done
              /path/to/babel -d ./pdbqt/splitpdbqt$t/ligand10.pdbqt -r ./pdbqt/splitpdbqt$t/ligando10.pdb
done

#Carry out 50 more Docking taking random the 500 previous structures and convert all of them from .pdbqt to .pdb

for u in {51..100}
do
        mkdir ./pdbqt/splitpdbqt$u
       /path/to/vina --config input.conf --ligand ./pdbqt/splitpdbqt$(( RANDOM % (50 - 1 + 1 ) + 1 ))/ligand0$(( RANDOM % (9 - 1 + 1 ) + 1 )).pdbqt --out ./pdbqt/merge/ligand$u.pdbqt > ./output/output$u.log
       /path/to/vina_split --input ./pdbqt/merge/ligand$u.pdbqt --ligand ./pdbqt/splitpdbqt$u/ligand
       for w in {1..9}
       do
              /path/to/babel -d ./pdbqt/splitpdbqt$u/ligand0$w.pdbqt -r ./pdbqt/splitpdbqt$u/ligando0$w.pdb
       done
              /path/to/babel -d ./pdbqt/splitpdbqt$u/ligand10.pdbqt -r ./pdbqt/splitpdbqt$u/ligando10.pdb
done


```

## Build Ligand-Receptor complexes after Massive Docking

```bash
#####################################################################
############# Created by @davidRFB  #############
#####################################################################

cp receptor.pdbqt pdbqt/
cd pdbqt/
    obabel -ipdbqt receptor.pdbqt -opdb -O receptor.pdb
    count=1
    for i in {1..100};
    do
    cd splitpdbqt$i
        for j in {01..10}; do
            echo "$i $j"
            obabel -ipdbqt ligand"$j".pdbqt -opdb -O ligand"$j".pdb ---errorlevel 0
            #Generate a complex file with all poses and the receptor
            echo "MODEL $count" >> ../Complexes.pdb
            grep "REMARK\|HETA" ligand"$j".pdb >> ../Complexes.pdb
            grep "ATOM" ../receptor.pdb >> ../Complexes.pdb
            echo "ENDMDL" >> ../Complexes.pdb
            #Generate a Poses file with all poses in pdb format
            echo "MODEL $count" >> ../Poses.pdb
            grep "REMARK\|HETA" ligand"$j".pdb >> ../Poses.pdb
            echo "ENDMDL" >> ../Poses.pdb
            ((count++))
            done
        cd ..
    done
#Generate clustering anaylsis folder.
bitclust -top splitpdbqt1/ligand1.pdb -traj Complexes.pdb -odir Analysis -cutoff 3 -sel "resname UNK"
```

## Citing

* Jessica Martínez Bernal, Cristian Buendía-Atencio & David Ramírez. (2020, October 15). Massive docking with vina (Version 1.0). Zenodo. http://doi.org/10.5281/zenodo.4089225

* [![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.4089225.svg)](https://doi.org/10.5281/zenodo.4089225)


