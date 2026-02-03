# 1.The CECI cluster

For this session, we will be working on a computer cluster. <br>
A cluster is a set of machines that allows an access to thousands of cores at the <br>
same time and a huge memory capacity. Clusters are extremely interesting to <br>
merge resources that would be otherwise inaccessible or too expensive. Cluster <br>
also allow a range of programs and environments preinstalled easily accessible <br>
to anyone. <br>
To access the cluster, users will go through a gateway and share the <br>
computational resources. That is why it is important to avoid running heavy jobs <br>
directly on the cluster interface as it will impact the other users. To manage and <br>
run the scripts, the cluster usually rely on a job scheduler (SLURM, QSUB etc.) <br>
with how many CPUs and memory it will need for a specific job. <br>
The cluster available is named CÉCI, for 'Consortium des Équipements de <br> 
Calcul Intensif'. CÉCI is thus a consortium which gathers all supercomputing <br>
centers from the universities in Fédération Wallonie-Bruxelles, namely <br>
Université libre de Bruxelles, Université catholique de Louvain, Université de <br>
Liège, Université de Mons, and Université de Namur <br>

![](https://i.imgur.com/2nl5zzP.png)

# 2.Connection to the cluster 

Check out all the possible clusters available to you through CECI at <br>
https://www.ceci-hpc.be/ and take time to navigate through the website. <br>
  - What is the cluster associated with UNamur?  <br>
  - What is the maximum RAM and CPU that you can access through this cluster? <br>
  - Can you explain what is RAM and CPU in your own words? <br>

To connect to the CECI cluster, you will need to use the ssh command <br>
that allow to connect to a remote machine. We are going to use a <br>
dedicated cluster for teaching: Hyades. To connect to Hyades you will <br>
need to know you unamur login (or your eid) and password. <br>
Use the command [ssh <your_eID>@hyades.ptci.unamur.be] and replace <your_eID> by your login. <br>
The first time you connect, the computer will ask you if you are sure you want to continue connecting. Type ‘yes’ in the terminal to continue.

In the cluster, you will have access to a personal space. <br>
> [!WARNING] <br>
> This space has a limited amount of memory of 100G. <br>

# 3. Downloading data
## Directly from WSL or Hyades

Whenever possible, it is preferable to download data directly where the computation will take place, instead of transferring large files from your personal computer.

From WSL or directly on Hyades:
```
wget https://example.org/data/SRR24965506.1
#This command download the file into your current directory
```
Check the download:
```
ls -lh
```

# 4. Transferring files using the command line
Working from the command line is the **recommended method** on a computing cluster, as it allows better control and reproducibility.

Before transferring files, make sure that:
- you know where your file is located on your local machine,
- you are connected to the correct directory.
- 
## Check your local directory
On your local machine (Linux, macOS, or WSL), move to the directory containing your file:
```
cd ~/Downloads
ls
```

## Uploading a file to Hyades
To upload a file to your home directory on Hyades:
```
scp SRR24965506.1.fastq.gz <user>@hyades.ptci.unamur.be:/home/<user>/
```
Replace `user` with your UNamur login.You will be prompted for your UNamur password (nothing will appear when typing — this is normal). If the command ends without error messages, the transfer was successful.

## Uploading a directory
If you want to transfer several files at once, place them in a directory and use:
```
scp -r data/ <user>@hyades.ptci.unamur.be:/home/<user>/
```

Finally, check the transfer.

## 5. Understanding SRA files and conversion to FASTQ
Files downloaded from NCBI SRA (e.g. SRR24965506.1) are not directly usable for analysis.
They must first be converted into FASTQ format.

Converting SRA to FASTQ (Using SRA-Toolkit)
```
fasterq-dump SRR24965506.1 --split-files
```

