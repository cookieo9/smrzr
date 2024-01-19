smrzr - A tool for summarizing stdin / log files
=====

smrzr (read: summarizer) is a CLI tool that reads it's input from a file, or stdin and cuts down on the size of the output by finding repetitive lines.

It to be used as a pipeline, summarizing stdin to stdout:
``` bash
$ run-task | smrzr
```

Or as a file reader like `cat` to read log files and the like:
``` bash
$ smrzr run-task.log
```

General technique
-----

It's basically doing the following:
 - If a line is followed by N exact duplicates: write it out once, with a count of the duplicates
     ```
     line of text
     line of text
     line of text
     ----------- becomes ----------
     line of text (repeated 3 times)
     ```
     
  - If a line is followed by a line that shares either a prefix, a suffix, or both (under certain conditions) it will be represented by a single line where the the common parts are printed with a wildcard expression representing the varying part
    ```
    Read /usr/bin/cat for fun
    Read /usr/bin/ls for fun
    Read /usr/share/README.debian for fun
    Read /usr/share/doc for fun
    Read /usr/share/man for fun
    ----------- becomes -----------
    Read /usr/bin/*** for fun (2 items)
    Read /usr/share/*** for fun (3 items)
    ```
    - If a new line shares a longer prefix / suffix than we are already tracking consider the new prefix:
      - after N copies with the longer prefix / suffix (configurable, default 5)
      - if the "new portion" of the prefix/suffix contains at least one of `"(" "/" "\" " "`
    - A prefix / suffix set only is considered valid:
        - if it matches or extends the current prefix/suffix
        - if it matches at least 50% of the current and/or previous line
          - e.g. it's not a prefix/suffix:
            - if two consecutive lines happen to start with the same character / word
            - if two consecutive lines end with a period / semicolon 
