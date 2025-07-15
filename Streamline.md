Sync Ubuntu to Github -Streamline codes Codes.txt


#check what it thinks your remote is: 
git remote -v

you'll see 
origin git@github.com:Dygasy/Metagenomics_V2.git (fetch)
origin git@github.com:Dygasy/Metagenomics_v2.git (push)


###If you SSH key is fine on disk, but your SSH agent isnt running
1. start the SSH agent
```bash
eval "$(ssh-agent -s)"
```
###This will start your SSH authentication agent and print something like:
```bash
Agent pid 12345
```

2. Add your SSH private key
```bash
ssh-add ~/test1
```
###It should return like the following
```bash
Identity added: home/bulat/test1

###Test your connection to Github

```bash
ssh -T git@github.com
```

###If its successful, "Hi Dygasy! You've successfully authenticated, but Github does not provide shell access"

Create folders like scripts/, data/, figures/, docs/ on Github using local Git (on your desktop)

```bash
cd (repo of your choice)

### create directories
mkdir -p scripts data/GTDB_relative_abundance data/ metadata figures/ barplots figures/heatmap figures/nmds figures/maaslin docs

### Make sure Git tracks empty directories
touch scripts/.gitkeep data/GTDB_relative_abundance/.gitkeep figures/barplots/.gitkeep docs/.gitkeep

### Add & commit
git add
git commit -m "Organise repo structure: add scripts, data, figures, doc folders"
git push



