# perl-cell_statistics
perl to statistics the standard cell type and its number used in netlist

Code:

#!/usr/bin/perl

$in=$ARGV[0];

if(!defined $in) {
    die "Usage:$0 filename";
} 

my $out=$in;
$out=~s/(\.\w+)?$/.txt/;

if(!open $in_fh, '<', $in) {
    die "cannot open '$in':$!";
}

if(!open $out_fh, '>', $out) {
    die "connot open '$out':$!";
}

my @celltype =();
my $flag = 0;
my %cells = ();

while(<$in_fh>) {

    chmop;    
    if(/^\s*[A-Z]\w+_X[0-9]+_A[0-9]+TR/) {
        $_ =~ /(.*_.*_.*)\s+(\w+)\s+(\(.*)/;
        my $type = $1;
        foreach my $elem (@celltype) {
            if ($elem eq $type) {
                  $flag = 1;
                  last;
            }
        }
        if($flag==0) {
            push @celltype, $1;
        }
        $flag = 0;
    }
}

print "cell_type: @celltype\n";
print "\n";

foreach my $element (@celltype) {
    $cells{"$element"} = 0;
}

if(!open $in_fh_1, '<', $in) {
    die "cannot open '$in':$!";
}

while(<$in_fh_1>) {

    chmop;
    if(/^\s*[A-Z]\w+_X[0-9]+_A[0-9]+TR/) {
        $_ =~ /(.*_.*_.*)\s+(\w+)\s+(\(.*)/;
        if(exists $cells{"$1"}) {
            $cells{"$1"} = $cells{"$1"} + 1;   
        }
    }
}

foreach my $celnum (sort(keys{%cells})) {
    print $out_fh "$celnum : $cells{$celnum}\n";}
