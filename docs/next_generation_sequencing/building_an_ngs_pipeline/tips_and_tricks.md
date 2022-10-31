---
sidebar_position: 4
---

# Tips and tricks

Here is some guidance to help you write your pipeline. Click the links to jump to the relevant
section.

* [Wait, what?  How should I start?](./#getting-started)
* [How should I run snakemake?](./#how-should-i-run-snakemake)
* [How should I get sample information in?](./#how-should-i-get-sample-information-in)
* [Give me a first rule hint?](#give-me-a-first-rule-hint)
* [How should I organise my pipeline files?](#how-should-I-organise-my-pipeline-files)
* [My snakefiles are getting too big!](#my-snakefiles-are-getting-too-big)
* [Keeping a fast iteration time during development](#keeping-a-fast-iteration-time-during-development).
* [Dealing with intermediate files](#dealing-with-intermediate-files).
* [But I want to run the yellow bits too!](#but-I-want-to-run-the-yellow-bits-too)
* [Read groups what now?](#read-groups-what-now)
* [What's in the fastq header?](#whats-in-the-fastq-header)
* [Octopus is taking too long!](#octopus-is-taking-too-long)
* [What ploidy?](#what-ploidy)
* [Tools that use temporary directories](#tools-that-use-temporary-directories)
* [Tips on using `bwa mem`](#tips-on-using-bwa-mem).
* [Tips on using `samtools`](#tips-on-using-samtools).
* Or see a [complete solution](#solutions).

### Getting started

You should create: a folder called `data` with the data in, a folder called `results` to put
results in, and a folder called `pipelines` to put snakemake pipelines in. So your folder will look
something like this:

    this_folder/
      data/
        # reads and reference sequence here
      pipelines/
        # snakefiles go here
      results/
        # results files go here
        ...
  
	
Of course you should already have `data` and snakemake will create the `results` folder as you go.
So to get started, all you have to do is write a snakefile in `pipelines/`.

You will also need a **config file** as outlined [below](#how-should-i-put-sample-information-in) -
it can go in `pipelines` or in the top-level folder, whichever you prefer.

[Go back to the tips and tricks](#tips-and-tricks).

### How should I run snakemake?

Let's say your snakefile is `pipelines/analysis.snakefile`. The best way (I find) to run snakemake
is to *always run it from the top-level folder.* That is, you would run:
```sh
snakemake -s pipelines/analysis.snakefile -configfile config.json [other options...]
```

Doing this means that in your snakefile you can use relative pathnames. So for example if the input
file is one of the data files, you can write:
```python
rule something
	input:
		"data/reads/{ID}.fastq.gz"`
	output:
		"results/qc/{ID}.aligned.bam"
	(etc.)
```

and so on. This is great because you don't need absolute paths; it makes the snakefiles shorter and
it makes it is easy to copy the code around, or to share the pipeline via github, and so on.

**Note.** The snakemake documentation suggests a [similar, but slightly different
layout](https://snakemake.readthedocs.io/en/stable/snakefiles/deployment.html).

[Go back to the tips and tricks](#tips-and-tricks).

### How should I get sample information in?

Your pipeline is going to need to read in sample information, but you'd probably also like it to be
applicable to different samples sets. To accomodate this it's a good idea to put the information
about the samples / the needed data in through a **config file**.

This is a file called (say) `config.json` that you pass in using the `--configfile` argument. For
example, for this project you could use a `config.json` that looks like this:

```json config.json
{
	"reference": "data/reference/Pf3D7_v3.fa.gz",
	"fastq_filename_template": "data/reads/subsampled/{ID}_{read}.fastq.gz",
	"samples": [
		{ "name": "QG0033-C", "ID": "ERR377582" },
		{ "name": "QG0041-C", "ID": "ERR377591" },
		{ "name": "QG0049-C", "ID": "ERR417627" },
		{ "name": "QG0056-C", "ID": "ERR417621" },
		{ "name": "QG0088-C", "ID": "ERR377629" }
	]
}
```

(This contains information copied from `samples.tsv`.) And then you would run snakemake like this:

```
snakemake -s pipelines/master.snakefile --configfile config.json
```

The point of this is that it makes it easy to run the pipeline on different sets of data - such as
[a test data for pipeline testing](#keeping-a-fast-iteration-time-during-development) - you just
swap out the config file for a different one.

In a real pipeline there are likely to be many samples, so it'd be better to reference the sample
sheet in the config file:

```json config.json
{
	"reference": "data/reference/Pf3D7_v3.fa.gz",
	"fastq_filename_template": "data/reads/subsampled/{ID}_{read}.fastq.gz",
	"samples": "samples.tsv"
}
```

and then have your snakefile load the samples using (for example)
[pandas](/prerequisites/pandas.md):

```
import pandas
config['samples'] = pandas.read_table( "samples.tsv" )
```

[Go back to the tips and tricks](#tips-and-tricks).

### Give me a first rule hint?

There are two sensible things you could do at the start of the pipeline.

One is just to dive in and run `fastqc` (after all that's the first step outlined above.) That's
pretty easy, right?  Something like:

```
rule run_fastqc
	input:
		fq1 = "data/reads/{ID}_1.fq.gz",
		fq2 = "data/reads/{ID}_2.fq.gz"
	output:
		html1 = "results/qc/{ID}_1_fastqc.html",
		html2 = "results/qc/{ID}_2_fastqc.html"
	params:
		outputdir = "results/qc/"
	shell: """
		fastqc -q -o {params.outputdir} {input.fq1} {input.fq2}
	"""
	
```

This is what I've done in my solution.

However, this dataset has an annoying property: the files aren't named quite the way we would want
them. In our case, we want to name them after the samples, but they are currently named after
accessions.

Therefore another way to start is to have a first rule that renames the files to have the right
names. (This is great because it gets done once, at the top, and all the other rules never have to
worry about the accession at all.) In practice it'd be better to leave the original files and
instead copy of symlink them into the right names - like this:

```
rule copy_data_by_sample
	input:
		fq1 = "data/reads/{something}_1.fq.gz",
		fq2 = "data/reads/{something}_2.fq.gz"
	output:
		fq1 = "results/reads/{ID}_1.fq.gz"
		fq2 = "results/reads/{ID}_2.fq.gz"
	shell: """
		cp {input.fq1} {output.fq1}
		cp {input.fq2} {output.fq2}
	"""
```

The difficult here is what goes in the "something" in the input wildcards.

To get this right you have to remember that snakemake works backwards from its output file
wildcards (i.e. `{ID}`) to its input files. You therefore need a way to compute the accession from
the supplied `ID`.

All this info is in `config['samples']` though, so that should be easy, right? Let's make a
function that looks up the info by sample name:

```
def find_sample_with_name( name ):
	samples = config['samples']
	result = None
	for x in samples:
		if x['name'] == name:
			return x

	raise( "Uh-oh, no sample with name \"%s\" could be found." % name )
```

:::tip Note
This of course a terrible implementation, but it works for this example.
If you loaded the sample sheet as a pandas dataframe, you could use pandas subsetting to find the sample instead.
:::

Now your input filenames can use a function:

```python
def get_input_filename_for_sample( wildcards ):
	return "data/reads/{ID}_1.fq.gz".format( 
		ID = find_sample_with_name( wildcards.name )['ID']
	)
```

And the rule is:
```
rule copy_data_by_sample
	input:
		fq1 = get_input_filename_for_sample
		(etc.)
```

:::tip Note

A slicker way that involves less function-writing is to just use a *lambda function* (i.e. an
unnamed function). So it would look like:

```
rule copy_data_by_sample
	input:
		fq1 = lambda w: "data/reads/{ID}_1.fq.gz".format(
			ID = find_sample_with_name( w.name )['ID']
		)
		(etc.)
```
:::

In my solutions I've used a slightly different approach where the fastq files themselves aren't copied/renamed, but the renaming
gets done by the alignment step. This has the advantage that we don't copy the data, but it's not as simple because all major
'output' rules have to figure out how to rename the output file appropriately.

:::tip Note

Another way to avoid copying is to use symbolic links ("*symlinks*") instead. The `ln` command can be used to do this. For example:

```
ln -s file.txt link_name.txt
```

will create a symlink called `link_name.txt` that points to the original file.

This is helpful because it avoids copying data, but it comes with a big caveat: having multiple names referencing the same file in
the filesystem can quickly get very confusing. (For example, what happens if you delete the symlink above - does the file get
deleted? What happens to the symlink if you delete the original file?)

I typically don't use symlinks now except in rare cases where I name them `something.symlink` so it's completely obvious that they
are symbolic links.
:::


[Go back to the tips and tricks](#tips-and-tricks).

### My snakefiles are getting too big!

After a while you'll find your pipelines have loads of rules and can become hard to understand.

To fix this, I often use the snakemake
[`include` feature](https://snakemake.readthedocs.io/en/stable/snakefiles/modularization.html) to split up the
file into components of related rules. For example, in our pipeline there are a bunch of rules for
read qc, some for alignment and post-processing, a bunch for variant calling, and a bunch for
computing coverage and so on. So for the whole pipeline above I might end up with this structure:

    analysis folder/
        pipelines/
            master.snakmake
            functions.snakemake
            qc.snakemake
            alignment.snakemake
            variants.snakemake
            coverage.snakemake

And the first few lines of `master.snakemake` would be:

```
include: "functions.snakemake"
include: "qc.snakemake"
include: "alignment.snakemake"
include: "variants.snakemake"
include: "coverage.snakemake"
```

Now you can write one 'module' of the pipeline in each file, keeping related rules together without
them becoming too big and unweildy.

:::tip Note
You'll notice I included a `functions.snakemake` above. This is where I tend to put helper
functions, like the `find_sample_with_name()` function mentioned above.
:::

[Go back to the tips and tricks](#Tips-and-tricks).

### Keeping a fast iteration time during development.

When you're developing a pipeline, you don't want to wait two hours only to discover that it didn't
work. A good idea would therefore be to develop your practical using smaller, sub-sampled version
of the datasets (whichever of the above raw data you use). For example, you could run:

```
gunzip -c filename.fastq.gz | head -n 4000 | gzip -c > filename.subsampled.fastq
```

to take the first few reads from each file.

:::tip Question
The above command specifies a multiple of 4 lines. Why? How many reads does the above
command extract?
:::

If you set your pipeline up the way I suggest above then you can have a config file for the small
test dataset, and then once it is all working, rerun using the real config file specifying the full
dataset.

[Go back to the tips and tricks](#Tips-and-tricks).

### Dealing with intermediate files

The alignment steps in [our pipeline](pipeline.svg) in particular are notorious for generating intermediate files.  Indeed:

* the alignment step outputs a SAM file...
* which is converted to a BAM file...
* which you then have to sort by position...
* in which you then have to mark the duplicates...
* which are then indexed.

That's at least 3 intermediate files along the way. We don't want to keep these, they were just
needed during the pipeline.

There are a few different ways to deal with this. One way is just to use unix pipes to pipe command
together within rules - as in

```sh
bwa mem reference.fa read1.fq read2.fq | samtools view -b -o aligned.bam
```

However, I find that this makes the pipeline hard to debug (which command failed? you can't tell.)

Instead, I typically go for temp files and use the snakemake `temp()` function to tell snakemake
files are temporary. So I might write the alignment rule as:

```
rule align_reads:
  input:
    fq1 = something,
    fq2 = something
  output:
    sam = temp( "results/alignment/{sample_name}.sam" )
  shell: "bwa mem ..."
```

As you can see, I tend to also put temporary files into their own `tmp/` folder as well - this
avoids cluttering up the results folder when jobs fail.

Second, rules can refer to other rule outputs, so the next step in the pipeline can be written:
```
rule fix_matepair_information_and_convert_to_bam:
  input:
    sam = rules.align_reads.output.sam
  output:
    bam = temp( "results/alignment/tmp/{sample_name}.bam" )
  shell: "samtools fixmate -m {input.sam} {output.bam}"
```
and so on down the pipeline.

Third - `snakemake` actually has a [named pipe
output](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#piped-output) feature, so
you can get the benefit of the UNIX pipe with the same syntax as above - just replace `temp()` with
`pipe()` and it should automatically work. (I've never actually used this feature but it's a nice
idea for this step, because the SAM file output by `bwa` might be huge when applied to real data.)

[Go back to the tips and tricks](#Tips-and-tricks).

### But I want to run the yellow bits too!

Be my guest! Running variant annotation in particular would be a good thing to do, as would looking
at post-alignment QC metrics.

### Read groups what now?

Some programs require reads to have 'read groups'. What are they and how do you get them in there?

BAM files can easily be post-processed and merged. Read groups are a way to put information in that
records the original sample and the sequencing run, so that downstream programs can distinguish
these. The read groups are encoded in the `@RG` header field of the BAM file (which you can see
using `samtools view -h`), and in the `RG` tag for each alignment. A good document on read groups
is [this one on the GATK
website](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups).

In our pipeline these don't seem that important (we have one alignment file per input fastq file
pair), but in other pipelines the same sample might have been sequenced many times and the results
merged. So for sensible downstream analysis it would be important to keep track of the originating
samples. In particular, variant callers like `octopus` require you to have read groups in the BAM
file.

The simplest way to put read groups into the BAM file is to have `bwa` put them in at the alignment
step. For our experiment this can be done using the `-R` option - e.g. for sample `QC0033-C` this
would look like this:

```
bwa mem -R "@RG\tID:ERR377582\tSM:QG0033-C\tPL:ILLUMINA" [other arguments]
```

You can put other stuff into a read group (see below), but the run ID, the sample name, and the
platform are all that we need just now.

Of course you have to be able to generate this for each sample. With the layout described above, I
wrote the following code (which I put, of course, in `pipelines/functions.snakemake`) to do it:

```
def get_read_group_line( name ):
	sample = find_sample_with_name( name )
	return "@RG\\tID:{ID}\\tSM:{sample}\\tPL:ILLUMINA".format(
		ID = sample['ID'],
		sample = sample['name']
    )
```

The alignment step could then be updated to use this function:
```
rule align_reads:
  input:
    fq1 = something,
    fq2 = something
  output:
    sam = temp( "results/alignment/{name}.sam" )
  params:
    read_group_spec = lambda wildcards: get_read_group_line( wildcards.name )
  shell: """
    bwa mem -R {params.read_group_spec} ...
  """
```

If you look at the resulting files, they have an `@RG` header record and `RG` tags for each read -
octopus will then accept these files.

What else can go in a read group? As the [GATK documentation
indicates](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups) the read
group can also contain information about the sequencing flowcell, lane, and sample barcode, and an
identifier for the library itself. Unfortunately some of this information can be hard to come by
depending on where your reads come from. As we describe below, some of it can be obtained from the
read names in the fastq files. For the data in this practical, some parts such as the library
identifier can be found on the [ENA website](ERR377582). But in general it's a bit hard to put it
all together. (Luckily just the sample name and identifier are enough for our analysis.)

[Go back to the tips and tricks](#Tips-and-tricks).

### What's in the fastq header?

If you look at the header / read name rows of a fastq file you'll see they actually contain a bunch
of information - like this:

```
@ERR377582.7615542 HS23_10792:2:2307:6524:31920#15/1
```

This row tells us the sample ID (`ERR377582`) and the read identifier (`7615542`). And this is
followed by information identifying the instrument that generated the reads (`HS23_10792`), the
flowcell lane and tile number in the lane (`2:2307`), the coordinates of the
[cluster](https://www.broadinstitute.org/files/shared/illuminavids/clusterGenSlides.pdf) within the
tile (`6524`, `31920`), a number identifying the index of the sample within a multiplexed set of
samples (i.e. all run at the same time; `#15`), and whether it's read 1 or 2.

Some of this info can be put in the read group as well.

**Note.** The format of this information is not standard across platforms, and it changes depending
on your data provider. Some other examples can be found [on
wikipedia](https://en.wikipedia.org/wiki/FASTQ_format#Illumina_sequence_identifiers) or on the
[GATK read groups page](https://gatk.broadinstitute.org/hc/en-us/articles/360035890671-Read-groups).

[Go back to the tips and tricks](#Tips-and-tricks).

### Octopus is taking too long!

The [Octopus variant caller](https://github.com/luntergroup/octopus) can take a long time to do its
work - hopefully reflecting that it is trying its best to make high-quality variant calls. This
might take too long to run on your laptop. If so, here are some options for speeding it up:

* If you have a multi-core CPU, you can use more threads (`--threads` argument).

* Restrict to a set of regions.  You can add the `--regions` option to tell Octopus to only work on specified regions.  For this tutorial, please include these regions: `--regions Pf3D7_02_v3:616190-656190 Pf3D7_02_v3:779288-859288 Pf3D7_11_v3:1023035-1081305`.  (This brought the calling down to about half and hour when I tried it.)

* You could also try the Octopus 'fast' or 'very fast' modes - though I haven't tried this.

See `octopus --help` for a full list of options.

(In general this might be less of a problem for real work as you might run it a compute cluster.)

Another option is to try a different variant caller - [`GATK HaplotypeCaller`](https://gatk.broadinstitute.org/hc/en-us), [`DeepVariant`](https://www.nature.com/articles/nbt.4235) or [`freebayes`](https://github.com/freebayes/freebayes) might be possibilities.  For a simple approach you could also use [`bcftools mpileup` and `bcftools call`](https://samtools.github.io/bcftools/bcftools.html) (this is likely to be substantially faster as it relies on the input alignments does not try to reconstruct local haplotypes near each variant.)

[Go back to the tips and tricks](#Tips-and-tricks).

### What ploidy?

Blood-stage malaria parasites are [haploid](https://www.cdc.gov/malaria/about/biology/index.html).
So set octopus `--organism-ploidy 1`.

On the other hand, [mixed infections are common](https://doi.org/10.7554/eLife.40845.001), and to
handle this most projects actually treat samples as if they were diploid - they then treat
heterozygote calls as 'mixed' calls. This is *ad hoc* but works ok. So you could set
`--organism-ploidy 2`.

For the purposes of this tutorial you could do either - or both so we can see the difference?

[Go back to the tips and tricks](#Tips-and-tricks).

### Tools that use temporary directories

Some tools have that bad habit of leaving 'stuff' in the directory you run them in. This is really
annoying for pipelines because you don't want that - you want to put the temp stuff away somewhere
out of the way. `octopus` is one of these tools: if you run it you'll see it outputs a directory
called `octopus-temp`.

Really the only way to deal with this is use the program help to find the option that renames the
temp dir - then send it somewhere different. For example:

```
octopus [other options] --temp-directory-prefix results/variants/tmp/octopus/
```

This will probably work here. (In general you may need to use snakemake wildcards etc. to name this
temp directory so it doesn't clash if the same rule runs multiple jobs in parallel.)s

### Tips on using `bwa mem`

Here are a few options you can use to `bwa mem` which you might want to consider using.

* Run `bwa mem` for a list of options.

* The basic usage is `bwa mem -o output.sam [path to indexed fasta file] read1.fq.gz read2.fq.gz`

* The `-t` option tells `bwa` to use more than one thread.  (If doing this, make sure to also [tell snakemake the number of threads it will use](https://snakemake.readthedocs.io/en/stable/snakefiles/rules.html#threads).)

* The `-R` option specifies that `bwa` should include a read group header line and read groups tags in the output - this is [described above](#read-groups-what-now).

* By default, `bwa` will output only the best alignment for each read.  However, if you specify the `-a` option then `bwa` will output multiple possible alignments for each read, but all except one will be marked as 'not a primary alignment' (using the [`SAM flags`](https://broadinstitute.github.io/picard/explain-flags.html)).  This can be useful in cases when you want to consider possible alternate alignments.

There are also other aligners out there. [`minimap2`](https://github.com/lh3/minimap2) is another
good choice. Nevertheless the field typically currently uses `bwa mem` for Illumina short-read NGS
data.

### Tips on using `samtools`

Here are some tips on using `samtools`:

* Run `samtools` for a list of commands, and `samtools [command]` for a list of options for each command.

* Some commands, like `samtools markdup`, take the output filename as a seperate argument.  But others, such as `samtools view` or `samtools sort`, want you to use the `-o` option to specify the output file (otherwise they output to standard output).

* `multiqc` can read `samtools stats` output, useful for post-alignment QC.

### Solutions

My solution to this tutorial can be found
[in this folder](https://github.com/whg-training/whg-training-resources/tree/main/docs/next_generation_sequencing/building_an_ngs_pipeline/solutions).

(As mentioned above, I used a rename-files-at-the end strategy so this might look a bit different to yours.)

## Good luck!

