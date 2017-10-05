### Server Pull Down

We are going to be using an ftp transfer to pull down entire studies on the metabolights servers. http://www.ebi.ac.uk/metabolights/

There may be another faster option that does some fancy compression and optimizes bandwidth that may be fun to check out (I can't remember the name right now)

```unix
~$ sudo lftp 
lftp> open ftp.ebi.ac.uk
# it may prompt for username and password
Name (ftp.ebi.ac.uk:luke): anonymous
331 Please specify the password.
Password: <leave blank>
```

Gets you into the ftp server. From which you can look around the studies with corresponding MTBL ID's.

Lets look at MTBLS36: "Metabolic differences in ripening of Solanum lycopersicum 'Alisa Craig' and three monogenic mutants"

```unix
cd /pub/databases/metabolights/studies/public/
ls .
```

Look around! The public directory has all the available studies.

The tricky thing about ftp is that we need to have matching files on our local machine to tranfser to. With lftp we use the mirror command to populate a directory so we can easily transfer files to the correct locations. It would look simply like this: but lets give it a few more options.

```unix
mirror MTBLS36
```

To pull a single sample, and then the whole directory using segmentation (which helps speed the transfer, the only thing faster I think would be peer to peer, which I should set up as a seed once I get the data transfered):

To transfer one file using 10 segments use something like:

```unix
pget -n 10 MTBLS36/20100917_01_TomQC.mzML
```
To transfer the entire directory of files and mirror it to your local machine, but only if a file has an mzML extension we can use something like:

```unix
mirror MTBLS36 -P 10 -I *mzML
```
This will take a while, so minimize the terminal and let it run. Alternitively you can set it to run in the background by adding it to a queue, see man page of lftp. Note: If you are restarting the ftp be sure to add the -c option to the mirror call.

```unix
queue mirror MTBLS36 -P 10 -I *mzML
jobs
```
Now you should be able to type exit and leave the lftp processing running. Use something like htop or take a peek at the directory to make sure everything is running well. You can attach a process to a terminal, not sure if you would have to do that if the terminal session running it was from a remote connection or not.... hmmmmm

I removed the segmentation code because it seemed to be more difficult for me to get to run properly. 

Because this file is already in .mzML format, we don't need to convert it. But we do need to copy the hrms.R script and the lipidlist.csv file into our directory in order to run the hrms code. Be sure you are in your own directory where you downloaded the mass spectral file and run the following.
 
```shell
cp /home/luke/github/hrms/hrms.R ./ # copy the files we need from the github repository (replace the path with your own)
cp /home/luke/github/hrms/lipidlist.csv ./ 
chmod +x hrms.R # may need to make the hrms.R file executable
./hrms.R 20100917_01_TomQC.mzML
```

Take a look at the resulting list of peak heights
```unix
head 20100917_01_TomQC.csv
```

I haven't scripted pulling down the entire study yet, and would appreciate that a bunch. There is a way to analyse an entire study with the HRMS code that uses the threading module in python which is already writen by me and is in the HRMS repository with the file hrms_multithread.py. Also, there is a python library to do this whole FTP transfer via a python interpreter. The problem with running all of this in python is that it may occasionally leave threads open, so in the long run I will need to write an admin script to check for hanging threads.

An example of what the process would look like when running HRMS on all .mzML files in a directory using 4 cores:
```unix
cp /home/luke/github/hrms/hrms_multithread.py ./
python hrms_multithread.py .mzML 4 # to use 4 cores
```

Then take a look at the signals.csv and deviations.csv files.

```unix
head signals.csv
head deviations.csv
```

The output is a list of signals or deviations from the mass target used for each sample and each metabolite.

