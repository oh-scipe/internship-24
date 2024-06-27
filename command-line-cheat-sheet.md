## Useful bash, emacs, and git:
| Command | Action |
| --- | --- |
| `<cmd> -h` |          print the help documentation for a command, showing usage information and options |
| `cd`	|		change directory |
|`cd ..` |		up one directory |
| `pwd` | 		current working directory |
| `ls` | 		list everything in current directory (use `-a` to also show **a**ll files including hidden, `-l` for a **l**ong list including permissions and ownership info, `-1` ("dash one") to display the output with **1** item on each line |
| `wc -l <file>` |      use the **w**ord **c**ount command with the `-l` **l**ines option to list the number of lines in a file |
| `du <dirname>/`|      calculate and show how much **d**isk **u**sage is consumed by a directory (use `-h` to make it **h**uman-readable, i.e. report in MB, GB or whatever units are most appropriate, and `-s` for **s**ummary of all the contents together rather than each item individually) |        
| `ctrl r` |		search for command (will pop up `bck-i-search:`) |
| `rm <target>` |       remove a file (or folder with `-r`). Beware when using `rm -rf <folder>` to **f**orce the **r**ecursive removal of all contents in a folder, which cannot be undone unless there is a backup. |
| `<cmd1> \| <cmd2>` |   The "pipe" operator (`\|`) feeds the output of the first command (`cmd1`) to the input of the second command (`cmd2`). For example, show the total number of files in a directory with `ls -1 <dir> \| wc -l`|

### Git-Specific
| Command | Action |
| --- | --- |
| `git log`   | 		list of commits with author, date, time (type `q` to leave) |
| `git log --oneline` | 		list of just commits (ID, location, message), type `q` to leave |
| `git status`  | 	status of local vs remote repo (commits, ignored files, etc), <br> shows changed files that git is tracking and that git is not tracking   |
| `git rm <target>`   | 		remove file (or folder with `-r`) from repo and filesystem (or just from the repo and not filesystem with `--cached`) <br>cache file ex: `git rm -r --cached __pycache__` |
| `git mv <file> <folder>`   | 			move file to folder <br>or rename: `git mv <filename> <new_filename>` |
| `git branch`   | 				list branches, current branch has `*` in front and is green |
| `git checkout -b <branch>`   | 		create new branch and check it out |
| `git checkout <branch>`   |			checkout branch |
| `git branch -d <branch>`   | 			delete branch |

**Usual Process:**
After making changes to a file, check the status of your current working branch (with `git status`). Then, you "add" the file, state what is new about the file ("commit the change"), and `push` the file from your local copy of the repo to the remote copy:

```bash
git add <filename>

git commit -m "Changed x,y,z"

git push

```

**Note:** If you need to update your branch with changes from `main`, first switch to the branch, then set pull from `main` instead of the current branch, as below.

```bash
git checkout <branch>		

git pull origin main
```

## More Advanced | Ohio Supercomputing Center
Here we list advanced commands based on their intended use. Please be sure you understand what they do, as they can be resource-intensive, and should likely be run on back-end/worker nodes rather than front-end/login nodes.

### Estimate queue time before a job will start.
Specify the [options](https://www.osc.edu/supercomputing/batch-processing-at-osc/slurm_migration/how_to_prepare_slurm_job_scripts) for the job you want to schedule, and use `srun --test-only` to receive an estimated start time based on current cluster state and queue.
Sample input:
```bash
srun --test-only --cluster=ascend --account=<your-account-number> --time=55:00:00 --nodes=2 --gpus-per-node=4 --ntasks-per-node=4 --cpus-per-task=16
```
Sample output:
```bash
srun: Job 85619 to start at YYYY-MM-DDTHH:mm:ss using 128 processors on nodes a[0018,0022] in partition gpu
```

This can also be run using a Slurm job script! So you can prepare your job and run `sbatch --test-only my_job.slurm`.

### Find open nodes.
During periods of high demand, you might have long waits before a job can start. If you are not constrained by your hardware selection, you can be flexible with what cluster to use. Replace the argument for `--cluster=` with `owens`, `pitzer`, or `ascend` (choices available in September, 2023).
```bash
sinfo --cluster=ascend -o "%t %N" | grep "idle"
```
Sample output:
```bash
idle a[0005-0006,0010]
```

### Copy a huge number of files from one location to another on OSC in parallel.
> **Note:** Back-end/worker nodes only

```bash
cd /fs/ess/<your-account-number>/dir/ && find . -type f -print0 | xargs -0 -P 10 -I {} cp -f --parents {} /fs/scratch/<your-account-number>/dir/
```
This example command uses 10 workers (`-P 10`) to copy the folder `dir/` from the research storage source location under `ess/` to the scratch storage destination location under `scratch/`, preserving the directory structure of the contents. You can check the progress as it runs by using a separate terminal to compare the number of files present in the source and destination locations with `ls -R1 <source-dir> | wc -l` and `ls -R1 <destination-dir> | wc -l`.
