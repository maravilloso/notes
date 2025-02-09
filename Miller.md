Miller
======

- [Official project site](https://github.com/johnkerl/miller)
- [Official documentation for v6.6](https://johnkerl.org/miller-docs-by-release/6.6.0/)

# Install
On CentOS
  
```bash
  $ wget https://github.com/johnkerl/miller/releases/download/v6.9.0/miller-6.9.0-linux-386.rpm
  $ sudo yum localinstall miller-6.9.0-linux-386.rpm
```

# Useful one-liners
Get a copy of a CSV file, but including a subset of the columns only. The input file is gzipped and uses custom `~` separator:
  
```bash
  mlr --prepipe zcat --csv --ifs \~ --fs \~ cut -f field1,field2 myarchive.uuid.gz > out.csv
```
