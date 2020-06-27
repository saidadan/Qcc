sub scan {
    my ($url) = @_;

    # URLs that denote directories are s'posed end with a /.  We really
    # need to talk to the server and read the response headers.
    if($url !~ m|/$| && $url !~ m|\.htm(l?)$|) {
        $url .= '/';
    }

    # Do surgery on the url to extract certain parts.

    # Find the protocol://hostname part.
    $url =~ m@^([a-zA-Z]+\://[a-zA-Z\.]+)(/|$)@;
    my $prothost = $1;

    
    my $stem = $url;
    $stem =~ s|/[^/]*$|/|;

    
    open(IN, "lynx -useragent=fred -source $url 2>/dev/null|") or return 0;

    while(<IN>) {
        while(s/<\s*A\s+[^>]*HREF\s*\=\s*"([^"]+)"//i) {
            my $ref = $1;

            if($ref =~ m|^[a-zA-Z]*\://|) {
                # Already absolute.
            } elsif($ref =~ m|^/|) {
                # Relative to host.
                $ref = "$prothost$ref";
            } else {
                # Relative to page location.
                $ref = "$stem$ref";
            }

            print "   $ref\n";
        }
    }

    close IN;
    return 1;
}
