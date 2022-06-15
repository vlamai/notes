# Whats git 

## Snapshots, not difference 

These other systems (CVS, Subversion, Perforce, Bazaar, and so on) is **delta-based** version control 

Git is series of snapshot. Basically take a *picture* of all files and store reference to snapshot. 

Everything in Git is check-summed before it is stored and is then referred to by that checksum. The mechanism that Git uses for this check-summing is called a SHA-1 hash.


## The Three States
### modified 
change file but not **committed**
### staged
mark changed a modified in it's current state to go to **committed** state.

The staging area is a file, generally contained in your Git directory, that stores information about what will go into your next commit

### committed
data is stored 


![[Pasted image 20220615231933.png]]


# Getting Help
```console
git help <verb>
git <verb> --help

git add -h
```


[stop here](https://git-scm.com/book/en/v2/Git-Basics-Recording-Changes-to-the-Repository)

















#git