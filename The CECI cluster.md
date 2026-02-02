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

![Screenshot of a comment on a GitHub issue showing an image, added in the Markdown, of an Octocat smiling and raising a tentacle.](https://i.imgur.com/2nl5zzP.png)]

# 2.Connection to the cluster 

Check out all the possible clusters available to you through CECI at
https://www.ceci-hpc.be/ and take time to navigate through the website.
What is the cluster associated with UNamur? What is the maximum RAM

To connect to the CECI cluster, you will need to use the ssh command
that allow to connect to a remote machine. We are going to use a
dedicated cluster for teaching: Hyades. To connect to Hyades you will
need to know you unamur login and password. 
Use the command [ssh <your_eID>@hyades.ptci.unamur.be] and replace <your_eID> by your
login. 
The first time you connect, the computer will ask you if you are
sure you want to continue connecting. Type ‘yes’ in the terminal to
continue.

In the cluster, you will have access to a personal space. This space has a
limited amount of memory of 100G.
To download and upload files to and from the CECI, we will see two
techniques. The first one is in command line and the second technique is
to use a file manager.
Example command line: scp ./file.txt hyades:~/
For convenience, you can install the file manager here:
https://winscp.net/eng/download.php
- Follow the instructions given in class to make a connection to your home
directory. 

and CPU that you can access through this cluster? Can you explain what
is RAM and CPU in your own words? 
