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

# Recording Changes to the Repository
Each file in your working directory can be in one of two states: **tracked** or **untracked**
## Tracked 
files are files that were in the last snapshot, as well as any newly staged files.
States : 
- unmodified
- modified
- staged

## Untracked
files are everything else

![[Pasted image 20220616140727.png]]


## Check status 
```console
git status
```
git status -s for short print 

```console
$ git status -s
 M README
MM Rakefile
A  lib/git.rb
M  lib/simplegit.rb
?? LICENSE.txt
```

staging area have an **A**
modified files have an **M**



### Tracking New Files
```console
git add file_name
```
**git add** is a multipurpose command — you use it to : 
 - begin tracking new files
 - stage files
 - do other things like marking merge-conflicted files as resolved


### Ignoring Files

The rules for the patterns you can put in the `.gitignore` file are as follows:

-   Blank lines or lines starting with `#` are ignored.    
-   Standard glob patterns work, and will be applied recursively throughout the entire working tree.    
-   You can start patterns with a forward slash (`/`) to avoid recursivity.    
-   You can end patterns with a forward slash (`/`) to specify a directory.    
-   You can negate a pattern by starting it with an exclamation point (`!`).

Glob patterns are like simplified regular expressions that shells use.
- An asterisk (`*`) matches zero or more characters;
- `[abc]` matches any character inside the brackets (in this case a, b, or c);
- a question mark (`?`) matches a single character;
- and brackets enclosing characters separated by a hyphen (`[0-9]`) matches any character between them (in this case 0 through 9)
- use two asterisks to match nested directories; `a/**/z` would match `a/z`, `a/b/z`, `a/b/c/z`, and so on.

It is also possible to have additional `.gitignore` files in subdirectories


### Viewing Your Staged and Unstaged Changes

`git status` answers those questions very generally by listing the file names,
`git diff` shows you the exact lines added and removed — the patch, as it were.


If you want to see what you’ve staged that will go into your next commit, you can use `git diff --staged`

### Committing Your Changes

```console
git commit -m "Story 182: fix benchmarks for speed"
```

```console
git commit -a -m 'Add new benchmarks'
```

Adding the `-a` option to the `git commit` command makes Git automatically stage every file that is already tracked before doing the commit, letting you skip the `git add` part


### Removing Files

```console
git rm PROJECTS.md
```

### Moving Files

```console
git mv file_from file_to
```








[stop here](https://git-scm.com/book/en/v2/Git-Basics-Viewing-the-Commit-History)

















#git