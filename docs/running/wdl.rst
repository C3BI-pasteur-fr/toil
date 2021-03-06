WDL Support in Toil
*******************

Support is still in the alpha phase and should be able to handle basic wdl files.  See the specification below for more
details.

How to Run a WDL file in Toil
-----------------------------
Recommended best practice when running wdl files is to first use the Broad's wdltool for syntax validation and generating
the needed json input file.  Full documentation can be found on the repository_, and a precompiled jar binary can be
downloaded here: wdltool_ (this requires java7_).

That means two steps.  First, make sure your wdl file is valid and devoid of syntax errors by running:

``java -jar wdltool.jar validate example_wdlfile.wdl``

Second, generate a complementary json file if your wdl file needs one.  This json will contain keys for every necessary
input that your wdl file needs to run:

``java -jar wdltool.jar inputs example_wdlfile.wdl``

When this json template is generated, open the file, and fill in values as necessary by hand.  WDL files all require
json files to accompany them.  If no variable inputs are needed, a json file containing only '{}' may be required.

Once a wdl file is validated and has an appropriate json file, workflows can be run in toil using:

``toil-wdl-runner example_wdlfile.wdl example_jsonfile.json``

See options below for more parameters.

ENCODE Example from ENCODE-DCC
------------------------------
To follow this example, you will need docker installed.  The original workflow can be found here:
https://github.com/ENCODE-DCC/pipeline-container

We've included the wdl file and data files in the toil repository needed to run this example.  First, download
the example code_ and unzip.  The file needed is: "testENCODE/encode_mapping_workflow.wdl".

Next, use wdltool_ (this requires java7_) to validate this file:

``java -jar wdltool.jar validate encode_mapping_workflow.wdl``

Next, use wdltool to generate a json file for this wdl file:

``java -jar wdltool.jar inputs encode_mapping_workflow.wdl``

This json file once opened should look like this::

    {
    "encode_mapping_workflow.fastqs": "Array[File]",
    "encode_mapping_workflow.trimming_parameter": "String",
    "encode_mapping_workflow.reference": "File"
    }

The trimming_parameter should be set to 'native'.
Download the example code_ and unzip.  Inside are two data files required for the run:

``ENCODE_data/reference/GRCh38_chr21_bwa.tar.gz``
``ENCODE_data/ENCFF000VOL_chr21.fq.gz``

Editing the json to include these as inputs, the json should now look something like this::

    {
    "encode_mapping_workflow.fastqs": ["/path/to/unzipped/ENCODE_data/ENCFF000VOL_chr21.fq.gz"],
    "encode_mapping_workflow.trimming_parameter": "native",
    "encode_mapping_workflow.reference": "/path/to/unzipped/ENCODE_data/reference/GRCh38_chr21_bwa.tar.gz"
    }

The wdl and json files can now be run using the command:

``toil-wdl-runner encode_mapping_workflow.wdl encode_mapping_workflow.json``

This should deposit the output files in the user's current working directory (to change this, specify a new directory
with the '-o' option).

GATK Examples from the Broad
----------------------------
Simple examples of WDL can be found on the Broad's website as tutorials:
https://software.broadinstitute.org/wdl/documentation/topic?name=wdl-tutorials

One can follow along with these tutorials, write their own wdl files following the directions and run them using either
cromwell or toil.  For example, in tutorial 1, if you've followed along and named your wdl file 'helloHaplotypeCall.wdl'
then once you've validated your wdl file using wdltool_ (this requires java7_):

``java -jar wdltool.jar validate helloHaplotypeCaller.wdl``

And generated a json file (and subsequently typed in appropriate filepaths* and variables):

``java -jar wdltool.jar inputs helloHaplotypeCaller.wdl``

* Absolute filepath inputs are recommended for local testing.

Then the wdl script can be run using the command:

``toil-wdl-runner helloHaplotypeCaller.wdl helloHaplotypeCaller_inputs.json``

toilwdl.py Options
------------------
**'-o'** or **'--output_directory'**: Specifies the output folder, and defaults to the current working directory if
not specified by the user.

**'--gen_parse_files'**: Creates "AST.out", which holds a printed AST of the wdl file and "mappings.out", which holds the
printed task, workflow, csv, and tsv dictionaries generated by the parser.

**'--dont_delete_compiled'**: Saves the compiled toil python workflow file for debugging.

Any number of arbitrary options may also be specified.  These options will not be parsed immediately, but passed down
as toil options once the wdl/json files are processed.  For valid toil options, see the documentation:
http://toil.readthedocs.io/en/3.12.0/running/cli.html

WDL Specifications
------------------
WDL language specifications can be found here: https://github.com/broadinstitute/wdl/blob/develop/SPEC.md

Implementing support for more features is currently underway, but a basic roadmap so far is:

CURRENTLY IMPLEMENTED:
 * scatter
 * read_tsv, read_csv
 * docker calls
 * handles priority, and output file wrangling
 * currently handles primitives and arrays

TO BE IMPLEMENTED SOON:
 * implement type: $type_postfix_quantifier
 * "default" values inside variables
 * $map_types & $object_types
 * wdl files that "import" other wdl files (including URI handling for 'http://' and 'https://')

.. _repository: https://github.com/broadinstitute/wdltool
.. _wdltool: https://github.com/broadinstitute/wdltool/releases
.. _java7: http://www.oracle.com/technetwork/java/javase/downloads/java-archive-downloads-javase7-521261.html
.. _here: https://github.com/BD2KGenomics/toil/tree/master/src/toil/test/wdl/ENCODE_data.zip
.. _code: http://toil-datasets.s3.amazonaws.com/ENCODE_data.zip
