This uses the `traitar`software available from [here](https://github.com/hzi-bifo/traitar/releases)

Download the zipped file and install it on your server:
```
pip install traitar-1.1.1.tar.gz --user
```

Make sure to have `prodigal`, `hmmer` and `parallel` loaded.  Make a tab separated sample (`sample_list.tsv`) file that looks like this:
```
sample_file_name        sample_name     category
BetIa_bin-contigs.fa    BetIa_bin       environment_1
bacIa_vizbin2-contigs.fa        bacIa_vizbin2   environment_2
```

Then run `traitar` with `-c` being the number of processors:
```
traitar phenotype path/to/genomefastas sample_list.tsv from_nucleotides output -c 20
```

Example output:
![image](https://cloud.githubusercontent.com/assets/19682548/25242859/1b0f59a4-25c9-11e7-9a50-18898b60c0bb.png)
