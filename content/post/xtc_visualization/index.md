---
title: Gromacs trajectory treatment for visualization
summary: How to visualize a xtc trajectory of a protein?
date: 2024-09-13
math: true
tags:
  - GROMACS
  - Visualization
image:
  caption: 'gmx trjconv'
  preview_only: true
commentable: true

draft: false
---
When visualizing simulation trajectories to check if a simulation ran smoothly, we often need to remove periodic boundary conditions (PBC) as well as eliminate translation and rotation of the protein. Sometimes, we also want to retain the solvent. In this post, I’ll share the commands to achieve these tasks.

## Step-by-Step: Visualizing a Trajectory
Because removing the PBC and correcting for translation and rotation can’t be done in a single command, we’ll need to run `gmx trjconv` twice to process the trajectory for visualization.

## Step 1: Removing PBC
Let’s start by removing the PBC. For a single-chain protein, the command below works well. If you have multiple chains, check out this tutorial [here](https://jerkwin.github.io/2016/05/31/GROMACS%E8%BD%A8%E8%BF%B9%E5%91%A8%E6%9C%9F%E6%80%A7%E8%BE%B9%E7%95%8C%E6%9D%A1%E4%BB%B6%E7%9A%84%E5%A4%84%E7%90%86/).
```bash
gmx trjconv -f demo.xtc -s demo.tpr -o example.xtc -ur compact -center -pbc mol
```
- `demo.xtc` is the raw trajectory file from your simulation. 
- `demo.tpr` is the .tpr file used for the simulation.

When prompted:
1. “Select group for centering”: Choose index 1 (Protein).
2. “Select group for output”: Choose index 0 (System) if you want to keep the solvent, or index 1 (Protein) if you’re only interested in the protein.

## Step 2: Preparing the Topology (Protein Only)
If you’re only interested in the protein (and not the solvent), you’ll need to generate a new topology file that only contains the protein. This ensures the trajectory and topology file match in terms of atom count, which is crucial for GROMACS analyses. Use the `gmx convert-tpr` command:
```bash
gmx convert-tpr -s demo.tpr -o demo_pro.tpr
```
Select the protein group, and you’ll get a .tpr file containing only the protein topology.

## Step 3: Removing Translation and Rotation
Now, to remove the translation and rotation, run the following command. I’ll use `demo.tpr` here because I want to keep the solvent. If you’re only interested in the protein, use the `demo_pro.tpr` file we generated in the previous step.
```bash
gmx trjconv -f example.xtc -s demo.tpr -o example_no_tran+rot.xtc -ur compact -fit rot+trans
```
When prompted:
1. “Select group for least squares fit”: Select index 1 (Protein) to remove translation and rotation of the protein.
2. “Select group for output”: Choose index 0 (System) to keep the solvent, or index 1 (Protein) if you only want the protein.

## Step 4: Visualizing the Trajectory in PyMOL
To visualize the processed trajectory in PyMOL, you’ll first need to load a compatible topology file. Since PyMOL can’t read .tpr files, you can either convert the .tpr to a .gro file using `gmx convert-tpr` or use the .gro file from your energy minimization step.

Once the topology file is loaded, you can load the trajectory file (.xtc) into PyMOL. If the trajectory is too large, you can reduce the number of frames loaded into memory by setting an interval.
