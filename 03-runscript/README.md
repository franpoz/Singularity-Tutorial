# The Runscript: Making containerized apps behave more like normal apps

We are now going to consider an extended example describing a containerized application that takes a file as input, analyzes the data in the file, and produces another file as output.
This is obviously a very common situation.

Let's imagine that we want to use the cowsay program in our `lolcow.simg` to "analyze data".  We should give our container an input file, it should reformat it (in the form of a cow speaking), and it should dump the output into another file.  

Here's an example.  First I'll make some "data"

```
$ echo "The grass is always greener over the septic tank" > input
```

Now I'll "analyze" the "data"

```
$ cat input | singularity exec lolcow.sif cowsay > output
```

The "analyzed data" is saved in a file called `output`. 

```
$ cat output
 ______________________________________
/ The grass is always greener over the \
\ septic tank                          /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```

This _works..._ but the syntax is ugly and difficult to remember.  

Singularity supports a neat trick for making a container function as though it were an executable.  We need to create a **runscript** inside the container. It turns out that our Singularity recipe file already contains a runscript.  It causes our container to print a helpful message.  

```
$ ./lolcow.sif
This is what happens when you run the container...
```

Let's rewrite this runscript in the definition file and rebuild our container
so that it does something more useful.  

```
Bootstrap: library
From: library/default/debian

%runscript
    infile=
    outfile=

    usage() {
        >&2 echo "Usage:"
        >&2 echo "$SINGULARITY_NAME -i <infile> -o <outfile> [ -- <cowsay options> ]"
        exit 1
    }

    while getopts i:o: argument
    do
        case $argument in
        i)
            infile="$OPTARG"
            ;;
        o)
            outfile="$OPTARG"
            ;;
        ?)
            usage
            ;;
        esac
    done

    shift "$((OPTIND - 1))"

    if [ -z "$infile" ] || [ -z "$outfile" ]
    then
        usage
    fi

    cat "$infile" | cowsay "$@" > "$outfile"

%post
    echo "Hello from inside the container"
    apt-get update
    apt-get -y install fortune cowsay lolcat
    apt-get clean

%environment
    export PATH=$PATH:/usr/games
    export LC_ALL=C
```

Now we must rebuild out container to install the new runscript.  

```
$ sudo singularity build --force lolcow.sif Singularity
```

Note the `--force` option which ensures our previous container is completely overwritten.

After rebuilding our container, we can call the lolcow.simg as though it were an executable, give it input and output file names, and optionally give additional arguments to go directly to the `cowsay` program.  

```
$ ./lolcow.sif
Usage:
lolcow.sif -i <infile> -o <outfile> [ -- <cowsay options> ]

$ ./lolcow.sif -i input -o output2

$ cat output2
 ______________________________________
/ The grass is always greener over the \
\ septic tank                          /
 --------------------------------------
        \   ^__^
         \  (oo)\_______
            (__)\       )\/\
                ||----w |
                ||     ||
```


Now you can ship your image to lemaitre3
```
$ scp ./lolcow.sif lemaitre3:~
```
This might not work from the virtual machine (it does not know your keys).
you can copy it to your local host file system via
```
$ cp ./lolcow.sif /vagrant
```
and then retry from your physical os.

Once the image is on lemaitre3, try to run it locally and/or via srun.
