---
layout: post
Title: Necessary Evil  
Author: Chris Prather
Date: 2005-10-14 17:22:18
---

# Necessary Evil
<code>
$> alias tarcat="perl5 -MArchive::Tar -e 'print Archive::Tar->new(shift)->get_content(shift)'"
</code><code>
$> tarcat mytarball.tar.gz filename
</code>

Now you can cat a specific file from inside the tarball... requires IO::Zlib and Archive::Tar though but hey ... it works!
