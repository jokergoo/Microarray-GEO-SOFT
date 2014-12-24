## Reading microarray data in SOFT format from GEO database.

### install

```
cpan -i Microarray::GEO::SOFT
```

### usage

```perl
use Microarray::GEO::SOFT;
use strict;
 
# initialize
my $soft = Microarray::GEO::SOFT->new; 
 
# download
$soft->download("GDS3718");
$soft->download("GSE10626");
$soft->download("GPL1261");
 
# or else you can read local data
$soft = Microarray::GEO::SOFT->new(file => "GDS3718.soft");
$soft = Microarray::GEO::SOFT->new(file => "GSE10626_family.soft");
$soft = Microarray::GEO::SOFT->new(file => "GPL1261.annot");
 
# parse
# it returns a  Microarray::GEO::SOFT::GDS,
# Microarray::GEO::SOFT::GSE or Microarray::GEO::SOFT::GPL object
# according the the GSE ID type
my $data = $soft->parse;
 
# some meta info
$data->meta;
$data->title;
$data->platform;
 
# for GPL and GDS, you can get the data table
$data->table;
$data->colnames;
$data->rownames;
$data->matrix;
 
# sinece GSE can contain more than one GPL
# we can get the GPL list in a GSE
my $gpl_list = $data->list("GPL");
 
# merge samples belonging to a same GPL into a data set
my $gds_list = $data->merge;
 
# if the GSE only have one platform
# then the merged data set is the first one in gds_list
# and the platform is the first one in gpl_list
my $g = $gds_list->[0];
my $gpl = $gpl_list->[0];
 
# since GPL data contains different mapping of genes or probes
# we can transform from probe id to gene symbol
# it returns a Microarray::ExprSet object
my $e = $g->id_convert($gpl, "Gene Symbol");
my $e = $g->id_convert($gpl, qr/gene[-_\s]?symbol/i);
 
 
# if you pased a GDS data
# you can first find the platform
my $platform_id = $data->platform;
# downloaded or parse the local file
my $gpl = Microarray::GEO::SOFT->new->download($platform_id);
# and do the id convert thing
my $e = $data->id_convert($gpl, qr/gene[-_\s]?symbol/i);
 
# or just transform into Microarray::ExprSet direct from GDS
my $e = $g->soft2exprset;
 
# then you can do some simple processing thing
# eliminate the blank lines
$e->remove_empty_features;
 
# make all symbols unique
$e->unify_features;
 
# obtain the expression matrix
$e->save('some-file');
```

### command line

```
getgeo --id=GDS3718
getgeo --file=GDS3718.soft --verbose
```

options are

```
--id=[GEOID]
 
  GEO ID. such as GSE123, GDS123 or GPL123. If this is set, the script would
  download data from GEO FTP site.
 
--proxy=[PROXY]
 
  Proxy to connect to GEO FTP site. Format should look like
  http://username:password@host:port/.
 
--file=[FILE]
 
  Filename for local GEO file. If --id is set, this option is ignored.
 
--tmp-dir=[DIR]
 
  Temporary directory name for processing of GEO data. By default it is
  '.tmp_soft' in your working directory.
 
--verbose
 
  Whether print message while processing.
 
--sample-value-column=[FIELD]
 
  Since there may be multiple columns in GSM record, users may specify which
  column is the expression value they want. By default it is 'VALUE'. Ignored
      when analyzing GPL and GDS data.
 
--output-file=[FILE]
 
  Filename for the output file. By default it is 'GEOID.table' in your current
 
  working directory.
 
--help
 
  Help message.
  ```
