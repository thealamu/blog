# Automatically sourcing virtual environments
Python virtual environments are a mechanism for separating project dependencies.
> A virtual environment is a Python environment such that the Python interpreter, libraries and scripts installed into it are isolated from those installed in other virtual environments, and (by default) any libraries installed in a “system” Python, i.e., one which is installed as part of your operating system. — Python Docs

When working with a virtual environment, a key step before installing or using project dependencies is *sourcing* the environment.
This is usually achieved by asking an interpreter (for example, *bash*) to process an *activate* script, found in the ```bin``` subdirectory of the environment.
This can get tedious when you work on multiple python projects, each with its own virtual environment. There is also a tendency to forget to source a project's environment (Uh oh, why isn't this working? It worked an hour ago).

[venv_cd](https://github.com/thealamu/venv_cd) is a script that automatically sources a python virtual environment (if any) present in a directory you ```cd``` into. So, how does it work?

## Hooking into ```cd```
Effectively, ```venv_cd``` replaces the builtin ```cd```. This requires that it presents itself to the interpreter as ```cd```.
Interpreters like bash have a command discovery/search mechanism which allows them locate commands or functions you execute. For bash, this is effectively (ommiting a couple): 
> ```aliases``` ```functions``` ```builtins``` ```path```

in that order. ```cd``` is a builtin, which means providing bash with a function of the same name allows us get called before the shell builtin command. Even better, ```venv_cd``` aliases cd to a function. Here's what that looks like:
```shell
_cd() {
    builtin cd $@
    ...do custom stuff...
}
alias cd=_cd
```
```builtin cd $@``` makes a call to the builtin ```cd``` command passing all arguments, to make sure bash changes directories first before executing our custom commands.

As a side note, the ```$@``` was used instead of ```$*``` because ```$@``` presents the arguments as individual strings as such: "a1" "a2" "a3", where ```$*``` presents arguments as a single string: "a1 a2 a3".

## Checking for a virtual environment
Changing directories was just the first step, the next would be to check that the directory we just changed to has a virtual environement. There are multiple ways to create a virtual environment for python, one thing common to them though is the presence of the *activate* and *deactivate* scripts.
A simple but quick enough implementation used in ```venv_cd``` was to search the top level directories for a ```bin/activate``` file. If this file exists, we instruct the interpreter to source the file.

## Bringing it all together
The final script works something like this:
1. Hook into ```cd```
2. Do actual directory change
3. Walk the top level directories
4. For each top level directory, check for the presence of a ```bin/activate``` script
5. If found, *source* the file

```shell
_cd() {
	builtin cd $@	# do actual directory change
	for dir in */	# walk top-level dirs
	do
		dir=${dir%*/} # remove trailing '/'
		if [ -f "$dir/bin/activate" ]
		then
			# source the virtual environment
			echo "Environment found in '$dir', sourcing..."
			. $dir/bin/activate
		fi
	done
}

alias cd=_cd
```
