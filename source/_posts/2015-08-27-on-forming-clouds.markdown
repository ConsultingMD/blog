---
layout: post
title: "On Forming Clouds"
date: 2015-08-27 17:00:16 -0700
comments: true
author: Justin Ahn
categories: AWS CloudFormation Ruby Configuration troposphere
---

CloudFormation! What a wonderful tool.  For the uninitiated, CloudFormation is a tool that allows you to create and manage collections of AWS objects by submitting JSON templates.  What a wonderfully simple idea, make some JSON, get some infrastructure.

So what's the catch?

Often times, your JSON templates get too large and unwieldly to manage well.  You end up attempting to read thousands of lines, where any one line could affect many lines throughout the entire template, and become impossible to manage.  Just take this snipet for example.
```
"route1": {
  "Properties": {
    "DestinationCidrBlock": "0.0.0.0/0",
    "InstanceId": {
      "Ref": "ec2natinstance"
    },
  }
    "RouteTableId": {
    "Ref": "XXXXXXXX"
  }
  "Type": "AWS::EC2::Route"
},
"XXXXXXXX": {
  "Properties": {
    "Tags": [{
      "Key": "Name",
      "Value": "Private Routes"
    }],
    "VpcId": {
      "Ref": "thevpc"
    }
  },
  "Type": "AWS::EC2::RouteTable"
},
```
Changing just `route1` might affect your entire VPC, and the routes involved.

"Well, I'd just write a wrapper for the JSON, and treat the JSON like a template, passing the needed variables into it and make it generate the right JSON for me!" you might say.  
That would be a pretty smart approach.  Your template is then ruled by code, and objects that will render the correct JSON to build yourself your monolithic JSON document.  A snippet for such a template might look like the following.
```
<%- selected_instances.each do |instance| -%>
"<%= instance[:ec2_task_name] %>" : {
"Type" : "AWS::EC2::Instance",
"Properties" : {
"InstanceType" : "<%= instance[:type]%>",
"ImageId" : "<%= ami %>",
...
<%- if instance[:associate_pub_ip] %>
"NetworkInterfaces": [
{
"AssociatePublicIpAddress" : "true",
"DeleteOnTermination" : "true",
"Description" : "Primary network interface",
"DeviceIndex" : 0,
"GroupSet" : [ "<%= instance[:security_group] %>" ],
```
This looks way more managable right?  You can use code the update the references and everything will change with it, no more ctrl+f search-and-replacing large parts of your codebase.Pretty cool right?  
***wrong***

This is just as unbearable, your templates start to get confusing, the code gets unbearble to read.  Eventually, the scaling problem from trying to write code that renders the right templates becomes just as bad as the scaling problem of writing the right JSON.

*So what is the better solution?*  Instead of thinking of your JSON template as a JSON template, think of it as objects that require rendering.

Here's an example of such an Class
```
class MetalAWS
  include Virtus.model

  def type
    raise 'Uninitialized AWS Class'
  end

  def basics
    {
      'Properties' => properties,
      'Type' => type
    }
  end

  def to_cfn
    blob = {}
    blob[name] = basics
    blob.to_naked_json
  end
end

class SimpleNotificationService < MetalAWS
  attribute :base_name, String

  private

  def type
    'AWS::SNS::Topic'
  end

  def name
    "topic#{base_name.gsub('_','')}"
  end

  def properties
    {
      'DisplayName' => base_name,
      'TopicName' => base_name
    }
  end
end
```

Awesome!  Now we can create any simple notification service we want!  All we need know is the base name and we can render the object!
```
SimpleNotificationService.new(
  base_name: 'records_request'
)
puts sns.to_cfn

"topicrecordsrequest": {
  "Properties": {
    "DisplayName": "records_request",
    "TopicName": "records_request"
  },
  "Type": "AWS::SNS::Topic"
}
```

How awesome is that?!