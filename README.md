# pcast-batch

## Description 

From a list of MP3 audio inputs, apply one or more of the following operations:

1. Cut the indicated sections from the audio.
1. Apply a custom audio [filter](#filter).
1. Name the final audio appropriately.
1. Create a new (web page, feed, snippet) entry from the new audio based on a configured [template](#template). 

## Requirements

- Perl 5.0+
- ffmpeg

## Usage

```
pcast-batch audios.lst
```

`audios.lst` should contain (by default) semicolon delimited entries in the following format.

```
date;input;output;title;cut;desc
[optional];input.mp3;output.mp3;[optional];[optional];[optional]
...
```

Blanks or comments beginning with '#' are ignored. The header line is optional. All the fields beyond input and output are optional. The *date*, *title*, and *desc* can be used in the output [template](#template). *cut* can contain a comma-delimited list of start and end timestamps that you wish to eliminate from each audio. See the below example:

```
date;input;output;title;cut;desc
2019-09-25;input1.mp3;output1.mp3;First podcast;00:30-00:35,01:03-01:10;
2019-09-26;input2.mp3;output2.mp3;Second podcast;;
```

The above list specifies that the first entry be cut at the indicated time ranges, a custom filter then applied (see the [configuration](#filter)), and finally saved as output1.mp3 in the configured output directory. The date and title can optionally appear in the output [template](#template). The second entry is similar except requiring no cuts.

The application skips any nonexistent input audio and does not overwrite any already existing files.

## Configuration

Include a configuration file `.pcast-batch.cfg` in the current or home directory, or `~/.config/pcast-batch.cfg` to overwrite the restricted default values. See `.pcast-batch.cfg.def` for an example.

### Custom audio filter <a name="filter" />

Set `$FILTER_CMD` to a custom ffmpeg command with your preferred filter of choice. The string '\$input' gets replaced with the audio prior to the filter stage. The '\$output' string gets replaced with the appropriately named output filename per the audio list. The following command, for example, overlays the audio with a cross-fading 'outro' taken from the file 'intro\_outro\_64k.mp3', and beginning at 4 seconds before the main audio terminates.

```
$FILTER_CMD='"ffmpeg -v warning -n -i $pre_output -i $INPUTDIR/intro_outro_64k.mp3 -filter_complex acrossfade=d=4:o=1:c1=qsin:c2=qsin $output"';
```

Note carefully the outer single quotes enclosing a set of double quotes around the filter!

### Output template <a name="template" />

The option `$OUTPUT_TEMPLATE` indicates the format for an entry (corresponding to the processed audio) to be appended to the file in `$OUTPUT_APPEND_FILE`. An example best demonstrates this:

```
$OUTPUT_TEMPLATE = '"$date#$title#/podcasts/$output#$desc"';
$OUTPUT_APPEND_FILE = "all-pcasts.csv";
```

The above appends a string in the given format a '#'-delimited file 'all-pcasts.csv'. The metadata fields '\$date', '\$title', '\$output', '\$desc' correspond precisely to the entries in the audio list file. This particular example, perhaps not clear in intent, accumulates new entries in this special CSV format for another tool to then process. 

Alternatively, for a more transparent use case, the following example demonstrates how you might append an entry to the body of a podcast web page:

```
$OUTPUT_TEMPLATE = '"<p>Date: $date, <a href=\"/podcasts/$output\">$title</a> - $desc</p>"';
$OUTPUT_APPEND_FILE = "index.html";
```
