# Some background on Unix:
unix = operating system which users can connect to remotely via terminals
text is used as input & output. text is very light on resources compared to graphics which allows unix to be used quickly/efficiently even when multiple terminals are running. commands are also kept minimalistic = speeds up use even more.

## 1. Accessing the terminal

for window users - have mobaxterm downloaded
[https://mobaxterm.mobatek.net](https://mobaxterm.mobatek.net/download-home-edition.html)

for mac users - open the terminal from launchpad/other or finder

You can keep two open at a time: 
One terminal open to your computers hardrive = use this for file transfers. This will help when copying files by seeing pathways in a different terminal
The other = Logged into the [compute canada cluster](https://ccdb.alliancecan.ca/security/login) or [Centre for advanced computing](https://cac.queensu.ca/) or any other external computing resource you are using.

## 2. Logging into compute canada clusters
At Queen's, we have access to 3 different clusters = Beluga, Cedar, Graham
cluster = a group of interconnected compute nodes (comp. unit) managed by a resource 
	manager acting like a single system. Basically it houses a bunch of nodes & nodes
	are what get allocated for you to run a job = bigger job = more nodes 
[compute canada's technical glossary](https://www.computecanada.ca/research-portal/accessing-resources/glossary/)

let's log in:
```command
ssh -Y user@cluster.computecanada.ca
#or
ssh -X user@login.cac.queensu.ca
```
ssh = secure shell, securely connects to a remote server or system 
This will ask for a password - It will remain hidden while typing, just type it in and click enter and it should work


## 3. Resources available/used for each cluster
```command
quota #displays users' disk usage & limits 
free -m #gives overview of how much memory have, how much in use, how much available
```


## 4. Basic command line - navigating files/directories:

`pwd` Will display the present working directory

`ls` lists the contents of the working directory. Gives the names of all the file (in black), directories (in blue), and zipped files (in red). 
Files will also end in .something (e.g. `.txt`) while directories will not.
You can look inside a directory using the `ls directoryname` command 

`ls -F` will further differentiate files and directories by including a forward slash after `directories/`  
`ls -R` lists the contents of directories recursively  
`ls -t` lists things by time of last change, with most recent first  

## change working directory
'cd <directory name>' move into a directory  
`cd ..` move back one directory level  
`cd ../..` move back two levels  
`cd  ` will move back to the home directory  
`cd ~` will also move back to the home directory  
you can list a path to streamline the process   
`cd outer_directory/inner_directory/more_inner_directory`  

## making directories
`mkdir <diretory name>` to make a directory 
good practices for directory names = no white space, use letters/#s/_/-/.

## File transfer from computer to server 
`scp <source file path & name> <where you want to go>` scp = secure copy paste
scp /Users/computername/Desktop/file.csv sa123456@login.cac.queensu.ca:~/directory_x/
it will ask for your password to allow the transfer 
will say 100% when finished loading 
this works both ways - can also transfer files from a server to your computer









