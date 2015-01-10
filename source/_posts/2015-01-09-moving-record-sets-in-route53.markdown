---
layout: post
title: "Moving Record Sets in Route53"
date: 2015-01-09 22:06:42 -0800
comments: true
categories:
author: Kenneth Berland <ken@grnds.com>
---

Amazon makes it easy to import a zone file into Route53 through their GUI, but not so easy to export.  Thankfully, you can export using the aws CLI, transform that result, and easily import using the GUI.

- Find the hosted zone to export:

```bash
aws route53 list-hosted-zones
```

- Save this ruby to parse the output.  You can tune this script to pick the types of records you'd like or to transform the record name:

```ruby
#!/usr/bin/env ruby

require 'json'
json = $stdin.read
records_to_print = ['A']
transform_domain = ['grandrounds.com', 'grandroundshealth.com'] # old, new

JSON.parse(json)['ResourceRecordSets'].each do |rs|
  rs['ResourceRecords'].each do |r|
    print "#{rs['Name'].gsub(transform_domain[0], transform_domain[1])} #{rs['TTL']} IN #{rs['Type']} #{r['Value']}\n" if records_to_print.rindex(rs['Type'])
  end
end
```

- Export the zone json, pipe it to the ruby parser, and capture the results on your clipboard:

```bash
aws route53 list-resource-record-sets --hosted-zone-id /hostedzone/Z2DIRTC0AKGD29 | ./zone-parse.rb | xsel -ib
```

- Finally, import the zone file snippet

![Route 53 Image #2](http://grh-wiki-pics.s3.amazonaws.com/r53-2.png)
![Route 53 Image #1](http://grh-wiki-pics.s3.amazonaws.com/r53-1.png)
